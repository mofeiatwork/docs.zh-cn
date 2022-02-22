# PERCENTILE_APPROX

## description

### Syntax

`PERCENTILE_APPROX(expr, DOUBLE p[, DOUBLE compression])`

返回第 p 个百分位点的近似值，p 的值介于 0 到 1 之间

compression 参数是可选项，可设置范围是 [2048, 10000]，值越大，精度越高，内存消耗越大，计算耗时越长。
compression 参数未指定或设置的值在 [2048, 10000] 范围外，以 10000 的默认值运行

该函数使用固定大小的内存，因此对于高基数的列可以使用更少的内存，可用于计算 tp99 等统计值

## example

```plain text
MySQL > select `table`, percentile_approx(cost_time,0.99)
from log_statis
group by `table`;
+----------+--------------------------------------+
| table    | percentile_approx(`cost_time`, 0.99) |
+----------+--------------------------------------+
| test     |                                54.22 |
+----------+--------------------------------------+

MySQL > select `table`, percentile_approx(cost_time,0.99, 4096)
from log_statis
group by `table`;
+----------+----------------------------------------------+
| table    | percentile_approx(`cost_time`, 0.99, 4096.0) |
+----------+----------------------------------------------+
| test     |                                        54.21 |
+----------+----------------------------------------------+
```

## keyword

PERCENTILE_APPROX, PERCENTILE, APPROX
