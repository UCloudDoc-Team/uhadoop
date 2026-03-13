## Flink Connector

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
