# StarRocks version 2.0

## 2.0.0

发布日期：2022 年 1 月 5 日

### New Feature

- 外表
  - [实验功能] 支持 S3 上的 Hive 外表功能 [参考文档](/using_starrocks/External_table.md#Hive外表)
  - DecimalV3 支持外表查询 [#425](https://github.com/StarRocks/starrocks/pull/425)
- 实现存储层复杂表达式下推计算，获得性能提升
- Broker Load 支持华为 OBS [#1182](https://github.com/StarRocks/starrocks/pull/1182)
- 支持国密算法 sm3
- 适配 ARM 类国产 CPU：通过鲲鹏架构验证
- 主键模型（Primary Key）正式发布，该模型支持 Stream Load、Broker Load、Routine Load，同时提供了基于 Flink-cdc 的 MySQL 数据的秒级同步工具。[参考文档](/table_design/Data_model.md#主键模型)

### Improvement

- 优化算子性能
  - 低基数字典性能优化 [#791](https://github.com/StarRocks/starrocks/pull/791)
  - 单表的 int scan 性能优化 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 高基数下的 `count(distinct int)` 性能优化 [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250)  [#544](https://github.com/StarRocks/starrocks/pull/544) [#570](https://github.com/StarRocks/starrocks/pull/570)
  - 执行层优化和完善 `Group by 2 int` / `limit` / `case when` / `not equal`
- 内存管理优化
  - 重构内存统计/控制框架，精确统计内存使用，彻底解决 OOM
  - 优化元数据内存使用
  - 解决大内存释放长时间卡住执行线程的问题
  - 进程优雅退出机制，支持内存泄漏检查 [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### Bugfix

- 修复 Hive 外表大量获取元数据超时的问题
- 修复物化视图创建报错信息不明确的问题
- 修复向量化引擎对 `like` 的实现 [#722](https://github.com/StarRocks/starrocks/pull/722)
- 修复 `alter table` 中谓词 is 的解析错误 [#725](https://github.com/StarRocks/starrocks/pull/725)
- 修复 `curdate` 函数没办法格式化日期的问题

## 2.0.1

发布日期： 2022 年 1 月 21 日

### Imporvement

- 优化 StarRocks 读取 Hive 外表时 Hive 外表隐式数据转换的功能。 [#2829](https://github.com/StarRocks/starrocks/pull/2829)
- 优化高并发查询场景下，StarRocks CBO 优化器采集统计信息时的锁竞争问题。 [#2901](https://github.com/StarRocks/starrocks/pull/2901)
- 优化 CBO 的统计信息工作，UNION 算子等。

### Bugfix

- 修复副本的全局字典不一致而引起查询的问题。 [#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- 修复数据导入至 StarRocks 前设置参数 `exec_mem_limit` 不生效的问题。 [#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > 参数 `exec_mem_limit` 用于指定数据导入时单个 BE 节点计算层使用的内存上限。
- 修复数据导入至 StarRocks 主键模型时触发 OOM 的问题。 [#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- 修复 StarRocks 在查询大数量级的 MySQL 外部表时的查询卡死问题。 [#2881](https://github.com/StarRocks/starrocks/pull/2881)

### Behavior Change

- StarRocks 支持使用 Hive 外表访问创建在 Hive 外表上的 Amazon S3 外表。由于用于访问 Amazon S3 外表的 jar 包较大，因此 StarRocks 二进制产品包目前暂未包含该 jar 包。如有需要，请单击 [Hive_s3_lib](https://cdn-thirdparty.starrocks.com/hive_s3_jar.tar.gz) 进行下载。
