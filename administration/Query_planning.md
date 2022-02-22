# 查询分析

性能优化是 StarRocks 集群管理员经常遇到的问题，慢查询不仅影响用户体验，而且影响整个集群的服务能力，所以定期针对慢查询进行分析并优化是一项非常重要的工作。

我们可以在 fe/log/fe.audit.log 中看到所有查询和慢查询信息，每个查询对应一个 QueryID，我们可以在页面或者日志中查找到查询对应的 QueryPlan 和 Profile，QueryPlan 是 FE 通过解析 SQL 生成的执行计划，而 Profile 是 BE 执行后的结果，包含了每一步的耗时和数据处理量等数据，可以通过 StarRocksManager 的图形界面看到可视化的 Profile 执行树。<br>
同时，StarRocks 还支持对慢查询中的 SQL 语句进行归类，并为各类 SQL 语句计算出 SQL 指纹（即 MD5 哈希值，对应字段为 `Digest`）。

## Plan 分析

SQL 语句在 StarRocks 中的生命周期可以分为查询解析(Query Parsing)、规划(Query Plan)、执行(Query Execution)三个阶段。对于 StarRocks 而言，查询解析一般不会成为瓶颈，因为分析型需求的 QPS 普遍不高。

决定 StarRocks 中查询性能的关键就在于查询规划(Query Plan)和查询执行(Query Execution)，二者的关系可以用一句话描述：Query Plan 负责组织算子(Join/Order/Aggregation)之间的关系，Query Exectuion 负责执行具体算子。

Query Plan 可以从宏观的角度提供给 DBA 一个视角，获取 Query 执行的相关信息。一个好的 Query Plan 很大程度上决定了查询的性能，所以 DBA 经常需要去查看 Query Plan，确定 Query Plan 是否生成得当。本章以 TPCDS 的 query96 为例，展示如何查看 StarRocks 的 Query Plan。

~~~ SQL
-- query96.sql
select  count(*)
from store_sales
    , household_demographics
    , time_dim
    , store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
~~~

Query Plan 可以分为逻辑执行计划(Logical Query Plan)，和物理执行计划(Physical Query Plan)，本章节所讲述的 Query Plan 默认指代的都是逻辑执行计划。StarRocks 中运行 EXPLAIN + SQL 就可以得到 SQL 对应的 Query Plan，TPCDS query96.sql 对应的 Query Plan 展示如下。

