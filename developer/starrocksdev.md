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