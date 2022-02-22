# CloudCanal 数据导入常见问题

## 导入时任务异常，报错 close index failed/too many tablet versions

### CloudCanal 侧解决方法

- 打开任务详情
- 打开任务参数
- 调节 fullBatchWaitTimeMs 和 increBatchWaitTimeMs 参数，增加一批数据写入 StarRocks 之后的停顿时间，避免写入过于频繁导致的异常报错

![image.png](../../assets/8.2.1.9-1.png)

![image.png](../../assets/8.2.1.9-2.png)

### StarRocks 侧解决办法

调整 compaction 策略，加快合并(调整完需要观察内存和 IO)，在 be.conf 中修改以下内容

```properties
cumulative_compaction_num_threads_per_disk = 4
base_compaction_num_threads_per_disk = 2
cumulative_compaction_check_interval_seconds = 2
```
