# StarRocks version 1.19

## 1.19.0

发布日期：2021 年 10 月 25 日

### New Feature

* 实现 Global Runtime Filter，可以支持对 shuffle join 实现 Runtime filter。
* 默认开启 CBO Planner，完善了 colocated join / bucket shuffle / 统计信息等功能。[参考文档](/using_starrocks/Cost_based_optimizer.md)
* [实验功能] 发布主键模型（Primary Key）：为更好地支持实时/频繁更新功能，StarRocks 新增了一种表的类型: 主键模型。该模型支持 Stream Load、Broker Load、Routine Load，同时提供了基于 Flink-cdc 的 MySQL 数据的秒级同步工具。[参考文档](/table_design/Data_model.md#主键模型)
* [实验功能] 新增外表写入功能。支持将数据通过外表方式写入另一个 StarRocks 集群的表中，以解决读写分离需求，提供更好的资源隔离。[参考文档](/using_starrocks/External_table.md)

### Improvement

#### StarRocks

* 性能优化：
  * count distinct int 语句
  * group by int 语句
  * or 语句
* 优化磁盘 Balance 算法，单机增加磁盘后可以自动进行数据均衡。
* 支持部分列导出。 [参考文档](/unloading/Export.md)
* 优化 show processlist，显示具体 SQL。
* SET_VAR 支持多个变量设置。
* 完善更多报错信息，包括 table_sink、routine load、创建物化视图等。

#### StarRocks-Datax Connector

* StarRocks-DataX Writer 支持设置 interval flush。

### Bugfix

* 修复动态分区表在数据恢复作业完成后，新分区无法自动创建的问题。 [# 337](https://github.com/StarRocks/starrocks/issues/337)
* 修复 CBO 开启后 row_number 函数报错的问题。
* 修复统计信息收集导致 fe 卡死的问题。
* 修复 set_var 针对 session 生效而不是针对语句生效的问题。
* 修复 Hive 分区外表 `select count(*)` 返回异常的问题。

## 1.19.1

发布日期： 2021 年 11 月 2 日

### Improvement

* 优化 show frontends 的性能 [# 507](https://github.com/StarRocks/starrocks/pull/507) [# 984](https://github.com/StarRocks/starrocks/pull/984)
* 补充慢查询监控 [# 502](https://github.com/StarRocks/starrocks/pull/502) [# 891](https://github.com/StarRocks/starrocks/pull/891)
* 优化 hive 外表元数据获取，并行获取元数据 [# 425](https://github.com/StarRocks/starrocks/pull/425) [# 451](https://github.com/StarRocks/starrocks/pull/451)

### BugFix

* 修复 Thrift 协议兼容性问题，解决 hive 外表对接 Kerberos 的问题 [# 184](https://github.com/StarRocks/starrocks/pull/184) [# 947](https://github.com/StarRocks/starrocks/pull/947) [# 995](https://github.com/StarRocks/starrocks/pull/995) [# 999](https://github.com/StarRocks/starrocks/pull/999)
* 修复 view 创建的若干 bug [# 972](https://github.com/StarRocks/starrocks/pull/972) [# 987](https://github.com/StarRocks/starrocks/pull/987) [# 1001](https://github.com/StarRocks/starrocks/pull/1001)
* 修复 FE 无法灰度升级的问题 [# 485](https://github.com/StarRocks/starrocks/pull/485) [# 890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

发布日期： 2021 年 11 月 20 日

### Improvement

* bucket shuffle join 支持 right join 和 full outer join [# 1209](https://github.com/StarRocks/starrocks/pull/1209)  [# 31234](https://github.com/StarRocks/starrocks/pull/1234)

### Major Bugfix

* 修复 repeat node 无法进行谓词下推的问题 [# 1410](https://github.com/StarRocks/starrocks/pull/1410) [# 1417](https://github.com/StarRocks/starrocks/pull/1417)
* 修复 routine load 在集群切主场景下可能导入丢失数据的问题 [# 1074](https://github.com/StarRocks/starrocks/pull/1074) [# 1272](https://github.com/StarRocks/starrocks/pull/1272)
* 修复创建视图无法支持 union 的问题 [# 1083](https://github.com/StarRocks/starrocks/pull/1083)
* 修复一些 Hive 外表稳定性问题 [# 1408](https://github.com/StarRocks/starrocks/pull/1408)
* 修复一个 group by 视图的问题 [# 1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

发布日期： 2021 年 11 月 30 日

### Improvement

* 升级 jprotobuf 版本提升安全性 [# 1506](https://github.com/StarRocks/starrocks/issues/1506)

### Major Bugfix

* 修复部分 group by 结果正确性问题
* 修复 grouping sets 部分问题 [# 1395](https://github.com/StarRocks/starrocks/issues/1395) [# 1119](https://github.com/StarRocks/starrocks/pull/1119)
* 修复 date_format 的部分参数问题
* 修复一个聚合 streamming 的边界条件问题 [# 1584](https://github.com/StarRocks/starrocks/pull/1584)
* 详细内容参考 [链接](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)

## 1.19.4

发布日期： 2021 年 12 月 09 日

### Improvement

* 支持 cast(varchar as bitmap) [# 1941](https://github.com/StarRocks/starrocks/pull/1941)
* 更新 hive 外表访问策略 [# 1394](https://github.com/StarRocks/starrocks/pull/1394) [# 1807](https://github.com/StarRocks/starrocks/pull/1807)

Bugfix

* 修复带谓词 Cross Join 查询结果错误 bug [# 1918](https://github.com/StarRocks/starrocks/pull/1918)
* 修复 decimal 类型，time 类型转换 bug [# 1709](https://github.com/StarRocks/starrocks/pull/1709) [# 1738](https://github.com/StarRocks/starrocks/pull/1738)
* 修复 colocate join/replicate join 选错 bug [# 1727](https://github.com/StarRocks/starrocks/pull/1727)
* 修复若干 plan cost 计算问题

## 1.19.5

发布日期： 2021 年 12 月 20 日

### Imporvement

* 优化 shuffle join 的一个规划 [# 2184](https://github.com/StarRocks/starrocks/pull/2184)
* 优化多个大文件导入 [# 2067](https://github.com/StarRocks/starrocks/pull/2067)

### Bugfix

* 升级 Log4j2 到 2.17.0， 修复安全漏洞 [# 2284](https://github.com/StarRocks/starrocks/pull/2284) [# 2290](https://github.com/StarRocks/starrocks/pull/2290)
* 修复 Hive 外表的空分区的问题 [# 707](https://github.com/StarRocks/starrocks/pull/707) [# 2082](https://github.com/StarRocks/starrocks/pull/2082)
