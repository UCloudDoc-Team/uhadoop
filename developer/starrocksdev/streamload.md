## Stream Load

### 导入csv数据文件

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


### 导入json数据文件

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
