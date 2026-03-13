# StarRocks开发指南

## 什么是StarRocks
StarRocks 是优刻得（UCloud）在 UHadoop 体系内推出的实时分析托管服务，作为一款高性能、全场景 MPP 大规模并行处理分析引擎，专为满足企业级实时数据分析需求打造。

依托全托管模式，企业无需关注部署、运维、扩容等底层工作，可专注于业务数据洞察与决策，轻松搭建稳定、高效、成本最优的统一分析平台。服务深度融入 UHadoop 生态，可与 Hadoop、Spark、Kafka、Flink 等组件无缝协同，实现数据全链路处理。

从数据实时接入、实时计算与存储，到即时查询分析、报表可视化，StarRocks 提供端到端实时分析流水线，支持 Kafka、Flink、数据湖等多种数据源，并可对接主流 BI 工具，完整覆盖企业实时数仓建设全流程。

<img src="/images/developer/starrocksdev/starrocks_jiagoutu.png" width="600">


## 产品核心能力
### 1、高并发，稳如磐石
为高并发交互式查询深度优化，能够从容应对业务高峰期的海量查询请求。无论是运营监控大屏、管理驾驶舱还是业务自助分析，都能保证关键指标的查询响应速度与稳定性。

### 2、复杂查询，深度洞察
针对多表关联（复杂JOIN）分析进行专项优化，在用户行为分析、画像分析、多维指标拆解等复杂业务场景下，依然保持高效稳定的查询性能，满足生产级的深度数据分析需求。

### 3、湖仓一体，简捷统一
一套引擎同时支持实时数据和离线数据分析。能够无缝对接数据湖（如Hadoop生态）中的表格式，实现实时明细、汇总分析与数仓级查询的统一，消除数据孤岛，降低多系统拼接带来的复杂性和运维成本。

### 4、全托管，弹性易用
UCloud提供全生命周期的托管服务，包括分钟级部署、自动运维、监控告警和按需弹性扩缩容。用户无需关心底层基础设施，可专注于业务开发，大幅降低运维投入。


## 应用场景
### 1、实时报表与BI分析
支撑企业运营大盘、实时销售看板、流量监控等需要秒级甚至毫秒级延迟的报表场景。

### 2、用户行为与画像分析
高效处理海量用户行为日志，支持快速的标签筛选、人群圈选和画像分析。

### 3、自助数据探索
业务人员可通过主流BI工具（如Tableau、帆软等）对海量数据进行灵活、快速的即席查询和探索。

### 4、实时数据湖分析
作为数据湖的上层分析引擎，直接对湖中的数据进行高效查询分析，构建湖仓一体架构。


## 如何创建StarRocks
1、在控制台选择“托管Hadoop集群 UHadoop”>“集群列表”，点击“创建集群”按钮进行创建

<img src="/images/developer/starrocksdev/starrocks_create01.png" width="900">

2、创建页选择地域、可用区

<img src="/images/developer/starrocksdev/starrocks_create02.png" width="900">

3、集群框架选择“StarRocks框架”，并配置其余参数

<img src="/images/developer/starrocksdev/starrocks_create03.png" width="900">

4、配置节点参数

<img src="/images/developer/starrocksdev/starrocks_create04.png" width="900">

5、网络配置

<img src="/images/developer/starrocksdev/starrocks_create05.png" width="900">

6、设置登录密码，登录名称固定为“root”

<img src="/images/developer/starrocksdev/starrocks_create06.png" width="900">

7、设置集群名称等信息

<img src="/images/developer/starrocksdev/starrocks_create07.png" width="900">

8、最后确认各项配置与金额，确认后进行支付

<img src="/images/developer/starrocksdev/starrocks_create08.png" width="900">

9、支付完成后，回到集群列表，待集群创建完成后，即可进行使用。

<img src="/images/developer/starrocksdev/starrocks_create09.png" width="900">


## 使用示例

1. SSH登录StarRocks的Master节点。

2. 通过mysql命令连接StarRocks集群。

   ```
   mysql -uroot -h127.0.0.1 -P 9030 -p
   ```

   + 密码是创建实例时设置的密码

3. 执行以下SQL，创建数据库并选择数据库

   ```
   CREATE DATABASE IF NOT EXISTS test;
   USE test;
   ```