~~~sql
+------------------------------------------------------------------------------+
| Explain String                                                               |
+------------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                              |
|  OUTPUT EXPRS: <slot 11>                                                      |
|   PARTITION: UNPARTITIONED                                                   |
|   RESULT SINK                                                                |
|   12: MERGING-EXCHANGE                                                        |
|      limit: 100                                                              |
|      tuple ids: 5                                                            |
|                                                                              |
| PLAN FRAGMENT 1                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 12                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   8: TOP-N                                                                    |
|   |  order by: <slot 11> ASC                                                 |
|   |  offset: 0                                                               |
|   |  limit: 100                                                              |
|   |  tuple ids: 5                                                            |
|   |                                                                          |
|   7: AGGREGATE (update finalize)                                              |
|   |  output: count(*)                                                        |
|   |  group by:                                                               |
|   |  tuple ids: 4                                                            |
|   |                                                                          |
|   6: HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_store_sk` = `s_store_sk`                       |
|   |  tuple ids: 0 2 1 3                                                      |
|   |                                                                          |
|   |----11: EXCHANGE                                                           |
|   |       tuple ids: 3                                                       |
|   |                                                                          |
|   4: HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_hdemo_sk` = `household_demographics`.`hd_demo_sk`|
|   |  tuple ids: 0 2 1                                                        |
|   |                                                                          |
|   |----10: EXCHANGE                                                           |
|   |       tuple ids: 1                                                       |
|   |                                                                          |
|   2: HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: table not in same group                        |
|   |  equal join conjunct: `ss_sold_time_sk` = `time_dim`.`t_time_sk`         |
|   |  tuple ids: 0 2                                                          |
|   |                                                                          |
|   |----9: EXCHANGE                                                            |
|   |       tuple ids: 2                                                       |
|   |                                                                          |
|   0: OlapScanNode                                                             |
|      TABLE: store_sales                                                      |
|      PREAGGREGATION: OFF. Reason: `ss_sold_time_sk` is value column          |
|      partitions = 1/1                                                          |
|      rollup: store_sales                                                     |
|      tabletRatio = 0/0                                                         |
|      tabletList =                                                             |
|      cardinality =-1                                                          |
|      avgRowSize = 0.0                                                          |
|      numNodes = 0                                                              |
|      tuple ids: 0                                                            |
|                                                                              |
| PLAN FRAGMENT 2                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|                                                                              |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 11                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   5: OlapScanNode                                                             |
|      TABLE: store                                                            |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `store`.`s_store_name` = 'ese'                              |
|      partitions = 1/1                                                          |
|      rollup: store                                                           |
|      tabletRatio = 0/0                                                         |
|      tabletList =                                                             |
|      cardinality =-1                                                          |
|      avgRowSize = 0.0                                                          |
|      numNodes = 0                                                              |
|      tuple ids: 3                                                            |
|                                                                              |
| PLAN FRAGMENT 3                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 10                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   3: OlapScanNode                                                             |
|      TABLE: household_demographics                                           |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `household_demographics`.`hd_dep_count` = 5                 |
|      partitions = 1/1                                                          |
|      rollup: household_demographics                                          |
|      tabletRatio = 0/0                                                         |
|      tabletList =                                                             |
|      cardinality =-1                                                          |
|      avgRowSize = 0.0                                                          |
|      numNodes = 0                                                              |
|      tuple ids: 1                                                            |
|                                                                              |
| PLAN FRAGMENT 4                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 09                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   1: OlapScanNode                                                             |
|      TABLE: time_dim                                                         |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `time_dim`.`t_hour` = 8, `time_dim`.`t_minute` >= 30        |
|      partitions = 1/1                                                          |
|      rollup: time_dim                                                        |
|      tabletRatio = 0/0                                                         |
|      tabletList =                                                             |
|      cardinality =-1                                                          |
|      avgRowSize = 0.0                                                          |
|      numNodes = 0                                                              |
|      tuple ids: 2                                                            |
+------------------------------------------------------------------------------+
128 rows in set (0.02 sec)
~~~

Query96 展示的查询规划中，涉及多个 StarRocks 概念，理解此类概念对于理解查询规划至关重要，可以通过一个表格进行阐述。

|名称|解释|
|---|---|
|avgRowSize|扫描数据行的平均大小|
|cardinality|扫描表的数据总行数|
|colocate|表是否采用了 Colocate 形式|
|numNodes|扫描涉及的节点数|
|rollup|物化视图|
|preaggregation|预聚合|
|predicates|谓词，也就是查询过滤条件|
|partitions|分区|
|table|表|

Query96 的 Query Plan 分为五个 Plan Fragment，编号从 0~4。阅读 Query Plan 可以采用自底向上的方式进行，逐个进行阅读。

上图中最底部的 Plan Fragment 为 Fragment 4，它负责扫描 time_dim 表，并提前执行相关查询条件 time_dim.t_hour = 8 and time_dim.t_minute >= 30，也就是大家所熟知的谓词下推。对于聚合表(Aggregate Key)，StarRocks 会根据不同查询选择是否开启 PREAGGREGATION，上图中 time_dim 的预聚合为关闭状态，关闭状态之下会读取 time_dim 的全部维度列，当表中维度列多的时候，这个可能会成为影响性能的一个关键因素。如果 time_dim 表有选择 Range Partition 进行数据划分，Query Plan 中的 partitions 会表征查询命中几个分区，无关分区被自动过滤会有效减少扫描数据量。如果有物化视图，StarRocks 会根据查询去自动选择物化视图，如果没有物化视图，那么查询自动命中 base table，也就是上图中展示的 rollup: time_dim。其他字段可以暂时不用关注。

当 time_dim 数据扫描完成之后，Fragment 4 的执行过程也就随之结束，此时它将扫描得到的数据传递给其他 Fragment，上图中的 EXCHANGE ID : 09 表征了数据传递给了标号为 9 的接收节点。

