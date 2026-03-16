## Iceberg数据源

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
