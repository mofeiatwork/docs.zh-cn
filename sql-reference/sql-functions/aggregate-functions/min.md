# MIN

## description

### Syntax

```Haskell
MIN(expr)
```

返回 expr 表达式的最小值

## example

```plain text
MySQL > select min(scan_rows)
from log_statis
group by datetime;
+------------------+
| min(`scan_rows`) |
+------------------+
|                0 |
+------------------+
```

## keyword

MIN
