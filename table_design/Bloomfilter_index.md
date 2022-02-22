# Bloomfilter 索引

## 原理

### **1 什么是 Bloom Filter**

Bloom Filter（布隆过滤器）是用于判断某个元素是否在一个集合中的数据结构，优点是空间效率和时间效率都比较高，缺点是有一定的误判率。

![bloomfilter](../assets/3.7.1.png)

布隆过滤器是由一个 Bit 数组和 n 个哈希函数构成。Bit 数组初始全部为 0，当插入一个元素时，n 个 Hash 函数对元素进行计算, 得到 n 个 slot，然后将 Bit 数组中 n 个 slot 的 Bit 置 1。

当我们要判断一个元素是否在集合中时，还是通过相同的 n 个 Hash 函数计算 Hash 值，如果所有 Hash 值在布隆过滤器里对应的 Bit 不全为 1，则该元素不存在。当对应 Bit 全 1 时, 则元素的存在与否, 无法确定.  这是因为布隆过滤器的位数有限,  由该元素计算出的 slot, 恰好全部和其他元素的 slot 冲突.  所以全 1 情形, 需要回源查找才能判断元素的存在性。

### **2 什么是 Bloom Filter 索引**

StarRocks 的建表时, 可通过 PROPERTIES{" bloom\_filter\_columns "=" c1, c2, c3 "}指定需要建 BloomFilter 索引的列，查询时, BloomFilter 可快速判断某个列中是否存在某个值。如果 Bloom Filter 判定该列中不存在指定的值，就不需要读取数据文件；如果是全 1 情形，此时需要读取数据块确认目标值是否存在。另外，Bloom Filter 索引无法确定具体是哪一行数据具有该指定的值。

## 适用场景

满足以下几个条件时可以考虑对某列建立 Bloom Filter 索引：

1. 首先 BloomFilter 也适用于非前缀过滤。
2. 查询会根据该列高频过滤，而且查询条件大多是 in 和 =。
3. 不同于 Bitmap, BloomFilter 适用于高基数列。

## 如何使用

### **1 创建索引**

建表时使用指定 bloom\_filter\_columns 即可：

~~~ SQL
PROPERTIES ( "bloom_filter_columns" = "k1, k2, k3" )
~~~

### **2 查看索引**

展示指定 table\_name 下的 Bloom Filter 索引：

~~~ SQL
SHOW CREATE TABLE table_name;
~~~

### **3 删除索引**

删除索引即为将索引列从 bloom\_filter\_columns 属性中移除：

~~~SQL
ALTER TABLE example_db.my_table SET ("bloom_filter_columns" = "");
~~~

### **4 修改索引**

修改索引即为修改表的 bloom\_filter\_columns 属性：

~~~SQL
ALTER TABLE example_db.my_table SET ("bloom_filter_columns" = "k1, k2, k3");
~~~

## 注意事项

* 不支持对 Tinyint、Float、Double 类型的列建 Bloom Filter 索引。
* Bloom Filter 索引只对 in 和 = 过滤查询有加速效果。
* 如果要查看某个查询是否命中了 Bloom Filter 索引，可以通过查询的 Profile 信息查看（TODO：加上查看 Profile 的链接）。
