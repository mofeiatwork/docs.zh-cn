# json_array

## Description

构造JSON数组。

## Syntax

- JSON json_array()
- JSON json_array(ANY value, ...)

## Arguments

- `value ANY`
JSON值，允许integer/double/float/varchar/char等类型

## Return value

- 返回类型：JSON
- 返回构造出的JSON数组

## Example

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

## keyword

JSON, JSON_ARRAY
