# day

## description

### Syntax

```Haskell
INT DAY(DATETIME date)
```

获得日期中的天信息，返回值范围从 1-31。

参数为 Date 或者 Datetime 类型

## example

```Plain Text
MySQL > select day('1987-01-31');
+----------------------------+
| day('1987-01-31 00:00:00') |
+----------------------------+
|                         31 |
+----------------------------+
```

## keyword

DAY
