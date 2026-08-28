# OpenSearch 到 StarRocks 迁移方案

本文档介绍将 OpenSearch 集群中的数据全量迁移至 StarRocks 的方案。数据传输链路如下：

```
OpenSearch → Logstash → Kafka → Flink → StarRocks
```

各组件职责：

| 组件 | 职责 |
|---|---|
| OpenSearch | 数据源，提供待迁移的索引数据 |
| Logstash | 从 OpenSearch 按分片切片（slice）并行抽取数据，写入 Kafka |
| Kafka | 数据缓冲中间层，解耦上下游，保障传输稳定性 |
| Flink | 从 Kafka 消费数据，完成 ETL 类型转换，写入 StarRocks |
| StarRocks | 数据目标，接收并存储迁移后的数据 |

---

## 前置准备

### 中间件资源规划

迁移涉及 Logstash、Kafka、Flink 三类中间件，建议按以下规格准备：

| 资源 | 说明 | 建议规格 |
|---|---|---|
| Logstash | 云主机自建 | 1 台云主机（4 核 16G / 1000GB） |
| UKafka | UC 控制台创建 | 3 个 Broker（4 核 16G / 800GB） |
| Flink（UHadoop） | UC 控制台创建 | 3 个 Master（4 核 16G / 100GB）、2 个 Task（16 核 64G / 200GB） |

### Kafka Topic 规划

| 参数 | 建议值 | 说明 |
|---|---|---|
| 分区数 | 等于 OpenSearch shard 数 | 每个 pipeline slice（切片）对应一个分区，避免热点 |
| 副本数 | ≥ 2 | 生产环境保障可用性 |

---

## 迁移步骤

### 步骤一：创建 StarRocks 目标表

在 StarRocks 集群中，根据 OpenSearch 源索引的数据结构创建目标表。目标表的字段类型需与源数据匹配；对于需要类型转换的字段（如时间戳、数字转字符串等），在 Flink ETL 阶段处理。

> **注意**：目标表结构、字段类型需与源数据及 Flink 转换逻辑保持一致，否则写入时可能出现类型错误。

---

### 步骤二：配置 Logstash 抽取数据写入 Kafka

Logstash 负责从 OpenSearch 读取数据，通过分片切片（slice）实现并行读取，并将数据写入 Kafka。

#### 2.1 Logstash Pipeline 配置

```ruby
input {
  opensearch {
    hosts          => ["<node1>:9200","<node2>:9200","<node3>:9200"]
    user           => "<username>"
    password       => "<password>"
    ssl            => true
    index          => "<index-name>"
    query          => '{"slice":{"id":<SLICE_ID>,"max":<TOTAL_SLICES>},"query":{"match_all":{}},"sort":[{"_doc":"asc"}]}'
    size           => 3000
    scroll         => "5m"
    docinfo        => true
    docinfo_fields => ["_index", "_id"]
  }
}

filter {
  mutate {
    add_field    => { "[@metadata][kafka_key]" => "%{[<id-field>]}" }
    remove_field => ["@version", "@timestamp"]
  }
}

output {
  kafka {
    bootstrap_servers  => "<broker1>:9092,<broker2>:9092,<broker3>:9092"
    topic_id           => "<topic-name>"
    message_key        => "%{[@metadata][kafka_key]}"
    codec              => json
    batch_size         => 524288
    linger_ms          => 10
    compression_type   => "lz4"
    acks               => "1"
    buffer_memory      => 67108864
    max_request_size   => 20971520
    retries            => 5
    retry_backoff_ms   => 100
    request_timeout_ms => 30000
  }
}
```

| 参数 | 说明 |
|---|---|
| `hosts` | OpenSearch 集群节点地址 |
| `index` | 待迁移的索引名称 |
| `query` | 切片查询，`slice.id` 为切片编号，`slice.max` 为切片总数 |
| `size` / `scroll` | 单次读取的文档数与 scroll 有效期 |
| `topic_id` | Kafka Topic 名称 |
| `message_key` | Kafka 分区键，建议使用业务唯一 ID 字段 |

#### 2.2 配置 pipelines.yml

每个 pipeline 对应一个 slice，实现并行读取：

```yaml
- pipeline.id: slice_00
  path.config: "/opt/logstash-es-kafka/pipeline/slice_00.conf"
  pipeline.workers: 1
  pipeline.batch.size: 500
- pipeline.id: slice_01
  path.config: "/opt/logstash-es-kafka/pipeline/slice_01.conf"
  pipeline.workers: 1
  pipeline.batch.size: 500
# ... 以此类推
```

