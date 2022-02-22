# dayofmonth

## description

### Syntax

```Haskell
INT DAYOFMONTH(DATETIME date)
```

获得日期中的天信息，返回值范围从 1-31。

参数为 Date 或者 Datetime 类型

## example

```Plain Text
MySQL > select dayofmonth('1987-01-31');
+-----------------------------------+
| dayofmonth('1987-01-31 00:00:00') |
+-----------------------------------+
|                                31 |
+-----------------------------------+
```

## keyword

DAYOFMONTH
