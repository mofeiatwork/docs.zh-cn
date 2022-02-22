# parse_json

## 功能

从 JSON 格式的字符串解析出 JSON 对象。

## 语法

- `JSON parse_json(VARCHAR str)`

## 参数说明

- `VARCHAR str`: JSON 字符串，允许多种 JSON 类型，包括 Number/Boolean/Null/Array/Object

## 返回值说明

如果字符串是合法的 JSON，则返回构造出的 JSON，否则返回 NULL。

## 注意事项

JSON 对象的 string 类型需要用双引号括起来，遵循 JSON 标准，例如 `parse_json('"starrocks"')`。而 JSON 格式的字符串需要用单引号。

## 示例

```sql

-- 构造值为 null 的JSON 
mysql> select parse_json('null');
+--------------------+
| parse_json('null') |
+--------------------+
| null               |
+--------------------+

-- 构造值为1的JSON
mysql> select parse_json('1');
+-----------------+
| parse_json('1') |
+-----------------+
| 1               |
+-----------------+

-- 构造一个JSON数组
mysql> select parse_json('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+

-- 构造一个JSON对象
mysql> select parse_json('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+

-- 不合法的JSON字符串，会返回NULL, 此例的字段名没有用双引号括起来
mysql> select parse_json('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+

```

## 关键词

JSON, PARSE_JSON
