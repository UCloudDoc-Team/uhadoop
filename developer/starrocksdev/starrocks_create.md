# 如何创建StarRocks

1、在控制台选择“托管Hadoop集群 UHadoop”>“集群列表”，点击“创建集群”按钮进行创建

<img src="/images/developer/starrocksdev/starrocks_create01.png" width="900">

2、创建页选择地域、可用区

<img src="/images/developer/starrocksdev/starrocks_create02.png" width="900">

3、集群框架选择“StarRocks框架”

<img src="/images/developer/starrocksdev/starrocks_create03.png" width="900">

  应用场景有存算一体和存算分离两种可选：存算一体架构采用 BE 节点进行部署，数据将会存储在云盘中，同时可以增加 CN 节点辅助计算；存算分离架构采用 CN 节点进行部署，数据将会存储在另一 HDFS 集群或US3中。

<img src="/images/developer/starrocksdev/starrocks_create031.png" width="900">

当选择存算分离时，需要选择集群存储位置，可选择HDFS集群或US3两种方式，
<img src="/images/developer/starrocksdev/starrocks_create032.png" width="900">

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