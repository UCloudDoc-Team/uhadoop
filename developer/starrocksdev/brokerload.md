## Broker Load

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
