# 数据流和控制流

## 查询

用户可使用 MySQL 客户端连接 FE，执行 SQL 查询，获得结果。

查询流程如下：

* MySQL 客户端执行 DQL SQL 命令。
* FE 解析, 分析, 改写, 优化和规划, 生成分布式执行计划。
* 分布式执行计划由 若干个可在单台 be 上执行的 plan fragment 构成，FE 执行 exec\_plan\_fragment, 将 plan fragment 分发给 BE，并指定其中一台 BE 为 coordinator。
* BE 执行本地计算, 比如扫描数据。
* 其他 BE 调用 transimit\_data 将中间结果发送给 BE coordinator 节点汇总为最终结果。
* FE 调用 fetch\_data 获取最终结果。
* FE 将最终结果发送给 MySQL client。

执行计划在 BE 上的实际执行过程比较复杂, 采用向量化执行方式，比如一个算子产生 4096 个结果，输出到下一个算子参与计算，而非 batch 方式或者 one-tuple-at-a-time。

![query_plan](../assets/2.4.1-1.png)

## 数据导入

用户创建表之后, 导入数据填充表。

* 支持导入数据源有: 本地文件, HDFS, Kafka 和 S3。
* 支持导入方式有: 批量导入, 流式导入, 实时导入。
* 支持的数据格式有: CSV, Parquet, ORC 等。
* 导入发起方式有: 用 RESTful 接口, 执行 SQL 命令。

数据导入的流程如下:

* 用户选择一台 BE 作为协调者, 发起数据导入请求, 传入数据格式, 数据源和标识此次数据导入的 label, label 用于避免数据重复导入. 用户也可以向 FE 发起请求, FE 会把请求重定向给 BE。
* BE 收到请求后, 向 FE master 节点上报, 执行 loadTxnBegin, 创建全局事务。 因为导入过程中, 需要同时更新 base 表和物化索引的多个 bucket, 为了保证数据导入的一致性, 用事务控制本次导入的原子性。
* BE 创建事务成功后, 执行 streamLoadPut 调用, 从 FE 获得本次数据导入的计划. 数据导入, 可以看成是将数据分发到所涉及的全部的 tablet 副本上, BE 从 FE 获取的导入计划包含数据的 schema 信息和 tablet 副本信息。
* BE 从数据源拉取数据, 根据 base 表和物化索引表的 schema 信息, 构造内部数据格式。
* BE 根据分区分桶的规则和副本位置信息, 将发往同一个 BE 的数据, 批量打包, 发送给 BE, BE 收到数据后, 将数据写入到对应的 tablet 副本中。
* 当 BE coordinator 节点完成此次数据导入, 向 FE master 节点执行 loadTxnCommit, 提交全局事务, 发送本次数据导入的 执行情况, FE master 确认所有涉及的 tablet 的多数副本都成功完成, 则发布本次数据导入使数据对外可见, 否则, 导入失败, 数据不可见, 后台负责清理掉不一致的数据。

![load](../assets/2.4.2-1.png)

## 更改元数据

更改元数据的操作有: 创建数据库, 创建表, 创建物化视图, 修改 schema 等等. 这样的操作需要:

* 持久化到永久存储的设备上;
* 保证高可用, 复制到多个 FE 实例上, 避免单点故障;
* 有的操作需要在 BE 上生效, 比如创建表时, 需要在 BE 上创建 tablet 副本。

元数据的更新操作流程如下:

* 用户使用 MySQL client 执行 SQL 的 DDL 命令, 向 FE 的 master 节点发起请求; 比如: 创建表。
* FE 检查请求合法性, 然后向 BE 发起同步命令, 使操作在 BE 上生效; 比如: FE 确定表的列类型是否合法, 计算 tablet 的副本的放置位置, 向 BE 发起请求, 创建 tablet 副本。
* BE 执行成功, 则修改 FE 内存的 Catalog. 比如: 将 table, partition, index, tablet 的副本信息保存在 Catalog 中。
* FE 追加本次操作到 EditLog 并且持久化。
* FE 通过复制协议将 EditLog 的新增操作项同步到 FE 的 follower 节点。
* FE 的 follower 节点收到新追加的操作项后, 在自己的 Catalog 上按顺序播放, 使得自己状态追上 FE master 节点。

上述执行环节出现失败, 则本次元数据修改失败。

![meta_change](../assets/2.4.3-1.png)
