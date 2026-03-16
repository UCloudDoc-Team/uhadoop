## Spark connector

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
