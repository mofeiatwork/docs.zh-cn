# StarRocks 的系统架构

## 系统架构图

![architecture](../assets/2.1-1.png)

## 组件介绍

StarRocks 集群由 FE 和 BE 构成, 可以使用 MySQL 客户端访问 StarRocks 集群。

### FE

FE 接收 MySQL 客户端的连接, 解析并执行 SQL 语句。

* 管理元数据, 执行 SQL DDL 命令, 用 Catalog 记录库, 表, 分区, tablet 副本等信息。
* FE 高可用部署, 使用复制协议选主和主从同步元数据, 所有的元数据修改操作, 由 FE leader 节点完成, FE follower 节点可执行读操作。 元数据的读写满足顺序一致性。  FE 的节点数目采用 2n+1, 可容忍 n 个节点故障。  当 FE leader 故障时, 从现有的 follower 节点重新选主, 完成故障切换。
* FE 的 SQL layer 对用户提交的 SQL 进行解析, 分析, 改写, 语义分析和关系代数优化, 生产逻辑执行计划。
* FE 的 Planner 负责把逻辑计划转化为可分布式执行的物理计划, 分发给一组 BE。
* FE 监督 BE, 管理 BE 的上下线, 根据 BE 的存活和健康状态, 维持 tablet 副本的数量。
* FE 协调数据导入, 保证数据导入的一致性。

### BE

* BE 管理 tablet 副本, tablet 是 table 经过分区分桶形成的子表, 采用列式存储。
* BE 受 FE 指导, 创建或删除子表。
* BE 接收 FE 分发的物理执行计划并指定 BE coordinator 节点, 在 BE coordinator 的调度下, 与其他 BE worker 共同协作完成执行。
* BE 读本地的列存储引擎获取数据, 并通过索引和谓词下沉快速过滤数据。
* BE 后台执行 compact 任务, 减少查询时的读放大。
* 数据导入时, 由 FE 指定 BE coordinator, 将数据以 fanout 的形式写入到 tablet 多副本所在的 BE 上。

### 其他组件

* 管理平台, 在后面会专门的章节介绍。
* Hdfs Broker:  用于从 Hdfs 中导入数据到 StarRocks 集群，见 [数据导入](../loading/Loading_intro.md) 章节。