4. 执行以下SQL，创建表

   ```
   CREATE TABLE `user_events` (
       `event_id` BIGINT,
       `event_time` DATETIME,
       `user_id` INT,
       `event_type` VARCHAR(50),
       `amount` DECIMAL(10, 2),
       `updated_at` DATETIME
   )
   PRIMARY KEY (`event_id`, `event_time`)
   PARTITION BY RANGE (`event_time`) (
       PARTITION p202601 VALUES [('2026-01-01'), ('2026-02-01')),
       PARTITION p202602 VALUES [('2026-02-01'), ('2026-03-01')),
       PARTITION p202603 VALUES [('2026-03-01'), ('2026-04-01'))
   )
   DISTRIBUTED BY HASH(`event_id`) BUCKETS 8
   PROPERTIES (
       "replication_num" = "3",
       "enable_persistent_index" = "true"
   );
   ```

5. 执行以下sql插入数据

   ```
   INSERT INTO `user_events` VALUES
   (1001, '2026-03-05 10:30:00', 501, 'purchase', 99.90, '2026-03-05 10:30:00'),
   (1002, '2026-03-05 11:15:00', 502, 'refund', 49.50, '2026-03-05 11:15:00'),
   (1003, '2026-02-18 09:00:00', 501, 'purchase', 199.00, '2026-02-18 09:00:00'),
   (1004, '2026-01-10 08:00:00', 503, 'login', 0.00, '2026-01-10 08:00:00'),
   (1005, '2026-03-05 14:20:00', 504, 'purchase', 59.90, '2026-03-05 14:20:00');
   ```

6. 执行以下sql查询数据

   ```
   select * from user_events;
   ```

   

## 数据导入

### Broker Load

1. 创建导入任务

   语法：

   ```
   LOAD LABEL [<database_name>.]<label_name>
   (
       data_desc[, data_desc ...]
   )
   WITH BROKER
   (
       StorageCredentialParams
   )
   [PROPERTIES
   (
       opt_properties
   )
   ]
   ```

   例子：使用mysql命令连接starrocks之后执行如下sql

   ```
   LOAD LABEL user_events_hdfs_20260309
   (
       DATA INFILE("hdfs://<namenode_host>:<fs_port>/data/user_events.csv")
       INTO TABLE test.user_events
       COLUMNS TERMINATED BY ","
       FORMAT AS "CSV" (skip_header = 1)
       (
           event_id,
           event_time,
           user_id,
           event_type,
           amount,
           updated_at
       )
   )
   WITH BROKER (
       "hadoop.security.authentication" = "simple",
       "username" = "<hdfs_用户名>",
       "password" = "<hdfs_密码>"
   )
   PROPERTIES (
       "timeout" = "3600",
       "max_filter_ratio" = "0.1"
   );
   ```

   + hadoop若无密码可不写此字段
   + user_events_hdfs_20260309为此次任务的标签

2. 查看任务状态

   ```
   SELECT * FROM information_schema.loads WHERE LABEL='user_events_hdfs_20260309'\G;
   ```

   ```
   mysql> SELECT * FROM information_schema.loads WHERE LABEL='user_events_hdfs_20260309'\G;
   *************************** 1. row ***************************
                     ID: 10427
                  LABEL: user_events_hdfs_20260309
             PROFILE_ID: NULL
                DB_NAME: test
             TABLE_NAME: user_events
                   USER: root
              WAREHOUSE:
                  STATE: FINISHED
               PROGRESS: ETL:100%; LOAD:100%
                   TYPE: BROKER
               PRIORITY: NORMAL
              SCAN_ROWS: 1000
             SCAN_BYTES: 64326
          FILTERED_ROWS: 0
        UNSELECTED_ROWS: 0
              SINK_ROWS: 1000
        RUNTIME_DETAILS: {"backends": {"32f5a74c-85f8-440d-8048-1a64a25622fa": [10002]}, "file_size": 64326, "load_id": "32f5a74c-85f8-440d-8048-1a64a25622fa", "task_num": 1, "txn_id": 7, "unfinished_backends": {"32f5a74c-85f8-440d-8048-1a64a25622fa": []}}
            CREATE_TIME: 2026-03-09 17:02:56
        LOAD_START_TIME: 2026-03-09 17:03:02
       LOAD_COMMIT_TIME: 2026-03-09 17:03:04
       LOAD_FINISH_TIME: 2026-03-09 17:03:04
             PROPERTIES: {"max_filter_ratio": 0.1, "timeout": 3600}
              ERROR_MSG: NULL
           TRACKING_SQL: NULL
   REJECTED_RECORD_PATH: NULL
                 JOB_ID: 10427
   1 row in set (0.02 sec)
   ```

