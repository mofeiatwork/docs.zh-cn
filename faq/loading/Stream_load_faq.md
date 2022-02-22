# Stream Load 常见问题

## Stream Load 是否可识别文本文件中首行的列名？或者能否指定不读取第一行？

Stream Load 不能识别文本中首行的列名，首行对 Stream Load 来说也只是普通数据。当前还不能指定不读取首行，若需要导入的文件首行为列名，可以使用如下四种方式处理：

1. 在导出工具中修改设置，重新导出不带列名的文本类数据文件。
2. 使用 sed -i '1d' filename 等命令删除文本类文件首行。
3. 在 Stream Load 语句中使用 `-H "where: 列名 != '列名称'"` 把该行过滤掉。首行字符串转其他类型的转不过去就会返回 null，所以该种方式要求表中字段不能设置的有 not null。
4. 在 Stream Load 命令中加入 `-H "max_filter_ratio:0.01"`，根据数据量给它一个“1%或者更小但能容错超过 1 行的容错率”，将首行的错误忽视掉。加入容错率后，返回结果的 ErrorURL 依旧会提示有错误，但任务整体会成功。容错率不宜设置过大，避免漏掉其他数据问题。

## 当前业务的分区键对应的数据不是标准的 date 和 int，比如是 202106.00 的格式，假如需要使用 Stream Load 导入到 StarRocks 中，要如何转换？

StarRocks 支持在导入过程中进行数据转换，具体可以参考企业版文档“4.7 导入过程中完成数据转换”。

以 Stream Load 为例，假设表 TEST 中有 NO、DATE、VERSION、PRICE 四列，导出的 CSV 数据文件中 DATE 字段内容是不规范的 202106.00 格式。如果在 StarRocks 中需使用的分区列为 DATE，那么首先我们需要在 StarRocks 中进行建表，指定 DATE 类型为 date、datetime 或 int。之后，在 Stream Load 命令中，使用：

```plain text
-H "columns: NO,DATE_1, VERSION, PRICE, DATE=LEFT(DATE_1,6)"
```

来实现列的转换。DATE_1 可以简单的认为是先占位进行取数，然后通过函数转换，赋值给 StarRocks 中对应的字段。特别注意，我们需要先列出 CSV 数据文件中的所有列，再进行函数转换，常规函数这里都可以使用。
