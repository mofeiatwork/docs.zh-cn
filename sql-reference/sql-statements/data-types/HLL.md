# HLL(HyperLogLog)

## description

VARCHAR(M)

变长字符串，M 代表的是变长字符串的长度。M 的范围是 1-16385

用户不需要指定长度和默认值。长度根据数据的聚合程度系统内控制

并且 HLL 列只能通过配套的 hll_union_agg、hll_raw_agg、hll_cardinality、hll_hash 进行查询或使用

## keyword

HLL, HYPERLOGLOG