3. 取消任务

   当导入作业状态不为 CANCELLED 或 FINISHED 时，可以通过 CANCEL LOAD 语句来取消该导入作业。

   ```
   CANCEL LOAD
   FROM test
   WHERE LABEL = "user_events_hdfs_20260309";
   ```

   + test为导入的库名

### Stream Load

#### 导入csv数据文件

Stream可以在客户端上通过指定文件同步导入数据到StarRocks集群。

1. 提交导入

```
curl --location-trusted -u user:password \
  -H "label:user_events_stream_20260309" \
  -H "Expect:100-continue" \
  -H "column_separator:," \
  -H "skip_header:1" \
  -H "columns: event_id, event_time, user_id, event_type, amount, updated_at" \
  -T "./user_events.csv" \
  -XPUT "http://10.9.94.203:8030/api/test/user_events/_stream_load"
```

+ 需要指定starrocks的用户和密码
+ -T指定需要导入的数据文件路径
+ 10.9.94.203是StarRocks FE节点IP。客户端、FE、BE或CN节点需要在同一网段下。FE会把客户端的请求路由到BE或CN节点上。

2. 返回结果

   ```
   {
       "TxnId": 21,
       "Label": "user_events_stream_20260309",
       "Status": "Success",
       "Message": "OK",
       "NumberTotalRows": 1000,
       "NumberLoadedRows": 1000,
       "NumberFilteredRows": 0,
       "NumberUnselectedRows": 0,
       "LoadBytes": 64326,
       "LoadTimeMs": 94,
       "BeginTxnTimeMs": 11,
       "StreamLoadPlanTimeMs": 17,
       "ReadDataTimeMs": 0,
       "WriteDataTimeMs": 36,
       "CommitAndPublishTimeMs": 28
   }
   ```


#### 导入json数据文件

2. 1. 提交导入

      ```
      curl --location-trusted -u root:4277813a. \
        -H "label:user_events_json_stream_20260309" \
        -H "Expect:100-continue" \
        -H "format: json" \
        -H "strip_outer_array: false" \
        -H "columns: event_id, event_time, user_id, event_type, amount, updated_at" \
        -T "./user_events_sample.json" \
        -XPUT "http://10.9.94.203:8030/api/test/user_events/_stream_load"
      ```

   2. 返回结果

      ```
      {
          "TxnId": 28,
          "Label": "user_events_json_stream_20260309",
          "Status": "Success",
          "Message": "OK",
          "NumberTotalRows": 5,
          "NumberLoadedRows": 5,
          "NumberFilteredRows": 0,
          "NumberUnselectedRows": 0,
          "LoadBytes": 693,
          "LoadTimeMs": 56,
          "BeginTxnTimeMs": 1,
          "StreamLoadPlanTimeMs": 3,
          "ReadDataTimeMs": 0,
          "WriteDataTimeMs": 29,
          "CommitAndPublishTimeMs": 21
      }
      ```

### Spark connector

1. 根据spark版本选择connector版本。下载地址：https://repo1.maven.org/maven2/com/starrocks/

2. | Connector | Spark              | StarRocks | Java | Scala |
   | --------- | ------------------ | --------- | ---- | ----- |
   | 1.1.2     | 3.2、3.3、3.4、3.5 | 2.5+      | 8    | 2.12  |
   | 1.1.1     | 3.2、3.3、3.4      | 2.5+      | 8    | 2.12  |
   | 1.1.0     | 3.2、3.3、3.4      | 2.5+      | 8    | 2.12  |

   将starrocks-spark-connector-xxxx.jar放入spark的jar目录下，例如：/home/hadoop/spark/jars/

3. spark连接starrocks还需要jdbc包。推荐版本：mysql-connector-java-8.0.22.jar。下载地址：https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.22/ 同样放入到spark的jar目录下