> 每台云主机 pipeline 数量的推荐值等于 CPU 核数，超过核数后总吞吐下降。

#### 2.3 配置 logstash.yml

```yaml
pipeline.batch.delay: 50
xpack.monitoring.enabled: false
queue.type: persisted          # 启用持久化队列，支持断点续传
queue.max_bytes: 4gb
log.level: warn
```

#### 2.4 配置 JVM 参数

```bash
-Xms12g -Xmx12g
-XX:+UseG1GC
-XX:G1ReservePercent=10
-XX:InitiatingHeapOccupancyPercent=45
-XX:MaxGCPauseMillis=200
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=<log-dir>/heap.hprof
```

> **必须使用 G1GC**：此 workload 产生大量短生命周期 byte 数组，CMS 下 GC 占比可高达 46%，切换 G1GC 后 GC 开销降至 14%，同等配置吞吐提升约 65%。

#### 2.5 启动 Logstash

```bash
# 启动正式 pipeline（后台运行）
nohup <logstash-home>/bin/logstash --path.settings /opt/logstash-es-kafka/config/ \
  > /opt/logstash-es-kafka/logs/startup.log 2>&1 &
```

至此，向 OpenSearch 中写入的数据即会通过 Logstash 写入 Kafka。

#### 2.6 Shard / Slice / Pipeline 对应关系

为最大化并发，三者数量关系需满足：

- Logstash slice 数量不能超过 OpenSearch shard 数量，最多等于 shard 数量
- 一个 pipeline 对应一个 slice，pipeline 与 slice 为 1:1 关系
- 最优配置：OpenSearch shard 数量 = Logstash slice 数量 = Logstash pipeline 数量 = Logstash 集群 CPU 核数

> 建议先确认源索引的 shard 数量，据此设置 slice 与 pipeline 数量。

#### 2.7 验证 Logstash 消费数据

```bash
# 查看 Logstash 日志，确认 slice 读取正常
tail -20 <logstash-home>/logs/logstash-plain.log | grep 'Slice'
```

预期每分钟可看到日志中出现 Slice starting 和 Slice complete。

```bash
# 查看 Logstash 是否正常启动
grep -E 'Pipeline started|Pipeline terminated|error|ERROR' /opt/logstash-es-kafka/logs/startup.log
```

---

### 步骤三：Flink 消费 Kafka 写入 StarRocks

Flink 从 Kafka 消费数据，完成字段类型转换后写入 StarRocks。

#### 3.1 添加 Maven 依赖

在 Flink Job 的 pom.xml 中引入 Kafka 与 StarRocks Connector：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>flink-kafka-starrocks</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>flink-kafka-starrocks</name>
    <description>Flink Kafka to StarRocks Job</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <flink.version>1.18.1</flink.version>
        <flink.kafka.version>3.1.0-1.18</flink.kafka.version>
        <kafka.clients.version>3.4.0</kafka.clients.version>
        <starrocks.flink.version>1.2.14_flink-1.18</starrocks.flink.version>
    </properties>

    <dependencies>
        <!-- Flink 核心依赖（provided scope，Yarn 集群会提供） -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-base</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- Kafka Connector -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka</artifactId>
            <version>${flink.kafka.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.clients.version}</version>
        </dependency>

        <!-- StarRocks Connector -->
        <dependency>
            <groupId>com.starrocks</groupId>
            <artifactId>flink-connector-starrocks</artifactId>
            <version>${starrocks.flink.version}</version>
        </dependency>

        <!-- 日志（provided，Yarn 集群会提供） -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.36</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.6.2</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <createDependencyReducedPom>false</createDependencyReducedPom>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.example.flinkkafkastarrocks.Flink</mainClass>
                                </transformer>
                            </transformers>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

#### 3.2 字段类型转换

OpenSearch 与 StarRocks 的字段类型存在差异，需在 Flink 中对字段做类型转换。常见转换类型如下：

| 转换类型 | 原始类型（Kafka JSON） | 目标类型（StarRocks） | 说明 |
|---|---|---|---|
| 时间戳 → 日期时间 | 毫秒时间戳（`long`） | `datetime`（字符串格式 `yyyy-MM-dd HH:mm:ss`） | 将毫秒值转为日期时间字符串 |
| 数字 → 字符串 | 数字（`long`） | `varchar`（字符串） | 将数字直接转为字符串 |
| 日期字符串 → 时间戳 | 日期字符串（如 `2026-01-01T00:00:00Z`） | `bigint`（毫秒时间戳） | 解析常见日期格式转为毫秒时间戳；若已是数字则保持不变 |