对于 Query96 的 Query Plan 而言，Fragment 2，3，4 功能类似，只是负责扫描的表不同。具体到查询中的 Order/Aggregation/Join 算子，都在 Fragment 1 中进行，下面着重介绍 Fragment 1。

Fragment 1 集成了三个 Join 算子的执行，采用默认的 BROADCAST 方式进行执行，也就是小表向大表广播的方式进行，如果两个 Join 的表都是大表，建议采用 SHUFFLE 的方式进行。目前 StarRocks 只支持 HASH JOIN，也就是采用哈希算法进行 Join。图中有一个 colocate 字段，这个用来表述两张 Join 表采用同样的分区/分桶方式，如此，Join 的过程可以直接在本地执行，不用进行数据的移动。Join 执行完成之后，就是执行上层的 Aggregation、Order by 和 TOP-N 的算子，Query96 的上述算子都比较浅显易懂。

至此，关于 Query96 的 Query Plan 的解释就告一段落，去掉具体的表达式，只保留算子的话，Query Plan 可以以一个更加宏观的角度展示，就是下图。

![8-5](../assets/8-5.png)

## Profile 分析

在理解了 Plan 的作用以后，我们来分析以下 BE 的执行结果 Profile，通过在 StarRocksManager 中执行查询，点击查询历史，就可看在“执行详情”tab 中看到 Profile 的详细文本信息，在“执行时间”tab 中能看到图形化的展示，这里我们采用 TPCH 的 Q4 查询来作为例子

~~~sql
-- TPCH Q4
select  o_orderpriority,  count(*) as order_count
from  orders
where
  o_orderdate >= date '1993-07-01'
  and o_orderdate < date '1993-07-01' + interval '3' month
  and exists (
    select  * from  lineitem
    where  l_orderkey = o_orderkey and l_commitdate < l_receiptdate
  )
group by o_orderpriority
order by o_orderpriority;
~~~

可以看到这是一个包含了相关子查询，group by，ordery by 和 count 的查询，其中 order 是订单表，lineitem 是货品明细表，这两张都是比较大的事实表，这个查询的含义是按照订单的优先级分组，统计每个分组的订单数量，同时有两个过滤条件：

* 订单创建时间是 1993 年 7 月 到 1993 年 10 月之间
* 这个订单对应的产品的提交日期(l_commitdate)小于收货日期（l_receiptadate）

执行 Query 后我们能看到 ：

![8-6](../assets/8-6.png)
  
左上角我们能看到整个查询执行了 3.106s，点击每个节点可以看到每一部分的执行信息，Active 表示这个节点（包含其所有子节点）的时间，整体结构上可以看到最下面的子节点是两个 scan node，他们分别 scan 了 573 万和 3793 万数据，然后进行了一次 shuffle join，完成后输出 525 万条数据，然后经过两层聚合，最后通过一个 Sort Node 后输出结果，其中 Exchange node 是数据交换节点，在这个 case 中是进行了两次 Shuffle。

一般分析 Profile 的核心就是找到执行时间最长的性能瓶颈所在的节点，比如我们可以从上往下依次查看，可以看到这个 Hash Join Node 占了主要时间：

~~~sql
HASH_JOIN_NODE (id = 2):(Active: 3s85ms, % non-child: 34.59%)
- AvgInputProbeChunkSize: 3.653K (3653)
- AvgOutputChunkSize: 3.57K (3570)
- BuildBuckets: 67.108864M (67108864)
- BuildRows: 31.611425M (31611425)
- BuildTime: 3s56ms
    - 1-FetchRightTableTime: 2s14ms
    - 2-CopyRightTableChunkTime: 0ns
    - 3-BuildHashTableTime: 1s19ms
    - 4-BuildPushDownExprTime: 0ns
- PeakMemoryUsage: 376.59 MB
- ProbeRows: 478.599K (478599)
- ProbeTime: 28.820ms
    - 1-FetchLeftTableTimer: 369.315us
    - 2-SearchHashTableTimer: 19.252ms
    - 3-OutputBuildColumnTimer: 893.29us
    - 4-OutputProbeColumnTimer: 7.612ms
    - 5-OutputTupleColumnTimer: 34.593us