4. 写入数据，这里使用spark sql写入数据举例

   ```
   spark-sql> CREATE TEMPORARY VIEW user_events
              USING starrocks
              OPTIONS
              (
                  "starrocks.table.identifier" = "test.user_events",
                  "starrocks.fe.http.url" = "<fe_host>:<fe_http_port>",
                  "starrocks.fe.jdbc.url" = "jdbc:mysql://<fe_host>:<fe_query_port>",
                  "starrocks.user" = "root",
                  "starrocks.password" = ""
              );
   ```

   - starrocks.table.identifier：starrocks侧的表名，格式：库名.表名
   - fe_http_port：默认8030
   - fe_query_port：默认9030

   插入数据

   ```
   INSERT INTO `user_events` VALUES
   (1001, '2026-03-05 10:30:00', 501, 'purchase', 99.90, '2026-03-05 10:30:00');
   ```

   

### Kafka connector

1. 前提：已经存在一个kafka集群。

2. 创建topic并写入测试数据

   ```
   # 创建 Topic
   bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092
   
   # 写入测试数据
   bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
   {"id":1,"city":"New York"}
   {"id":2,"city":"Los Angeles"}
   {"id":3,"city":"Chicago"}
   ```

3. 下载connector。下载地址：https://github.com/StarRocks/starrocks-connector-for-kafka/releases

4. 使用mysql连接StarRocks，并建表

   ```sql
   CREATE DATABASE example_db;
   USE example_db;
   CREATE TABLE test_tbl (id INT, city STRING);
   ```

3. 配置Kafka Connector。在 Kafka 的 `config` 目录下创建 `connect-starrocks-sink.properties`

   ```
   name=starrocks-kafka-connector
   connector.class=com.starrocks.connector.kafka.StarRocksSinkConnector
   topics=test
   key.converter=org.apache.kafka.connect.json.JsonConverter
   value.converter=org.apache.kafka.connect.json.JsonConverter
   key.converter.schemas.enable=true
   value.converter.schemas.enable=false
   # StarRocks FE 的 HTTP 地址
   starrocks.http.url=<starrocks_fe_ip>:8030
   # Topic 名称与 StarRocks 表名不一致时需要配置映射
   starrocks.topic2table.map=test:test_tbl
   starrocks.username=root
   starrocks.password=
   starrocks.database.name=example_db
   sink.properties.strip_outer_array=true
   ```

4. 配置Kafka Connect。编辑 `config/connect-standalone.properties`

   ```
   bootstrap.servers=localhost:9092
   offset.storage.file.filename=/tmp/connect.offsets
   offset.flush.interval.ms=10000
   key.converter=org.apache.kafka.connect.json.JsonConverter
   value.converter=org.apache.kafka.connect.json.JsonConverter
   key.converter.schemas.enable=true
   value.converter.schemas.enable=false
   # connector 解压后的路径
   plugin.path=/home/kafka-connect/starrocks-kafka-connector-1.0.4
   ```

5. 启动Kafka Connect

   ```
   cd kafka
   CLASSPATH=/home/kafka-connect/starrocks-kafka-connector-1.0.4/* \
     bin/connect-standalone.sh \
     config/connect-standalone.properties \
     config/connect-starrocks-sink.properties
   ```

6. 验证数据

   ```
   mysql> SELECT * FROM example_db.test_tbl;
   +------+-------------+
   | id   | city        |
   +------+-------------+
   |    1 | New York    |
   |    2 | Los Angeles |
   |    3 | Chicago     |
   +------+-------------+
   ```

### Flink Connector

1. 根据flink版本选择connector版本。下载地址：https://github.com/StarRocks/starrocks-connector-for-apache-flink/releases

   | Connector | Flink                    | StarRocks | Java | Scala     |
   | --------- | ------------------------ | --------- | ---- | --------- |
   | 1.2.10    | 1.15,1.16,1,17,1,18,1.19 | 2.1+      | 8    | 2.11,2.12 |
   | 1.2.9     | 1.15,1.16,1,17,1,18      | 2.1+      | 8    | 2.11,2.12 |
   | 1.2.8     | 1.13,1.14,1.15,1.16,1,17 | 2.1+      | 8    | 2.11,2.12 |
   | 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1+      | 8    | 2.11,2.12 |

2. 将flink-connector-starrocks-xxxx.jar放到flink的lib目录下,比如/home/hadoop/flink/lib/

3. flink连接starrocks还需要jdbc包。推荐版本：mysql-connector-java-8.0.22.jar。下载地址：https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.22/ 同样放入到flink的lib目录下