#### 3.3 编写 Flink Job 代码

核心流程：从 Kafka 读取数据 → ETL 类型转换 → 写入 StarRocks。以下为代码骨架，需按实际数据替换连接信息、字段名与目标表：

```java
package com.example.flinkkafkastarrocks;

import com.starrocks.connector.flink.StarRocksSink;
import com.starrocks.connector.flink.table.sink.StarRocksSinkOptions;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.node.LongNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.TextNode;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.connector.kafka.source.KafkaSource;
import org.apache.flink.connector.kafka.source.enumerator.initializer.OffsetsInitializer;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.time.Instant;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;

public class Flink {
    private static final Logger LOG = LoggerFactory.getLogger(Flink.class);
    private static final ObjectMapper MAPPER = new ObjectMapper();
    private static final DateTimeFormatter DT_FMT = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
            .withZone(ZoneId.of("UTC"));

    // Kafka 配置
    private static final String BOOTSTRAP_SERVERS = "<broker1>:9092,<broker2>:9092,<broker3>:9092";
    private static final String TOPIC = "<topic-name>";
    private static final String GROUP_ID = "<group-name>";

    // StarRocks 配置：JDBC 用 FE，load-url 也用 FE（存算分离模式下 Stream Load 走 FE）
    private static final String JDBC_URL = "jdbc:mysql://<fe1>:9030,<fe2>:9030,<fe3>:9030";
    private static final String LOAD_URL = "http://<fe1>:8030;http://<fe2>:8030;http://<fe3>:8030";
    private static final String DATABASE = "<database-name>";
    private static final String TABLE = "<table-name>";
    private static final String USERNAME = "<username>";
    private static final String PASSWORD = "<password>";
    // HDFS Checkpoint 路径
    private static final String CHECKPOINT_PATH = "hdfs:///flink-checkpoints/<table-name>";

    public static void main(String[] args) throws Exception {
        LOG.info("Starting Flink Kafka -> StarRocks job...");
        LOG.info("Kafka: {}, topic: {}", BOOTSTRAP_SERVERS, TOPIC);
        LOG.info("StarRocks: {}/{}, load_url: {}", JDBC_URL, DATABASE + "." + TABLE, LOAD_URL);
        LOG.info("Checkpoint: {}", CHECKPOINT_PATH);

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        // Checkpoint 配置：1 分钟一次，存储到 HDFS
        env.enableCheckpointing(Duration.ofMinutes(1).toMillis(), CheckpointingMode.EXACTLY_ONCE);
        env.getCheckpointConfig().setCheckpointTimeout(Duration.ofMinutes(10).toMillis());
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(Duration.ofSeconds(30).toMillis());
        env.getCheckpointConfig().setExternalizedCheckpointCleanup(
                CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION
        );
        env.getCheckpointConfig().setCheckpointStorage(CHECKPOINT_PATH);

        // Kafka Source
        KafkaSource<String> source = KafkaSource.<String>builder()
                .setBootstrapServers(BOOTSTRAP_SERVERS)
                .setTopics(TOPIC)
                .setGroupId(GROUP_ID)
                .setStartingOffsets(OffsetsInitializer.earliest())
                .setValueOnlyDeserializer(new SimpleStringSchema())
                .build();

        DataStream<String> raw = env.fromSource(
                source,
                WatermarkStrategy.noWatermarks(),
                "kafka_src"
        );

        // 核心转换：将 Kafka JSON 中的字段类型转换为 StarRocks 表兼容格式
        // 1. 时间戳字段：毫秒时间戳 -> "yyyy-MM-dd HH:mm:ss" 字符串（StarRocks datetime 列）
        // 2. 需转字符串的字段：long -> string（StarRocks varchar 列）
        // 3. 日期字段：确保为毫秒时间戳 long（StarRocks bigint 列）
        DataStream<String> transformed = raw.map(new MapFunction<String, String>() {
            @Override
            public String map(String value) throws Exception {
                try {
                    JsonNode root = MAPPER.readTree(value);

                    // 示例：根据实际字段调用对应转换方法
                    convertTimestampToDatetime(root, "<timestamp-field>");
                    convertLongToString(root, "<long-field>");
                    convertDateToMillis(root, "<date-field>");

                    return MAPPER.writeValueAsString(root);
                } catch (Exception e) {
                    LOG.warn("Failed to transform record: {}", e.getMessage());
                    return null;
                }
            }

            private void convertTimestampToDatetime(JsonNode root, String field) {
                if (root.has(field) && !root.get(field).isNull()) {
                    long millis = root.get(field).asLong();
                    if (millis > 0) {
                        String datetime = DT_FMT.format(Instant.ofEpochMilli(millis));
                        ((com.fasterxml.jackson.databind.node.ObjectNode) root).set(field, TextNode.valueOf(datetime));
                    }
                }
            }

            private void convertLongToString(JsonNode root, String field) {
                if (root.has(field) && !root.get(field).isNull() && root.get(field).isNumber()) {
                    String strVal = String.valueOf(root.get(field).asLong());
                    ((com.fasterxml.jackson.databind.node.ObjectNode) root).set(field, TextNode.valueOf(strVal));
                }
            }

            private void convertDateToMillis(JsonNode root, String field) {
                if (root.has(field) && !root.get(field).isNull()) {
                    JsonNode node = root.get(field);
                    if (node.isTextual()) {
                        // 日期字符串 -> 毫秒时间戳
                        String dateStr = node.asText();
                        try {
                            // 尝试解析常见格式
                            long millis;
                            if (dateStr.contains("T") || dateStr.contains("Z")) {
                                millis = Instant.parse(dateStr).toEpochMilli();
                            } else {
                                millis = java.time.LocalDateTime.parse(dateStr,
                                    DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))
                                    .atZone(ZoneId.of("UTC")).toInstant().toEpochMilli();
                            }
                            ((com.fasterxml.jackson.databind.node.ObjectNode) root).set(field, LongNode.valueOf(millis));
                        } catch (Exception e) {
                            LOG.warn("Cannot parse date field {}: {}", field, dateStr);
                        }
                    }
                    // 如果已经是数字则保持不变
                }
            }
        }).filter(v -> v != null);

        // StarRocks Sink
        transformed.addSink(StarRocksSink.sink(buildSinkOptions()))
                .name("sr_sink")
                .setParallelism(2);

        env.execute("kafka_to_starrocks");
    }

    private static StarRocksSinkOptions buildSinkOptions() {
        return new StarRocksSinkOptions.Builder()
                .withProperty("jdbc-url", JDBC_URL)
                .withProperty("load-url", LOAD_URL)
                .withProperty("database-name", DATABASE)
                .withProperty("table-name", TABLE)
                .withProperty("username", USERNAME)
                .withProperty("password", PASSWORD)
                // exactly-once 语义，配合 checkpoint 保证数据一致性
                .withProperty("sink.semantic", "exactly-once")
                .withProperty("sink.version", "V2")
                .withProperty("sink.label-prefix", "flink_<table-name>")
                .withProperty("sink.connect.timeout-ms", "30000")
                .withProperty("sink.socket.timeout-ms", "300000")
                .withProperty("sink.wait-for-continue.timeout-ms", "10000")
                .withProperty("sink.buffer-flush.max-rows", "64000")
                .withProperty("sink.buffer-flush.interval-ms", "5000")
                .withProperty("sink.properties.format", "json")
                .withProperty("sink.properties.strict_mode", "false")
                .withProperty("sink.properties.max_filter_ratio", "1.0")
                .withProperty("sink.properties.columns", "<comma-separated-column-list>")
                .build();
    }
}
```

> **说明**：`sink.properties.columns` 为 StarRocks 目标表的字段列表（逗号分隔），需与目标表结构一一对应。代码编译前，需将 `BOOTSTRAP_SERVERS`、`TOPIC`、`GROUP_ID`、`JDBC_URL`、`LOAD_URL`、`USERNAME`、`PASSWORD` 替换为实际值。

#### 3.4 编译

```bash
mvn clean install
```

#### 3.5 提交任务

将编译完成的 Flink Jar 包提交至 Yarn：

```bash
flink run -t yarn-per-job \
  -Dparallelism.default=3 \
  -Djobmanager.memory.process.size=4g \
  -Dtaskmanager.memory.process.size=4g \
  flink-kafka-starrocks-0.0.1-SNAPSHOT.jar
```

---

### 步骤四：验证

向 OpenSearch 中写入数据，验证 StarRocks 中是否有数据产生。

1. 向源 OpenSearch 索引写入一条测试数据
2. 依次观察 Logstash、Kafka、Flink 各环节数据流转
3. 查询 StarRocks 目标表，确认数据已写入且字段类型转换正确
