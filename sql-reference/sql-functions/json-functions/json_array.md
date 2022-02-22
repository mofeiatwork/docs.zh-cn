# json_array

## 功能

构造 JSON 数组。

## 语法

- JSON json_array()
- JSON json_array(ANY value, ...)

## 参数说明

- `value ANY`
JSON 值，允许 integer/double/float/varchar/char 等类型

## 返回值说明

- 返回类型：JSON
- 返回构造出的 JSON 数组

## 示例

```sql
-- 构造空的JSON对象
mysql> select json_array();
+--------------+
| json_array() |
+--------------+
| []           |
+--------------+

-- 构造一个多种数据类型组成的JSON对组
mysql> select json_array(1, true, 'starrocks', 1.1);
+---------------------------------------+
| json_array(1, TRUE, 'starrocks', 1.1) |
+---------------------------------------+
| [1, true, "starrocks", 1.1]           |
+---------------------------------------+
```

## 关键词

JSON, JSON_ARRAY
