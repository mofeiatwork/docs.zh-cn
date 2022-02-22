# JSON 类型转换

## 功能

JSON 类型可以通过 CAST 函数实现与其他 SQL 类型的转换。

## 语法

- `CAST(JSON json AS VARCHAR/INTEGER/BOOLEAN/DOUBLE/FLOAT)`: 将 JSON 转为 SQL 类型
- `CAST(VARCHAR/INTEGER/BOOLEAN/DOUBLE/FLOAT AS JSON)`: 将 SQL 类型转为 JSON 类型

## 参数说明

当前支持转换的类型为 VARCHAR/INTEGER/BOOLEAN/DOUBLE/FLOAT。

## 返回值说明

将 JSON 转型为 SQL 类型时:

- 如果类型不兼容，例如将数值类型转成字符串，会返回 NULL
- 如果 JSON 的数值类型转为更窄的 SQL 数值类型，会存在溢出问题
- 对于 JSON 中的 null 类型，会转为 SQL 中的 NULL 类型
- 将 JSON 转为 varchar 类型时，如果 JSON 本身是 string 类型，则返回无引号的字符串表示，否则会将其转为 JSON 的字符串表示

将 SQL 类型转为 JSON 类型时:

- 如果数值超出 JSON 的表示范围，会发生溢出问题
- 对于 SQL 的 NULL 类型，不会转为 JSON 中的 NULL 类型，仍然为 SQL 中的 NULL

## 示例

```sql

-- 将int转成JSON
mysql> select cast(1 as json);
+-----------------+
| CAST(1 AS JSON) |
+-----------------+
| 1               |
+-----------------+

-- 将varchar转为JSON
mysql> select cast("star" as json);
+----------------------+
| CAST('star' AS JSON) |
+----------------------+
| "star"               |
+----------------------+

-- 将bool转为JSON
mysql> select cast(true as json);
+--------------------+
| CAST(TRUE AS JSON) |
+--------------------+
| true               |
+--------------------+

-- 将JSON转为 int
mysql> select cast(parse_json('1') as int);
+------------------------------+
| CAST(parse_json('1') AS INT) |
+------------------------------+
|                            1 |
+------------------------------+

-- 将JSON string转为 varchar
mysql> select cast(parse_json('"star"') as varchar);
+---------------------------------------+
| CAST(parse_json('"star"') AS VARCHAR) |
+---------------------------------------+
| star                                  |
+---------------------------------------+

-- 将JSON object转为 varchar
mysql> select cast(parse_json('{"star": 1}') as varchar);
+--------------------------------------------+
| CAST(parse_json('{"star": 1}') AS VARCHAR) |
+--------------------------------------------+
| {"star": 1}                                |
+--------------------------------------------+

-- 将JSON数组转为 varchar
mysql> select cast(parse_json('[1,2,3]') as varchar);
+----------------------------------------+
| CAST(parse_json('[1,2,3]') AS VARCHAR) |
+----------------------------------------+
| [1, 2, 3]                              |
+----------------------------------------+


```