4. 写入数据。这里使用fink sql举例

   在flink sql中创建starrocks对应表的映射表

   ```
   CREATE TABLE user_events_sink (
       event_id       BIGINT,
       event_time     TIMESTAMP(0),
       user_id        INT,
       event_type     STRING,
       amount         DECIMAL(10, 2),
       updated_at     TIMESTAMP(0)
   ) WITH (
       'connector' = 'starrocks',
       'jdbc-url' = 'jdbc:mysql://<fe_host>:9030/<database>',
       'load-url' = '<fe_host>:8030',
       'database-name' = '<your_database>',
       'table-name' = 'user_events',
       'username' = 'root',
       'password' = ''
   );
   ```

   插入数据

   ```
   INSERT INTO `user_events` VALUES
   (1001, '2026-03-05 10:30:00', 501, 'purchase', 99.90, '2026-03-05 10:30:00');
   ```

### Insert Into

1. 使用mysql命令登录StarRocks集群

## 数据分析

### Hive 数据源

1. 将hadoop的 `hdfs-site.xml、core-site.xml` 这两个文件放到每个节点的 /home/starrocks/starrocks/fe/conf 和 /home/starrocks/starrocks/be/conf 目录下

2. 在starrocks的/home/starrocks/starrocks/fe/conf 和 /home/starrocks/starrocks/be/conf 下的 hadoop_env.sh`文件开头添加`

   ```
   export HADOOP_USER_NAME="hadoop"
   ```

3. 在 fe、be 节点上把 hadoop 集群所有节点的 ip/hostname 配置到`/etc/hosts` 文件中

4. 重启fe节点和be或cn节点

   ```
   service starrocks-fe restart
   
   service starrocks-be restart
   
   service starrocks-cn restart
   ```

5. 使用mysql命令登录StarRocks集群，执行以下命令创建Hive类型的catalog

   ```
   CREATE EXTERNAL CATALOG hive_catalog
   PROPERTIES (
       "type" = "hive",
       "hive.metastore.type" = "hive",
       "hive.metastore.uris" = "thrift://xxx:9083,thrift://xxx:9083"
   );
   ```

   - type：catalog类型
   - hive.metastore.type：元数据管理类型
   - hive.metastore.uris：hive hms地址

6. 在StarRocks中查询Hive中的数据

   切换catalog到hive_catalog

   ```
   set catalog hive_catalog;
   ```

   查询Hive test库下的test_table表

   ```
   SELECT * FROM test.test_table;
   ```

   或者使用完整路径

   ```
   SELECT * FROM hive_catalog.test.test_table;
   ```

   结果

   ```
   mysql> SELECT * FROM hive_catalog.test.test_table;
   +------+--------------+------+
   | id   | name         | age  |
   +------+--------------+------+
   |    1 | Zhang San    |   25 |
   |    2 | Li Si        |   30 |
   |    3 | Wang Wu      |   28 |
   |    4 | Zhao Liu     |   35 |
   |    5 | Qian Qi      |   22 |
   |    6 | Sun Ba       |   27 |
   |    7 | Zhou Jiu     |   31 |
   |    8 | Wu Shi       |   29 |
   |    9 | Zheng Shi Yi |   26 |
   |   10 | Wang Shi Er  |   33 |
   +------+--------------+------+
   10 rows in set (0.03 sec)
   ```

### Iceberg数据源

查询hive管理的iceberg表

1. mysql登录StarRocks集群

2. 创建创建iceberg类型的catalog

   ```
   CREATE EXTERNAL CATALOG iceberg_hive_catalog
   PROPERTIES (
       "type" = "iceberg",
       "iceberg.catalog.type" = "hive",
       "iceberg.catalog.hive.metastore.uris" = "thrift://10.9.103.239:9083,thrift://10.9.129.223:9083"
   );
   ```

   + type：catalog类型是iceberg
   + iceberg.catalog.type：catalog元数据管理类型
   + hive.metastore.uris：hive hms地址

3. 在StarRocks中查询Iceberg表数据

   ```
   mysql> select * from iceberg_hive_catalog.test.iceberg_test;
   +------+--------------+------+
   | id   | name         | age  |
   +------+--------------+------+
   |    1 | Zhang San    |   25 |
   |    2 | Li Si        |   30 |
   |    3 | Wang Wu      |   28 |
   |    4 | Zhao Liu     |   35 |
   |    5 | Qian Qi      |   22 |
   |    6 | Sun Ba       |   27 |
   |    7 | Zhou Jiu     |   31 |
   |    8 | Wu Shi       |   29 |
   |    9 | Zheng Shi Yi |   26 |
   |   10 | Wang Shi Er  |   33 |
   +------+--------------+------+
   ```

   

