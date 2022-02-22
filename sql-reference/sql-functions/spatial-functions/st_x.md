# ST_X

## description

### Syntax

```Haskell
DOUBLE ST_X(POINT point)
```

当 point 是一个合法的 POINT 类型时，返回对应的 X 坐标值

## example

```Plain Text
MySQL > SELECT ST_X(ST_Point(24.7, 56.7));
+----------------------------+
| st_x(st_point(24.7, 56.7)) |
+----------------------------+
|                       24.7 |
+----------------------------+
```

## keyword

ST_X, ST, X
