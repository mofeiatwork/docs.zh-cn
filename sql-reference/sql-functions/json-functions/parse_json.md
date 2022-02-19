# parse_json

## 功能

从JSON字符串解析出JSON对象。

## 语法

- `JSON parse_json(VARCHAR str)`

## 参数说明

- `VARCHAR str`: JSON字符串，允许多种JSON类型，包括 Number/Boolean/Null/Array/Object

## 返回值说明

如果字符串是合法的JSON，则返回构造出的JSON，否则返回NULL。

## 注意事项

JSON对象的string需要用双引号括起来，遵循JSON标准，而JSON字符串需要用单引号。

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