### US3数据源

这里是指iceberg表的元数据和data数据都存放在US3中，且hadoop中集成了US3数据源。就可以使用一下方法查询到存在US3中的IceBerg表。

前提：

1. 存储hadoop集群已按https://docs.ucloud.cn/uhadoop/developer/us3集成us3

2. 参考Hive数据源的步骤1和步骤2配置StarRocks

3. 在StarRocks中创建catalog

   ```
   CREATE EXTERNAL CATALOG iceberg_us3_catalog
   PROPERTIES (
       "type" = "iceberg",
       "iceberg.catalog.type" = "hadoop", 
       "iceberg.catalog.warehouse" = "us3://${s3-bucket}/warehouse" 
   );
   ```

   + type：catalog类型
   + iceberg.catalog.type：元数据管理类型
   + 

4. 在 StarRocks 中查询catalog中的数据

   ```
   mysql> select * from iceberg_us3_catalog.test.iceberg_test;
   +------+--------------+------+
   | id   | name         | age  |
   +------+--------------+------+
   |    1 | Zhang San    |   25 |
   |    2 | Li Si        |   30 |
   |    3 | Wang Wu      |   28 |
   |    4 | Zhao Liu     |   35 |
   |    5 | Qian Qi      |   22 |
   |    6 | Sun Ba       |   27 |
   |    7 | Zhou Jiu     |   31 |
   |    8 | Wu Shi       |   29 |
   |    9 | Zheng Shi Yi |   26 |
   |   10 | Wang Shi Er  |   33 |
   +------+--------------+------+
   ```

   

### Hudi数据源

查询hive管理的hudi表

1. mysql登录StarRocks集群

2. 创建创建iceberg类型的catalog

   ```
   CREATE EXTERNAL CATALOG hudi_catalog_hms
   PROPERTIES
   (
       "type" = "hudi",
       "hive.metastore.type" = "hive",
       "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
   );
   ```

   + type：catalog类型是iceberg
   + iceberg.catalog.type：catalog元数据管理类型
   + hive.metastore.uris：hive hms地址

3. 在StarRocks中查询Iceberg表数据

   ```
   mysql> set catalog hudi_catalog_hms;
   mysql> use default;
   mysql> select _hoodie_record_key,_hoodie_partition_path,ts from hudi_table_spark;
   +-----------------------+------------------------+---------------+
   | _hoodie_record_key    | _hoodie_partition_path | ts            |
   +-----------------------+------------------------+---------------+
   | 20260313112935347_4_0 | city=sao_paulo         | 1695516137016 |
   | 20260313112935347_5_0 | city=sao_paulo         | 1695376420876 |
   | 20260313112935347_1_0 | city=san_francisco     | 1695091554788 |
   | 20260313112935347_3_0 | city=san_francisco     | 1695332066204 |
   | 20260313112935347_0_0 | city=san_francisco     | 1695159649087 |
   | 20260313112935347_2_0 | city=san_francisco     | 1695046462179 |
   | 20260313112935347_7_0 | city=chennai           | 1695115999911 |
   | 20260313112935347_6_0 | city=chennai           | 1695173887231 |
   +-----------------------+------------------------+---------------+
   8 rows in set (0.09 sec)

### 内表数据源

1. 使用mysql登录starrocks之后，默认就是查询的内表数据，即default_catalog。

2. 查询内表数据

   ```
   mysql> select * from test.user_events;
   +----------+---------------------+---------+------------+--------+---------------------+
   | event_id | event_time          | user_id | event_type | amount | updated_at          |
   +----------+---------------------+---------+------------+--------+---------------------+
   |     1004 | 2026-01-10 08:00:00 |     503 | login      |   0.00 | 2026-01-10 08:00:00 |
   |     1002 | 2026-03-05 11:15:00 |     502 | refund     |  49.50 | 2026-03-05 11:15:00 |
   |     1005 | 2026-03-05 14:20:00 |     504 | purchase   |  59.90 | 2026-03-05 14:20:00 |
   |     1001 | 2026-03-05 10:30:00 |     501 | purchase   |  99.90 | 2026-03-05 10:30:00 |
   |     1003 | 2026-02-18 09:00:00 |     501 | purchase   | 199.00 | 2026-02-18 09:00:00 |
   +----------+---------------------+---------+------------+--------+---------------------+
   ```

   
