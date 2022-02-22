# dayofweek

## description

### Syntax

```Haskell
INT dayofweek(DATETIME date)
```

DAYOFWEEK 函数返回日期的工作日索引值，即星期日为 1，星期一为 2，星期六为 7

参数为 Date 或者 Datetime 类型或者可以 cast 为 Date 或者 Datetime 类型的数字

## example

```Plain Text
MySQL > select dayofweek('2019-06-25');
+----------------------------------+
| dayofweek('2019-06-25 00:00:00') |
+----------------------------------+
|                                3 |
+----------------------------------+

MySQL > select dayofweek(cast(20190625 as date));
+-----------------------------------+
| dayofweek(CAST(20190625 AS DATE)) |
+-----------------------------------+
|                                 3 |
+-----------------------------------+
```

## keyword

DAYOFWEEK
