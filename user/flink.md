

# Flink

## 如何从Checkpoint启动任务？

Checkpoint 默认在作业取消后会被清除。如需从 Checkpoint 恢复作业，需要开启 Checkpoint，并配置作业取消后保留 Checkpoint，可在代码、作业提交命令（`-D` 参数）或 `/home/hadoop/flink/conf/flink-conf.yaml` 中添加以下配置：

```
execution.checkpointing.interval: 60s
execution.checkpointing.externalized-checkpoint-retention: RETAIN_ON_CANCELLATION
```

说明：集群的 `flink-conf.yaml` 中已配置默认的 Checkpoint 目录 `state.checkpoints.dir: hdfs://Ucluster/flink/flink-checkpoints`，无需再配置。

作业运行后，在 Checkpoint 目录下会生成 `<jobId>/chk-<n>` 形式的子目录，其中 `<n>` 为 Checkpoint 编号，选择编号最大的目录即为最新的 Checkpoint：

```
hdfs dfs -ls hdfs://Ucluster/flink/flink-checkpoints/<jobId>
```

使用 `-s` 参数指定 Checkpoint 路径重新提交作业：

* Per-Job 模式：

  ```
  flink run -t yarn-per-job --detached -s hdfs://Ucluster/flink/flink-checkpoints/<jobId>/chk-<n> /path/to/your-job.jar
  ```

* Application 模式：

  ```
  flink run-application -t yarn-application -s hdfs://Ucluster/flink/flink-checkpoints/<jobId>/chk-<n> /path/to/your-job.jar
  ```

通过 StreamPark 提交的作业，可直接在启动对话框中选择 Checkpoint 或 Savepoint 恢复，详见[StreamPark开发指南](/uhadoop/developer/streampark)。

## 如何从Savepoint启动任务？

### 触发Savepoint

* 在不停止作业的情况下触发 Savepoint：

  ```
  flink savepoint -t yarn-application -Dyarn.application.id=<applicationId> <jobId> hdfs://Ucluster/flink/flink-savepoints
  ```

* 停止作业并同时触发 Savepoint：

  ```
  flink stop -t yarn-application -Dyarn.application.id=<applicationId> --savepointPath hdfs://Ucluster/flink/flink-savepoints <jobId>
  ```

说明：集群的 `flink-conf.yaml` 中已配置默认的 Savepoint 目录 `state.savepoints.dir: hdfs://Ucluster/flink/flink-savepoints`。

命令执行成功后会返回 Savepoint 的路径，形如 `hdfs://Ucluster/flink/flink-savepoints/savepoint-<jobId前6位>-<随机串>`，请记录该路径。

说明：`<applicationId>` 和 `<jobId>` 的查看方式参考[Flink开发指南](/uhadoop/developer/flink)。

### 从Savepoint启动

使用 `-s` 参数指定 Savepoint 路径重新提交作业：

* Per-Job 模式：

  ```
  flink run -t yarn-per-job --detached -s hdfs://Ucluster/flink/flink-savepoints/savepoint-<xxxxxx>-<xxxxxxxxxxxx> /path/to/your-job.jar
  ```

* Application 模式：

  ```
  flink run-application -t yarn-application -s hdfs://Ucluster/flink/flink-savepoints/savepoint-<xxxxxx>-<xxxxxxxxxxxx> /path/to/your-job.jar
  ```

如果作业代码做过修改，删除了原作业中的部分算子，导致无法匹配 Savepoint 中的状态，提交时会报错，可在 `-s` 之后增加 `--allowNonRestoredState` 参数忽略无法匹配的状态。

## Checkpoint与Savepoint有什么区别？

| 对比项   | Checkpoint                                       | Savepoint                                        |
| -------- | ------------------------------------------------ | ------------------------------------------------ |
| 触发方式 | Flink 按配置的间隔自动触发                       | 用户手动触发                                     |
| 主要用途 | 作业故障时自动恢复                               | 作业升级、迁移、扩缩容等有计划的重启             |
| 保留策略 | 默认作业取消后清除，需配置才会保留               | 由用户管理，不会自动清除                         |
| 兼容性   | 修改作业代码或升级 Flink 版本后不保证可恢复      | 设计上支持作业代码调整，建议用于作业升级         |
