# dayofyear

## description

### Syntax

```Haskell
INT DAYOFYEAR(DATETIME date)
```

获得日期中对应当年中的哪一天。

参数为 Date 或者 Datetime 类型

## example

```Plain Text
MySQL > select dayofyear('2007-02-03 00:00:00');
+----------------------------------+
| dayofyear('2007-02-03 00:00:00') |
+----------------------------------+
|                               34 |
+----------------------------------+
```

## keyword

DAYOFYEAR
