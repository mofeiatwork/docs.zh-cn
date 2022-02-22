# BITMAP

## description

描述：

BITMAP 与 HLL 类似只能作为聚合表的 value 类型使用，常见用来加速 count distinct 的去重计数使用，

通过位图可以进行精确计数，可以通过 bitmap 函数可以进行集合的各种操作，相对 HLL 他可以获得精确的结果，

但是需要消耗更多的内存和磁盘资源，另外 Bitmap 只能支持整数类型的聚合，如果是字符串等类型需要采用字典进行映射。

## keyword

BITMAP BITMAP_UNION
