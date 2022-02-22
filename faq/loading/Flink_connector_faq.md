# Flink Connector 常见问题

## flink-connector-jdbc_2.11sink 到 StarRocks 时间落后 8 小时

**问题描述：**

Flink 中 localtimestap 函数生成的时间，在 Flink 中时间正常，sink 到 StarRocks 后发现时间落后 8 小时。已确认 Flink 所在服务器与 StarRocks 所在服务器时区均为 Asia/ShangHai 东八区。Flink 版本为 1.12，驱动为 flink-connector-jdbc_2.11，需要如何处理？

**解决方案：**

可以在 Flink sink 表中配置时区参数'server-time-zone' = 'Asia/Shanghai'，或同时在 jdbc url 里添加&serverTimezone = Asia/Shanghai。示例如下：

```sql
CREATE TABLE sk (
    sid int,
    local_dtm TIMESTAMP,
    curr_dtm TIMESTAMP
)
WITH (
    'connector' = 'jdbc',
    'url' = 'jdbc:mysql://192.168.110.66:9030/sys_device?characterEncoding=utf-8&serverTimezone=Asia/Shanghai',
    'table-name' = 'sink',
    'driver' = 'com.mysql.jdbc.Driver',
    'username' = 'doris',
    'password' = 'doris123',
    'server-time-zone' = 'Asia/Shanghai'
);
```

## [Flink 导入] 在 starrocks 集群上部署的 kafka 集群的数据可以导入，其他 kafka 集群无法导入

**问题描述：**

```SQL
failed to query wartermark offset, err: Local: Bad message format
```

**解决方案：**

kafka 通信会用到 hostname，需要在 starrocks 集群节点配置 kafka 主机名解析/etc/hosts

## 没有查询时 BE 内存处于打满状态，且 cpu 也是打满状态

**问题描述:**

打满是什么原因引起的？

**解决方案:**

因为会定期收集统计信息，不是长期占用 cpu，10g 以内的内存使用完不会释放，be 会自己管理，可以通过 tc_use_memory_min 限制大小。

tc_use_memory_min，默认等于 10737418240。解释：TCmalloc 最小保留内存，只有超过这个值，StarRocks 才将空闲内存返还给操作系统，可以去到 BE 配置文件 " be.conf" 中去配置

## Be 申请的内存不会释放给操作系统

这是一种正常想象, 因为数据库从操作系统获得的大块的内存分配，在分配的时候会多预留，释放时候会延后，为了重复利用，因为大块内存的分配的代价比较大. 建议测试环境验证时，对内存使用进行监控，在较长的周期内看内存是否能够完成释放

## 关于下载 flink connector 后不生效问题

**问题描述：**

该包需要通过阿里云镜像地址来获取

**解决方案:**

请确认 /etc/maven/settings.xml 的 mirror 部分是否配置了全部通过阿里云镜像获取。

 如果是请修改为如下：

 <mirror>
    <id>aliyunmaven </id>
    <mirrorf>central</mirrorf>
    <name>阿里云公共仓库</name>
    <url>https：//maven.aliyun.com/repository/public</url>
</mirror>

## Flink-connector-StarRocks 中 sink.buffer-flush.interval-ms 参数的含义

**问题描述:**

```plain text
+----------------------+--------------------------------------------------------------+
|         Option       | Required |  Default   | Type   |       Description           |
+-------------------------------------------------------------------------------------+
|  sink.buffer-flush.  |  NO      |   300000   | String | the flushing time interval, |
|  interval-ms         |          |            |        | range: [1000ms, 3600000ms]  |
+----------------------+--------------------------------------------------------------+
```

如果这个参数设置为 15s，但是 checkpoint interval = 5mins, 那么这个值还生效么？

**解决方案:**

三个阈值先达到其中的哪一个，那一个就先生效，是和 checkpoint interval 设置的值没关系的，checkpoint interval 对于 exactly once 才有效，at_least_once 用 interval-ms
