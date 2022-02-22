# monthname

## description

### Syntax

```Haskell
VARCHAR MONTHNAME(DATE)
```

返回日期对应的月份名字

参数为 Date 或者 Datetime 类型

## example

```Plain Text
MySQL > select monthname('2008-02-03 00:00:00');
+----------------------------------+
| monthname('2008-02-03 00:00:00') |
+----------------------------------+
| February                         |
+----------------------------------+
```

## keyword

MONTHNAME
