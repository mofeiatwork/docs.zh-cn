# hour

## description

### Syntax

```Haskell
INT HOUR(DATETIME date)
```

获得日期中的小时的信息，返回值范围从 0-23。

参数为 Date 或者 Datetime 类型

## example

```Plain Text
MySQL > select hour('2018-12-31 23:59:59');
+-----------------------------+
| hour('2018-12-31 23:59:59') |
+-----------------------------+
|                          23 |
+-----------------------------+
```

## keyword

HOUR
