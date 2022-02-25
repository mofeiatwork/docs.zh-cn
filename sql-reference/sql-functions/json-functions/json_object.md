# json_object

## 功能

手动构造 JSON 对象。

## 语法

- `json_object()`
- `json_object(field_name, field_value, ...)`

## 参数说明

- `field_name`: JSON 字段名；支持 VARCHAR 类型
- `field_value`: JSON 字段值；支持 TINYINT/SMALLINT/INTEGER/BIGINT/LARGEINT/DOUBLE/FLOAT/VARCHAR/CHAR SQL 类型，以及 JSON 类型

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

-- 构造一个多种数据类型组成的 JSON 对象
mysql> select json_object('name', 'starrocks', 'active', true, 'published', 2020);
+---------------------------------------------------------------------+
| json_object('name', 'starrocks', 'active', TRUE, 'published', 2020) |
+---------------------------------------------------------------------+
| {"active": true, "name": "starrocks", "published": 2020}            |
+---------------------------------------------------------------------+

-- 构造嵌套的 JSON Object
mysql> select json_object('k1', 1, 'k2', json_object('k2', 2), 'k3', json_array(4, 5));
+--------------------------------------------------------------------------+
| json_object('k1', 1, 'k2', json_object('k2', 2), 'k3', json_array(4, 5)) |
+--------------------------------------------------------------------------+
| {"k1": 1, "k2": {"k2": 2}, "k3": [4, 5]}                                 |
+--------------------------------------------------------------------------+
```

## 关键词

JSON, JSON_OBJECT
