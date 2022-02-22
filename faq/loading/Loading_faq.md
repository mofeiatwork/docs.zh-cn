# 导入通用 FAQ

## 导入常见问题

### 异常：close index channel failed/too many tablet versions

**问题描述：**

导入频率太快，compaction 没能及时合并导致版本数过多，默认版本数 1000

**解决方案：**

增大单次导入数据量，降低频率

调整 compaction 策略，加快合并（调整完需要观察内存和 io）, 在 be.conf 中修改以下内容

```plain text
cumulative_compaction_num_threads_per_disk = 4
base_compaction_num_threads_per_disk = 2
cumulative_compaction_check_interval_seconds = 2
```