- PushDownExprNum: 0
- RowsReturned: 439.16K (439160)
- RowsReturnedRate: 142.323K /sec
~~~

从这里信息可以看到 hash join 的执行主要分成两部分时间，也就是 BuildTime 和 ProbeTime，BuildTime 是扫描右表并构建 hash 表的过程，ProbeTime 是获取左表并搜索 hashtable 进行匹配并输出的过程。可以明显的看出这个节点的时间大部分花在了 BuildTime 的 FetchRightTableTime 和 BuildHashTableTime，对比刚才的数据 Scan 行数数据，我们意识到这个查询的左表和右表的顺序其实不理想，应该把左表设置为大表，右表 Build hash table 会效果更好，而且对于这两个表都是事实表，数据都比较多，我们也可以考虑采用 colocate Join 来避免 最下面的数据 shuffle，同时减少 Join 的数据量，于是我们参考 [“Colocate join”](../using_starrocks/Colocation_join.md) 建立了 colocation 关系后，并且改写 SQL 如下：

~~~sql
with t1 as (
    select l_orderkey from  lineitem
    where  l_commitdate < l_receiptdate
) select o_orderpriority,  count(*)as order_count from t1 right semi join orders_co  on l_orderkey = o_orderkey 
    where o_orderdate >= date '1993-07-01'
  and o_orderdate < date '1993-07-01' + interval '3' month
group by o_orderpriority
order by o_orderpriority;
~~~

执行结果：

![8-7](../assets/8-7.png)
  
新的 SQL 执行从 3.106s 降低到了 1.042s，可以明显看到两张大表没有了 Exchange 节点，直接通过 Colocated Join 进行，而且左右表顺序调换了以后整体性能有了大幅提升，新的 Join Node 信息如下：

~~~sql
HASH_JOIN_NODE (id = 2):(Active: 996.337ms, % non-child: 52.05%)
- AvgInputProbeChunkSize: 2.588K (2588)
- AvgOutputChunkSize: 35
- BuildBuckets: 1.048576M (1048576)
- BuildRows: 478.171K (478171)
- BuildTime: 187.794ms
    - 1-FetchRightTableTime: 175.810ms
    - 2-CopyRightTableChunkTime: 5.942ms
    - 3-BuildHashTableTime: 5.811ms
    - 4-BuildPushDownExprTime: 0ns
- PeakMemoryUsage: 22.38 MB
- ProbeRows: 31.609786M (31609786)
- ProbeTime: 807.406ms
    - 1-FetchLeftTableTimer: 282.257ms
    - 2-SearchHashTableTimer: 457.235ms
    - 3-OutputBuildColumnTimer: 26.135ms
    - 4-OutputProbeColumnTimer: 16.138ms
    - 5-OutputTupleColumnTimer: 1.127ms
- PushDownExprNum: 0
- RowsReturned: 438.502K (438502)
- RowsReturnedRate: 440.114K /sec
~~~

## SQL 指纹

StarRocks 支持规范化慢查询（ 路径 `fe.audit.log.slow_query` ）中 SQL 语句，并进行归类。然后针对各个类型的 SQL 语句，计算出其的 MD5 哈希值，对应字段为 `Digest`。

~~~test
2021-12-27 15: 13: 39,108 [slow_query] |Client = 172.26.xx.xxx: 54956|User = root|Db = default_cluster: test|State = EOF|Time = 2469|ScanBytes = 0|ScanRows = 0|ReturnRows = 6|StmtId = 3|QueryId = 824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery = true|feIp = 172.26.92.195|Stmt = select count(*) from test_basic group by id_bigint|Digest = 51390da6b57461f571f0712d527320f4
~~~

SQL 语句规范化仅保留重要的语法结构：

* 保留对象标识符，如数据库和表的名称。
* 转换常量为?。
* 删除注释并调整空格。

如下两个 SQL 语句，规范化后属于同一类 SQL。

~~~sql
SELECT * FROM orders WHERE customer_id = 10 AND quantity > 20

SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
~~~

规范化后如下类型的 SQL。

~~~sql
SELECT * FROM orders WHERE customer_id =? AND quantity >?
~~~
