# json_object

## 功能

手动构造 JSON 对象。

## 语法

- `JSON json_object()`
- `JSON json_object(varchar field_name, any field_value, ...)`

## 参数说明

- `VARCHAR field_name`: JSON 字段名
- `ANY field_value`: JSON 字段值，允许 double/float/integer/varchar/char 等类型

## 返回值说明

返回类型为 JSON, 即构造出的 JSON 对象。

## 示例

```sql

-- 构造空的JSON对象
mysql> select json_object();
+---------------+
| json_object() |
+---------------+
| {}            |
+---------------+

-- 构造一个多种数据类型组成的JSON对象
mysql> select json_object('name', 'starrocks', 'active', true, 'published', 2020);
+---------------------------------------------------------------------+
| json_object('name', 'starrocks', 'active', TRUE, 'published', 2020) |
+---------------------------------------------------------------------+
| {"active": true, "name": "starrocks", "published": 2020}            |
+---------------------------------------------------------------------+
```

## 关键词

JSON, JSON_OBJECT
