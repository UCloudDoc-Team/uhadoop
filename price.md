

# 产品价格

UHadoop产品包含两种计费模式：

1\. 按配置计费：按照CPU、内存、存储3个维度进行计费；

2\. 按节点计费：按照节点维度根据节点配置进行整体计费；

其中按节点计费方式仅提供部分存量实例使用，新实例均采取按配置计费方式。


> 说明：文档中的价格以华北（北京2）可用区E为例，其他可用区价格可查看[UHadoop控制台](https://console.ucloud.cn/uhadoop/create)。

## 1. 按配置计费

产品价格按照CPU、内存、存储3个维度进行计费，详细价格如下：

### 1.1 CPU与内存

| 计费项           | 华北（北京2）可用区E 月单价 |
| ---------------- | -------------------- |
| CPU（元/核/月）  | 71                   |
| 内存（元/GB/月） | 18                   |

  常用机型（不含存储）价格如下：


| CPU内存比 | 名称                | 配置      | 华北（北京2）可用区E 月单价（元/月） |
| --------- | ------------------- | --------- | ----------------------------- |
| 1:2       | o.hadoop2m.xlarge   | 4核 8G    | 428                           |
| 1:2       | o.hadoop2m.2xlarge  | 8核 16G   | 856                           |
| 1:2       | o.hadoop2m.4xlarge  | 16核 32G  | 1712                          |
| 1:2       | o.hadoop2m.8xlarge  | 32核 64G  | 3424                          |
| 1:2       | o.hadoop2m.16xlarge | 64核128G  | 6848                          |
| 1:4       | o.hadoop4m.medium   | 2核 8G    | 286                           |
| 1:4       | o.hadoop4m.xlarge   | 4核 16G   | 572                           |
| 1:4       | o.hadoop4m.2xlarge  | 8核 32G   | 1144                          |
| 1:4       | o.hadoop4m.4xlarge  | 16核 64G  | 2288                          |
| 1:4       | o.hadoop4m.8xlarge  | 32核 128G | 4576                          |
| 1:4       | o.hadoop4m.16xlarge | 64核 256G | 9152                          |

### 1.2 存储

| 计费项 | 华北（北京2）可用区E 月单价（元/GB/月） |
| ------ | -------------------------------- |
| 存储   | 0.75                             |

## 2. 按节点计费

产品价格根据节点类型及配置不同，详细价格如下：

| 节点类型            | 机型            | 名称                  | CPU | 内存(G) | 硬盘(G)            | 华北（北京2）E价格(元/月) |
| --------------- | ------------- | ----------------- | --- | ----- | ---------------- | ----------- |
| Master&​Task ​ | 计算优化实例 ​ | C1-large | 2 | 4 | 40 | 198 |
| Master&​Task ​   | 计算优化实例 ​     | C1-xlarge ​       | 4   | 8     | 80(SATA) ​       | 398         |
| Master&​Task ​  | 计算优化实例 ​      | C1-2xlarge ​      | 8   | 16    | 160(SATA) ​      | 798         |
| Master&​Task ​  | 计算优化实例 ​      | C1-4xlarge ​      | 16  | 32    | 320(SATA) ​      | 1598        |
| Master&​Task ​  | 内存优化实例 ​      | R1-large ​        | 2   | 16    | 40(SATA) ​       | 419         |
| Master&​Task ​  | 内存优化实例 ​      | R1-3xlarge ​      | 16  | 64    | 400(SATA) ​      | 2013        |
| Master&​Task ​  | 内存优化实例 ​      | R1-6xlarge ​     | 32  | 128 | 800(SATA) ​      | 4700    |
|                 |                     |                   |     |       |                  |             |
| Master&​Task ​  | 计算优化Ⅱ型实例 ​    | H1-large ​        | 4   | 8     | 100(SSD) ​       | 508         |
| Master&​Task ​  | 计算优化Ⅱ型实例 ​    | H1-4xlarge ​      | 32  | 64    | 500(SSD) ​       | 4300        |
|                 |               |                   |     |       |                  |             |
| Master&​Task ​  | 内存优化Ⅱ型实例 ​    | H2-2xlarge ​      | 16  | 64    | 400(SSD) ​       | 2249        |
|                 |              |                   |     |       |                  |             |
| Master&​Task ​  | RSSD快杰机型 ​    | N4-large ​        | 4   | 16    | 100(RSSD) ​       | 590         |
| Master&​Task ​  | RSSD快杰机型 ​    | N4-xlarge ​       | 8   | 32    | 200(RSSD) ​       | 1180        |
| Master&​Task ​  | RSSD快杰机型 ​    | N4-2xlarge ​      | 16  | 64    | 400(RSSD) ​       | 2360        |
| Master&​Task ​  | RSSD快杰机型 ​    | N4-4xlarge ​      | 32  | 128    | 1000(RSSD) ​      | 4624        |
|                 |              |                   |     |       |                  |             |
| Core            | RSSD快杰机型 ​      | N3-large ​        | 2   | 8     | 500(RSSD) ​     | 565         |
| Core            | RSSD快杰机型 ​      | N3-xlarge ​       | 4   | 16    | 1000(RSSD) ​     | 1130         |
| Core            | RSSD快杰机型 ​      | N3-2xlarge ​      | 8   | 32    | 2000(RSSD) ​    | 2260        |
| Core            | RSSD快杰机型 ​      | N3-4xlarge ​      | 16  | 64    | 4000(RSSD) ​    | 4520        |
| Core            | RSSD快杰机型 ​      | N3-8xlarge ​      | 32  | 128   | 8000(RSSD) ​    | 8824        |
|                 |              |                   |     |       |                  |             |
| Core            | 密集存储实例 ​      | D1-large ​        | 2   | 6     | 4000(SATA) ​     | 498         |
| Core            | 密集存储实例 ​      | D1-xlarge ​       | 4   | 12    | 8000(SATA) ​     | 998         |
|                 |              |                   |     |       |                  |             |
| Core            | 密集存储II型实例 ​   | D2-xlarge ​       | 4   | 12    | 16000(SATA) ​    | 1135        |
| Core            | 密集存储II型实例 ​   | D2-2xlarge ​      | 8   | 24    | 32000(SATA) ​    | 2235        |
| Core            | 密集存储II型实例 ​   | D2-4xlarge ​      | 16  | 48    | 64000(SATA) ​    | 4541        |
|                 |                  |                   |     |       |                  |             |
| Core            | HDFS存储型实例 ​   | HDFS-large ​      | 2   | 4     | 8000(SATA) ​     | 467         |
| Core            | HDFS存储型实例 ​   | HDFS-xlarge ​     | 4   | 8     | 16000(SATA) ​    | 934         |
|                 |              |                   |     |       |                  |             |
| Core            | I/​O增强型 ​     | F1-xlarge ​       | 8   | 32    | 1000(SSD) ​      | 1978        |
| Core            | I/​O增强型 ​     | F1-2xlarge ​      | 16  | 64    | 2000(SSD) ​      | 3998        |
|                 |                |                    |     |       |                  |             |
| Core            | 物理机 ​         | UHADOOP-4 ​       | 32  | 96    | 8000\*12(SATA) ​ | 6072        |
| Core & Master ​ | 物理机 ​         | UHADOOP-6 ​       | 48  | 192   | 4800(SSD) ​      | 8360        |

