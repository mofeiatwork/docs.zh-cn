# json_query

## 功能

查询 JSON 中的某个字段。

## 语法

`json_query(doc, json_path)`

## 参数说明

- `doc`: 查询的 JSON，可以为列引用，或通过 `parse_json` 等函数构造的 JSON Object/Array；支持 JSON 类型
- `json_path`: JSON 中的查询路径；支持 VARCHAR 类型；语法参考 [JSON Path](/sql-reference/sql-functions/json-functions/json_path.md)

## 返回值说明

- 返回 JSON 类型
- 如果查询的字段存在，返回其 JSON 对象，否则返回 NULL。

## 注意事项

Starrocks 支持的 JSON Path 的语法请参考 [JSON Path 语法](/sql-reference/sql-functions/json-functions/json_path.md)。

## 示例

```sql

-- 查询JSON对象中的 $.a.b 字段，返回 JSON类型的 1
mysql> select json_query(parse_json('{"a": {"b": 1}}'), '$.a.b') ;
+----------------------------------------------------+
| json_query(parse_json('{"a": {"b": 1}}'), '$.a.b') |
+----------------------------------------------------+
| 1                                                  |
+----------------------------------------------------+

-- 查询JSON对象中的 $.a.c 字段，由于不存在此字段，返回NULL
mysql> select json_query(parse_json('{"a": {"b": 1}}'), '$.a.c') ;
+----------------------------------------------------+
| json_query(parse_json('{"a": {"b": 1}}'), '$.a.c') |
+----------------------------------------------------+
| NULL                                               |
+----------------------------------------------------+

-- 查询JSON对象中的 a 数组的第二个元素，返回 JSON 类型的 3
mysql> select json_query(parse_json('{"a": [1,2,3]}'), '$.a[2]') ;
+----------------------------------------------------+
| json_query(parse_json('{"a": [1,2,3]}'), '$.a[2]') |
+----------------------------------------------------+
| 3                                                  |
+----------------------------------------------------+

-- 查询JSON对象中的 a 数组的第三个元素，由于不存在，返回NULL 
mysql> select json_query(parse_json('{"a": [1,2,3]}'), '$.a[3]') ;
+----------------------------------------------------+
| json_query(parse_json('{"a": [1,2,3]}'), '$.a[3]') |
+----------------------------------------------------+
| NULL                                               |
+----------------------------------------------------+

```

## keyword

JSON, JSON_QUERY
