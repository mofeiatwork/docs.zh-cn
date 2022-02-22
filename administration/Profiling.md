# 性能优化

## 建表

### 数据模型选择

StarRocks 数据模型目前分为三类: AGGREGATE KEY， UNIQUE KEY， DUPLICATE KEY。三种模型中数据都是按 KEY 进行排序。

* AGGREGATE KEY: AGGREGATE KEY 相同时，新旧记录进行聚合，目前支持的聚合函数有 SUM， MIN， MAX， REPLACE。 AGGREGATE KEY 模型可以提前聚合数据，适合报表和多维分析业务。

~~~sql
CREATE TABLE site_visit
(
    siteid      INT,
    city        SMALLINT,
    username    VARCHAR(32),
    pv BIGINT   SUM DEFAULT '0'
)
AGGREGATE KEY(siteid, city, username)
DISTRIBUTED BY HASH(siteid) BUCKETS 10;
~~~

UNIQUE KEY: UNIQUE KEY 相同时，新记录覆盖旧记录。目前 UNIQUE KEY 实现上和 AGGREGATE KEY 的 REPLACE 聚合方法一样，二者本质上可以认为相同。适用于有更新的分析业务。

~~~sql
CREATE TABLE sales_order
(
    orderid     BIGINT,
    status      TINYINT,
    username    VARCHAR(32),
    amount      BIGINT DEFAULT '0'
)
UNIQUE KEY(orderid)
DISTRIBUTED BY HASH(orderid) BUCKETS 10;
~~~

DUPLICATE KEY: 只指定排序列，相同 DUPLICATE KEY 的记录会同时存在。适用于数据无需提前聚合的分析业务。

~~~sql
CREATE TABLE session_data
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          varchar(32),
    brower      CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid) BUCKETS 10;
~~~

### 内存表

StarRocks 支持把表数据全部缓存在内存中，用于加速查询，内存表适合数据行数不多维度表的存储。

~~~sql
CREATE TABLE memory_table
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          varchar(32),
    brower      CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid) BUCKETS 10
PROPERTIES (
           "in_memory" = "true"
);
~~~

### Colocate Table

为了加速查询，分布相同的相关表可以采用共同的分桶列。当分桶列相同时，相关表进行 JOIN 操作时，可以避免数据在集群中的传输，直接在本地进行 JOIN。

~~~sql
CREATE TABLE colocate_table
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          varchar(32),
    brower      CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid) BUCKETS 10
PROPERTIES(
    "colocate_with" = "group1"
);
~~~

更多关于 colocate join 的使用和副本管理机制参考 [Colocate join](../using_starrocks/Colocation_join.md)

### 大宽表与 star schema

业务方建表时, 为了和前端业务适配, 传统建模方式直接将 schema 定义成大宽表。对于 StarRocks 而言, 可以选择更灵活的星型模型来替代大宽表，比如用一个视图来取代宽表进行建模，直接使用多表关联来查询，比如在 SSB 的标准测试集的对比中，StarRocks 的多表关联性能比单表查询下降并不太多。然后相比星型模型，宽表的缺点有：

1. 维度信息更新会反应到整张表中，而更新的频率直接影响查询的效率，宽表的更新成本。
2. 宽表的建设需要额外的开发工作、存储空间和数据 backfill 的成本。
3. schema 中字段数比较多, 聚合模型中可能 key 列比较多, 导入过程中需要排序的列会增加，导致导入时间变长。

使用过程中，建议优先使用星型模型，可以在保证灵活的基础上获得高效的指标分析效果，但是对于有高并发或者低延迟要求的业务，还是可以选择宽表模型进行加速，StarRocks 也可以提供与 ClickHouse 相当的宽表查询性能。

### 分区(parition)和分桶(bucket)

StarRocks 支持两级分区存储, 第一层为 RANGE 分区(partition), 第二层为 HASH 分桶(bucket)。

* RANGE 分区(partition) : RANGE 分区用于将数据划分成不同区间, 逻辑上可以理解为将原始表划分成了多个子表。 业务上，多数用户会选择采用按时间进行 partition, 让时间进行 partition 有以下好处：

* 可区分冷热数据
* 可用上 StarRocks 分级存储(SSD + SATA)的功能
* 按分区删除数据时，更加迅速

* HASH 分桶(bucket) : 根据 hash 值将数据划分成不同的 bucket。

