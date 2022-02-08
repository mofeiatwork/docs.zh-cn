# json_object
## Description
构造JSON对象。

## Syntax
- JSON json_object()
- JSON json_object(varchar field_name, any field_value, ...)

## Arguments
- `field_name VARCHAR`
JSON字段名
- `field_value any`
JSON字段值，允许double/float/integer/varchar/char等类型。

## Return value
- 返回类型：JSON
- 返回构造出的JSON对象

## Example
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


## keyword
JSON, JSON_OBJECT
