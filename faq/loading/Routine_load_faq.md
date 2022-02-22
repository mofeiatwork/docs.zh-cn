# Routine Load 常见问题

## kafka 生产的 mysql binlog 数据算不算文本格式数据？

通过 canal 这种工具导入 kafka 以后是个 json 的格式。

## 从 kafka 消费数据再写入 StarRocks，直连 StarRocks，「语义一致性」能保证吗？

可以。

## recompile librdkafka with libsasl2 or openssl support

**问题描述：**

Routine load 到 kafka 集群失败报错，异常信息：

```plain text
ErrorReason{errCode = 4, msg='Job failed to fetch all current partition with error [Failed to send proxy request: failed to send proxy request: [PAUSE: failed to create kafka consumer: No provider for SASL mechanism GSSAPI: recompile librdkafka with libsasl2 or openssl support. Current build options: PLAIN SASL_SCRAM]]'}
```

**问题原因：**

当前 librdkafka 不支持 sasl 认证

**解决方案：**

重新编译 librdkafka，可参考 [编译 librdkafka](https://www.codeleading.com/article/8708676746/)。
