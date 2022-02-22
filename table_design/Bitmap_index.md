# Bitmap 索引

StarRocks 支持基于 Bitmap 索引，对于有 Filter 的查询有明显的加速效果。

## 原理

### **1 什么是 Bitmap**

Bitmap 是元素为 bit 的， 取值为 0、1 两种情形的, 可对某一位 bit 进行置位(set)和清零(clear)操作的数组。Bitmap 的使用场景有：

* 用一个 long 型表示 32 位学生的性别，0 表示女生，1 表示男生。
* 用 Bitmap 表示一组数据中是否存在 null 值，0 表示元素不为 null，1 表示为 null。
* 一组数据的取值为(Q1, Q2, Q3, Q4)，表示季度，用 Bitmap 表示这组数据中取值为 Q4 的元素，1 表示取值为 Q4 的元素, 0 表示其他取值的元素。

### **2 什么是 Bitmap 索引**

![Bitmap 索引](../assets/3.6.1-1.png)

Bitmap 只能表示取值为两种情形的列数组, 当列的取值为多种取值情形枚举类型时, 例如季度(Q1, Q2, Q3, Q4),  系统平台(Linux, Windows, FreeBSD, MacOS), 则无法用一个 Bitmap 编码; 此时可以为每个取值各自建立一个 Bitmap 的来表示这组数据; 同时为实际枚举取值建立词典。

如上图所示，Platform 列有 4 行数据，可能的取值有 Android、Ios。StarRocks 中会首先针对 Platform 列构建一个字典，将 Android 和 Ios 映射为 int，然后就可以对 Android 和 Ios 分别构建 Bitmap。具体来说，我们分别将 Android、Ios 编码为 0 和 1，因为 Android 出现在第 1，2，3 行，所以 Bitmap 是 0111，因为 Ios 出现在第 4 行，所以 Bitmap 是 1000。

假如有一个针对包含该 Platform 列的表的 SQL 查询，select xxx from table where Platform = iOS，StarRocks 会首先查找字典，找出 iOS 对于的编码值是 1，然后再去查找 Bitmap Index，知道 1 对应的 Bitmap 是 1000，我们就知道只有第 4 行数据符合查询条件，StarRocks 就会只读取第 4 行数据，不会读取所有数据。

## 适用场景

### **1 非前缀过滤**

StarRocks 对于建表中的前置列可以通过 shortkey 索引快速过滤，但是对于非前置列, 无法利用 shortkey 索引快速过滤，如果需要对非前置列进行快速过滤，就可以对这些列建立 Bitmap 索引。

### **2 多列过滤 Filter**

由于 Bitmap 可以快速的进行 bitwise 运算。所以在多列过滤的场景中，也可以考虑对每列分别建立 Bitmap 索引。

## 如何使用

### **1 创建索引**

在 table1 上为 site\_id 列创建 Bitmap 索引：

~~~ SQL
CREATE INDEX index_name ON table1 (site_id) USING BITMAP COMMENT 'balabala';
~~~

### **2 查看索引**

展示指定 table\_name 的下索引：

~~~ SQL
SHOW INDEX FROM example_db.table_name;
~~~

### **3 删除索引**

下面语句可以从一个表中删除指定名称的索引：

~~~ SQL
DROP INDEX index_name ON [db_name.] table_name;
~~~

## 注意事项

1. 对于明细模型，所有列都可以建 Bitmap 索引；对于聚合模型，只有 Key 列可以建 Bitmap 索引。
2. Bitmap 索引, 应该在取值为枚举型, 取值大量重复, 较低基数, 并且用作等值条件查询或者可转化为等值条件查询的列上创建。
3. 不支持对 Float、Double、Decimal 类型的列建 Bitmap 索引。
4. 如果要查看某个查询是否命中了 Bitmap 索引，可以通过查询的 [Profile](https://docs.starrocks.com/zh-cn/main/administration/Query_planning#profile%E5%88%86%E6%9E%90) 信息查看。
