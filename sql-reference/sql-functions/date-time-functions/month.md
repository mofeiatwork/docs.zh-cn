# month

## description

### Syntax

```Haskell
INT MONTH(DATETIME date)
```

返回时间类型中的月份信息，范围是 1, 12

参数为 Date 或者 Datetime 类型

## example

```Plain Text
MySQL > select month('1987-01-01');
+-----------------------------+
|month('1987-01-01 00:00:00') |
+-----------------------------+
|                           1 |
+-----------------------------+
```

## keyword

MONTH
