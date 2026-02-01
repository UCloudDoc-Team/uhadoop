# Spark on GPU 开发指南

## 1. 简介
UHadoop GPU 计算节点是在 UHadoop CPU 架构产品能力基础上扩展的 GPU 加速计算节点类型，通过引入 GPU 计算资源，在不改变用户现有作业逻辑与使用方式的前提下，为 Spark、SQL、ETL、特征工程等数据处理与分析任务提供高性能计算支持。

产品面向大规模或高维度的复杂计算、模型训练、预测与推理等计算场景，通过 GPU 计算能力提升任务执行效率并优化单位算力成本。

## 2. 使用限制

1. Spark Rapids 支持大部分常用 DQL 算子，但尚未实现全量覆盖。当遇到不支持的算子时，系统将自动回退至 Spark 原生 CPU 算子执行，以保证作业正确性。
2. 部分 Spark Rapids 算子在计算行为上可能与 Spark 原生实现存在差异（如浮点数精度）。对结果一致性要求较高的场景，需提前验证，详细差异请参考[官方文档](https://nvidia.github.io/spark-rapids/docs/compatibility.html)。 
3. 目前 UHadoop 仅支持V100S机型（不支持 [Multi-Instance GPU](https://github.com/NVIDIA/spark-rapids-examples/tree/main/examples/MIG-Support)，Mig）。
4. 当前有GPU计算节点的集群仅支持Spark组件任务，提交其他组件任务可能会导致任务失败（如Hive、Flink、DolphinScheduler等）。


## 3. 创建集群

前置条件
- 已注册并登录 UCloud 控制台账号
- UHadoop GPU 计算节点需单独开通使用权限，请联系[技术支持](https://spt.ucloud.cn/)提交申请后再进行相关操作。

注意事项
> 当前 UHadoop GPU 计算节点仅在华北（北京2）可用区B、华北（乌兰察布）可用区A地域提供服务，支持 uhadoop 3.1 版本，暂时支持 Task 节点类型使用。

操作流程

步骤一. 登录[UHadoop控制台](https://console.ucloud.cn/uhadoop/create)，选择“创建集群”。

步骤二. 选择目标地域，Hadoop 框架的 3.1 发行版本，按需选择节点类型及GPU机型，并“立即购买”。
![](/images/image-20260129180556485.png)

步骤三. 跳转列表，点击目标“集群名称”进入详情页，打开“服务管理”标签页，查看Hadoop服务的webUI地址并点击访问，在 Yarn-UI 页面查看GPU资源配额情况。
![](/images/image-20260129170150529.png)

## 4. 使用

### 4.1 启动任务 
手动指定配置操作如下：

  ```java
  spark-shell \
    --conf spark.plugins=com.nvidia.spark.SQLPlugin \
    --conf spark.rapids.sql.concurrentGpuTasks=2 \
    --conf spark.executor.resource.gpu.amount=1 \
    --conf spark.executor.resource.gpu.discoveryScript=./getGpusResources.sh \
    --conf spark.resources.discoveryPlugin=com.nvidia.spark.ExclusiveModeGpuDiscoveryPlugin \
    --jars /home/hadoop/extralibs/rapids-4-spark_2.12-25.12.0.jar \
    --files $SPARK_HOME/examples/src/main/scripts/getGpusResources.sh 
  
  ```

### 4.2 确认是否已启用 GPU 资源 

1. 提交作业后，您可以通过 Yarn-UI 界面，便可以看到 GPU 资源已经被启用：
   ![image-20260129170636808](/images/image-20260129170636808.png)

2. 启动 spark-shell，执行如下语句，可以看到图中显示已经走了 GPU 的算子：

   ```java
   import org.apache.spark.sql.functions._
   import spark.implicits._
   
   // --------------------------
   // 构造示例 DataFrames
   // --------------------------
   val people1 = Seq(
     ("1", "xiaoming", 15),
     ("2", "xiaohong", 20),
     ("3", "xiaobi",   10),
     ("4", "xiaozhang", 18)
   )
   
   val people2 = Seq(
     ("2", "Beijing"),
     ("3", "Shanghai"),
     ("5", "Guangzhou")
   )
   
   val df1 = people1.toDF("id","name","age")
   val df2 = people2.toDF("id","city")
   
   df1.createOrReplaceTempView("t1")
   df2.createOrReplaceTempView("t2")
   
   // --------------------------
   // Filter + Select 验证
   // --------------------------
   println("===== select + filter =====")
   val dfFiltered = df1.select("name","age").filter($"age" > 10)
   dfFiltered.explain(true)   // 打印执行计划
   dfFiltered.show()          // 执行 & 显示结果
   
   // --------------------------
   // Join 操作验证
   // --------------------------
   println("===== inner join =====")
   val dfJoin = df1.join(df2, Seq("id"), "inner")
   dfJoin.explain(true)
   dfJoin.show()
   
   // --------------------------
   // 聚合 + 排序验证
   // --------------------------
   println("===== group by city with count =====")
   val dfAgg = dfJoin
     .groupBy($"city")
     .agg(count($"id").alias("total"))
     .orderBy($"total".desc)
   
   
   dfAgg.explain(true)
   dfAgg.show()
   ```
   ![image-20260129172224755](/images/image-20260129172224755.png)