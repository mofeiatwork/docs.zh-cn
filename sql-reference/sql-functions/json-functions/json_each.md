# json_each

## 功能

将JSON对象的最外层展开成多行Key/Value的表示形式。

## 语法

`Table<VARCHAR, JSON> json_each(JSON object)`

## 参数说明

- `JSON doc`: 需要展开的JSON对象

## 返回值说明

返回JSON最外层展开的Key/Value结果，Key为JSON字段名，Value为字段值。

## 注意事项

`json_each`为Table Function，须在From子句中通过Lateral Join使用，不可用于select子句。

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
