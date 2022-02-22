# JSON 谓词

## 功能

JSON 类型支持谓词计算，作为 WHERE 子句进行数据过滤。

## 语法

- `JSON < JSON`
- `JSON <= JSON`
- `JSON > JSON`
- `JSON >= JSON`
- `JSON = JSON`
- `JSON != JSON`

## 参数说明

运算符两边都需要是 JSON 类型。

## 返回值说明

JSON 的比较运算符遵循以下规则：

1. 如果运算符两边类型相同，且都是基本类型（Number/String/Boolean)，遵循基本类型的运算规则
2. 如果运算符两边类型相同，且是复合类型(Object, Array)，按照元素逐个比较
3. 如果运算符两边类型不同，则按照类型顺序比较，当前的类型顺序是: `Null < Bool < Array < Object < Double < Integer < String`

## 示例

```sql
-- 数值类型比较
mysql> select * from tj where j->'a' <= parse_json('3');
+------+----------------------+
| id   | j                    |
+------+----------------------+
|    3 | {"a": 3, "b": true}  |
|    1 | {"a": 1, "b": true}  |
|    2 | {"a": 2, "b": false} |
+------+----------------------+

-- 字符串类型比较
mysql> select parse_json('"a"') < parse_json('"b"');
+---------------------------------------+
| parse_json('"a"') < parse_json('"b"') |
+---------------------------------------+
|                                     1 |
+---------------------------------------+

```
