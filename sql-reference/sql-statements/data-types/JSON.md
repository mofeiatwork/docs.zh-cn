# JSON

## 描述

JSON是一种半结构化类型，支持树形结构，且Schema相对灵活，因此广泛应用于数据存储和分析场景。

Starrocks所支持的JSON类型，着眼于为数据分析领域的半结构化数据提供高效的查询分析能力。
因此Starrocks没有采用常见的字符串方式存储JSON数据，而是采用了二进制格式进行编码，优化查询效率。
在此基础上，借助Starrocks的向量化引擎，使用JSON类型能够对灵活的数据，进行高效的分析。

## 示例

建表时使用`JSON`关键字即可创建JSON类型的列。

```sql
CREATE TABLE `tj` (
  `id` INT(11) NOT NULL COMMENT "",
  `j`  JSON NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`) BUCKETS 1
PROPERTIES (
"replication_num" = "1",
"in_memory" = "false",
"storage_format" = "DEFAULT"
);
```

当前JSON类型仅支持Duplicate模型，且不允许为Duplicate Key。

对于JSON类型的数据，可通过INSERT进行数据写入：

```sql
INSERT INTO tj VALUES (1, parse_json('{"a": 1, "b": true}'));
INSERT INTO tj VALUES (2, parse_json('{"a": 2, "b": false}'));
INSERT INTO tj VALUES (3, parse_json('{"a": 3, "b": true}'));
INSERT INTO tj VALUES (4, json_object('a', 4, 'b',  false));
```

在此基础上，可对JSON数据进行查询：

```sql
-- 全表查询
mysql> select * from tj;
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    2 | {"a": 2, "b": false} |
|    1 | {"a": 1, "b": true}  |
|    3 | {"a": 3, "b": true}  |
|    4 | {"a": 4, "b": false} |
+------+----------------------+

-- 按照id过滤
mysql> select * from tj where id = 1;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |
+------+---------------------+

-- 按照JSON过滤
mysql> select * from tj where j->'a' = parse_json('1');
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |
+------+---------------------+

-- 对JSON进行范围查询
mysql> select * from tj where j->'a' <= parse_json('3');
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    2 | {"a": 2, "b": false} |
|    3 | {"a": 3, "b": true}  |
|    1 | {"a": 1, "b": true}  |
+------+----------------------+


-- 按照JSON过滤，将其转型为SQL类型计算
mysql> select * from tj where cast(j->'b' as boolean) = true;
+------+---------------------+
| id   | j                   |
+------+---------------------+
|    1 | {"a": 1, "b": true} |
|    3 | {"a": 3, "b": true} |
+------+---------------------+

-- 将JSON转为SQL类型进行数值运算
mysql> select cast(j->'a' as int) from tj where cast(j->'b' as boolean) ;
+-----------------------+
| CAST(`j`->'a' AS INT) |
+-----------------------+
|                     3 |
|                     1 |
+-----------------------+

mysql> select sum(cast(j->'a' as int)) from tj where cast(j->'b' as boolean) ;
+----------------------------+
| sum(CAST(`j`->'a' AS INT)) |
+----------------------------+
|                          4 |
+----------------------------+


-- 按照JSON列进行排序
mysql> select * from tj where j->'a' <= parse_json('3') order by cast(j->'a' as int);
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    1 | {"a": 1, "b": true}  |
|    2 | {"a": 2, "b": false} |
|    3 | {"a": 3, "b": true}  |
+------+----------------------+

```

## 数据导入

当前JSON类型的数据导入有几种方式:

- 通过INSERT写入少量测试数据，通常辅之以 `parse_json`, `json_object`等构造函数
- 通过Stream Load的方式导入 JSON文件
- 通过Broker Load的方式导入 Parquet文件

对于JSON文件的导入，可以通过jsonpath参数将JSON数据映射到Starrocks中的列：

- 单个字段映射到JSON列：例如通过 `$.a` 将JSON对象中的 `a` 字段映射到 JSON 列
- 整个JSON对象映射到JSON列：通过 `$` 将整个JSON对象导入 JSON 列

对于Parquet文件的导入，目前Starrocks能够支持其中部分数据类型的转换：

- 整数类型：INT8/INT16/INT32/INT64/UINT8/UINT16/UINT32/UINT64
- 布尔类型：BOOL
- 字符串类型：STRING
- 嵌套类型：MAP/LIST/STRUCT

对于Parquet的MAP、STRUCT类型，会转换为JSON Object；对于LIST类型，会转换为JSON Array。

## 查询

当前JSON支持以下函数和类型转换：

- [箭头语法](/sql-reference/sql-functions/json-functions/json_arrow.md)
- [json_query](/sql-reference/sql-functions/json-functions/json_query.md)
- [json_exists](/sql-reference/sql-functions/json-functions/json_exists.md)
- [json_object](/sql-reference/sql-functions/json-functions/json_object.md)
- [json_array](/sql-reference/sql-functions/json-functions/json_array.md)
- [json_parse](/sql-reference/sql-functions/json-functions/parse_json.md)
- [JSON Path](/sql-reference/sql-functions/json-functions/json_path.md)
- [类型转换](/sql-reference/sql-functions/json-functions/json_cast.md)

## 限制

- 当前JSON类型的最大长度和String类型相同
- 当前JSON类型不能直接用于ORDER BY、GROUP BY、JOIN计算，可转为SQL类型计算
- 当前JSON类型仅可用于Duplicate模型，且不支持作为Duplicate Key
- 当前JSON类型不支持用作Distribute Key或Partition Key
- 当前JSON类型仅支持 `<, <=, >, >=, =, !=` 谓词查询，不支持IN查询

## 关键词

JSON
