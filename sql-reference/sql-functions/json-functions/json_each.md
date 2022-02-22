# json_each

## 功能

将 JSON 对象的最外层展开成多行 Key/Value 的表示形式。

## 语法

`Table<VARCHAR, JSON> json_each(JSON object)`

## 参数说明

- `JSON doc`: 需要展开的 JSON 对象

## 返回值说明

返回 JSON 最外层展开的 Key/Value 结果，Key 为 JSON 字段名，Value 为字段值。

## 注意事项

`json_each` 为 Table Function，须在 From 子句中通过 Lateral Join 使用，不可用于 select 子句。

## 示例

```sql

mysql> select * from tj;
+------+------------------+
| id   | j                |
+------+------------------+
|    1 | {"a": 1, "b": 2} |
|    3 | {"a": 3}         |
+------+------------------+

-- 将j列的JSON对象展开，返回其Key/Value
mysql> select * from tj, json_each(j);
+------+------------------+------+-------+
| id   | j                | key  | value |
+------+------------------+------+-------+
|    1 | {"a": 1, "b": 2} | a    | 1     |
|    1 | {"a": 1, "b": 2} | b    | 2     |
|    3 | {"a": 3}         | a    | 3     |
+------+------------------+------+-------+

```

## keyword

JSON, JSON_EACH
