# JSON 函数

Starrocks 当前支持以下 JSON 函数。

## 构造函数

| 函数名 | 功能 | 示例 |
| ---- | ---- | --- |
| json_object | 构造 JSON Object | `json_object('a', 1, 'b', 2)` | [json_object](/sql-reference/sql-functions/json-functions/json_object.md) |
| json_array | 构造 JSON Array | `json_array(1, 2, 3)` | [json_array](/sql-reference/sql-functions/json-functions/json_array.md) |
| parse_json | 从字符串解析 JSON | `parse_json('{"a": 1}')`| [parse_json](/sql-reference/sql-functions/json-functions/parse_json.md) |

## 查询函数

| 函数名 | 功能 | 示例 | 详细说明 |
| ---- | ---- | --- | ---- |
| -> | 查询 JSON 对象中的字段 | `obj->'a'` | [箭头语法](/sql-reference/sql-functions/json-functions/json_arrow.md) |
| json_query | 查询 JSON 对象中的字段 | `json_query(parse_json('{"a": 1}'), '$.a')` | [json_query](/sql-reference/sql-functions/json-functions/json_query.md) |
| json_exists | 查询 JSON 对象中是否存在某个字段 | `json_exists(parse_json('{" a ": 1}'), '$.a') | [json_exists](/sql-reference/sql-functions/json-functions/json_exists.md) |
| json_each | 将最外层的 JSON 对象展开成 Key/Value | `json_each('{" a ": 1, " b ": 2}') | [json_each](/sql-reference/sql-functions/json-functions/json_each.md) |

## 其他

在以上函数中，通常使用 JSON Path 进行查询，其语法参考文档 [JSON Path](/sql-reference/sql-functions/json-functions/json_path.md)。

## 关键词

JSON, JSON_ARRAY