* 建议采用区分度大的列做分桶, 避免出现数据倾斜
* 为方便数据恢复, 建议单个 bucket 的 size 不要太大, 单个 bucket 中数据压缩后大小保持在 100M-1GB 左右, 所以建表或增加 partition 时请合理考虑 buckets 数目, 其中不同 partition 可指定不同的 buckets 数。
* random 分桶的方式不建议采用，建表时烦请指定明确的 hash 分桶列。

### 稀疏索引和 bloomfilter

StarRocks 对数据进行有序存储, 在数据有序的基础上为其建立稀疏索引, 索引粒度为 block(1024 行)。

* 稀疏索引选取 schema 中固定长度的前缀作为索引内容, 目前 StarRocks 选取 36 个字节的前缀作为索引。

* 建表时建议将查询中常见的过滤字段放在 schema 的前面, 区分度越大，频次越高的查询字段越往前放。
* 这其中有一个特殊的地方, 就是 varchar 类型的字段, varchar 类型字段只能作为稀疏索引的最后一个字段，索引会在 varchar 处截断, 因此 varchar 如果出现在前面，可能索引的长度不足 36 个字节。

* 对于上述 site_visit 表

    `site_visit(siteid, city, username, pv)`

* 排序列有 siteid, city, username 三列, siteid 所占字节数为 4, city 所占字节数为 2，username 占据 32 个字节, 所以前缀索引的内容为 siteid + city + username 的前 30 个字节
* 除稀疏索引之外, StarRocks 还提供 bloomfilter 索引, bloomfilter 索引对区分度比较大的列过滤效果明显。 如果考虑到 varchar 不能放在稀疏索引中, 可以建立 bloomfilter 索引。

### 倒排索引

StarRocks 支持倒排索引，采用位图技术构建索引(Bitmap Index)。索引能够应用在 Duplicate 数据模型的所有列和 Aggregate, Uniqkey 模型的 Key 列上，位图索引适合取值空间不大的列，例如性别、城市、省份等信息列上。随着取值空间的增加，位图索引会同步膨胀。

### 物化视图(rollup)

Rollup 本质上可以理解为原始表(base table)的一个物化索引。建立 rollup 时可只选取 base table 中的部分列作为 schema，schema 中的字段顺序也可与 base table 不同。 下列情形可以考虑建立 rollup:

* base table 中数据聚合度不高，这一般是因 base table 有区分度比较大的字段而导致。此时可以考虑选取部分列，建立 rollup。 对于上述 site_visit 表

    `site_visit(siteid, city, username, pv)`

* siteid 可能导致数据聚合度不高，如果业务方经常根据城市统计 pv 需求，可以建立一个只有 city, pv 的 rollup。

    `ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);`

* base table 中的前缀索引无法命中，这一般是 base table 的建表方式无法覆盖所有的查询模式。此时可以考虑调整列顺序，建立 rollup。对于上述 session_data 表

    `session_data(visitorid, sessionid, visittime, city, province, ip, brower, url)`

* 如果除了通过 visitorid 分析访问情况外，还有通过 brower, province 分析的情形，可以单独建立 rollup。

    `ALTER TABLE session_data ADD ROLLUP rollup_brower(brower,province,ip,url) DUPLICATE KEY(brower,province);`

## 导入

StarRocks 目前提供 broker load 和 stream load 两种导入方式， 通过指定导入 label 标识一批次的导入。StarRocks 对单批次的导入会保证原子生效， 即使单次导入多张表也同样保证其原子性。

* stream load : 通过 http 推的方式进行导入，微批导入。1MB 数据导入延迟维持在秒级别，适合高频导入。
* broker load : 通过拉的方式导入， 适合天级别的批量数据的导入。

## schema change

StarRocks 中目前进行 schema change 的方式有三种，sorted schema change，direct schema change， linked schema change。

* sorted schema change: 改变了列的排序方式，需对数据进行重新排序。例如删除排序列中的一列， 字段重排序。

    `ALTER TABLE site_visit DROP COLUMN city;`

* direct schema change: 无需重新排序，但是需要对数据做一次转换。例如修改列的类型，在稀疏索引中加一列等。

    `ALTER TABLE site_visit MODIFY COLUMN username varchar(64);`

* linked schema change: 无需转换数据，直接完成。例如加列操作。

    `ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';`

    建表时建议考虑好 schema，这样在进行 schema change 时可以加快速度。
