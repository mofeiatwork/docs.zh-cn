# to_days

## description

### Syntax

```Haskell
INT TO_DAYS(DATETIME date)
```

返回 date 距离 0000-01-01 的天数

参数为 Date 或者 Datetime 类型

## example

```Plain Text
MySQL > select to_days('2007-10-07');
+-----------------------+
| to_days('2007-10-07') |
+-----------------------+
|                733321 |
+-----------------------+
```

## keyword

TO_DAYS, TO, DAYS
