# Stream load

StarRocks 支持从本地直接导入数据，支持 CSV 文件格式。数据量在 10GB 以下。

Stream Load 是一种同步的导入方式，用户通过发送 HTTP 请求将本地文件或数据流导入到 StarRocks 中。Stream Load 同步执行导入并返回导入结果。用户可直接通过请求的返回值判断导入是否成功。

---

## 名词解释

* **Coordinator**：协调节点。负责接收数据并分发数据到其他数据节点，导入完成后返回结果给用户。

---

## 基本原理

Stream Load 中，用户通过 HTTP 协议提交导入命令。如果提交到 FE 节点，则 FE 节点会通过 HTTP redirect 指令将请求转发给某一个 BE 节点，用户也可以直接提交导入命令给某一指定 BE 节点。该 BE 节点作为 Coordinator 节点，将数据按表 schema 划分并分发数据到相关的 BE 节点。导入的最终结果由 Coordinator 节点返回给用户。

下图展示了 Stream Load 的主要流程:

![stream load](../assets/4.4.2-1.png)

---

## 导入示例

### 创建导入任务

Stream Load 通过 HTTP 协议提交和传输数据。这里通过 curl 命令展示如何提交导入。用户也可以通过其他 HTTP client 进行操作。

**语法：**

~~~bash
curl --location-trusted -u user: passwd [-H ""...] -T data.file -XPUT \
    http://fe_host: http_port/api/{db}/{table}/_stream_load
~~~

Header 中支持的属性见下文的导入任务参数说明，格式为: -H "key1: value1"。如果同时有多个任务参数，需要用多个 -H 来指示，类似于 \-H "key1: value1" -H "key2: value2"……

**示例：**

~~~bash
curl --location-trusted -u root -T date -H "label: 123" \
    http://abc.com: 8030/api/test/date/_stream_load
~~~

创建导入任务的详细语法可执行 HELP STREAM LOAD 查看，下面介绍该命令中部分参数的意义。

**说明：**

* 当前支持 HTTP chunked 与非 chunked 上传两种方式，对于非 chunked 方式，必须要有 Content-Length 来标示上传内容长度，这样能够保证数据的完整性。
* 用户最好设置 Expect Header 字段内容 100-continue，这样可以在某些出错场景下避免不必要的数据传输。

**签名参数：**

* user/passwd，Stream Load 创建导入任务使用的是 HTTP 协议，可通过 Basic access authentication 进行签名。StarRocks 系统会根据签名来验证用户身份和导入权限。

**导入任务参数：**

Stream Load 中所有与导入任务相关的参数均设置在 Header 中。下面介绍其中部分参数的意义：

* **label** : 导入任务的标签，相同标签的数据无法多次导入。用户可以通过指定 Label 的方式来避免一份数据重复导入的问题。当前 StarRocks 系统会保留最近 30 分钟内成功完成的任务的 Label。
* **column_separator** ：用于指定导入文件中的列分隔符，默认为\\t。如果是不可见字符，则需要加\\x 作为前缀，使用十六进制来表示分隔符。如 Hive 文件的分隔符\\x01，需要指定为\-H "column_separator:\\x01"
* **row_delimiter**: 用户指定导入文件中的行分隔符，默认为 "\\n"。请注意，curl 无法解释传递 "\\n"，换行符手动指定为 "\\n" 时，shell 会先传递 "\\"，然后传递 "n" 而不是直接传递换行符 "\\n"。Bash 支持另一种转义字符串语法，如要传递 "\n" 和 "\t" 时，请使用 "$'" 启动字符串并以 "'" 结束字符串，eg：`-H $'row_delimiter:\n'`。
* **columns** ：用于指定导入文件中的列和 table 中的列的对应关系。如果源文件中的列正好对应表中的内容，那么无需指定该参数。如果源文件与表 schema 不对应，那么需要这个参数来配置数据转换规则。这里有两种形式的列，一种是直接对应于导入文件中的字段，可直接使用字段名表示；一种需要通过计算得出。举几个例子帮助理解：

  * 例 1：表中有 3 列 "c1, c2, c3"，源文件中的 3 列依次对应的是 "c3, c2, c1"; 那么需要指定\-H "columns: c3, c2, c1"
  * 例 2：表中有 3 列 "c1, c2, c3" ，源文件中前 3 列一一对应，但是还有多余 1 列；那么需要指定\-H "columns: c1, c2, c3, temp"，最后 1 列随意指定名称用于占位即可。
  * 例 3：表中有 3 个列“year, month, day "，源文件中只有一个时间列，为”2018-06-01 01: 02: 03“格式；那么可以指定 \-H " columns: col, year = year(col), month = month(col), day = day(col)" 完成导入。

* **where**: 用于抽取部分数据。用户如需将不需要的数据过滤掉，那么可以通过设定这个选项来达到。

  * 例 1：只导入 k1 列等于 20180601 的数据，那么可以在导入时指定\-H "where: k1 = 20180601"。

* **max_filter_ratio**：最大容忍可过滤（数据不规范等原因而过滤）的数据比例。默认零容忍。数据不规范不包括通过 where 条件过滤掉的行。
* **partitions**: 用于指定这次导入所涉及的 partition。如果用户能够确定数据对应的 partition，推荐指定该项。不满足这些分区的数据将被过滤掉。比如指定导入到 p1、p2 分区：\-H "partitions: p1, p2"。
* **timeout**: 指定导入的超时时间。单位秒，默认是 600 秒。可设置范围为 1 秒 ~ 259200 秒。
* **strict_mode**: 用户指定此次导入是否开启严格模式，默认为开启。关闭方式为：\-H "strict_mode: false"。
* **timezone**: 指定本次导入所使用的时区。默认为东八区。该参数会影响所有导入涉及的和时区有关的函数结果。
* **exec_mem_limit**: 导入内存限制。默认为 2GB。单位是「字节」。

**返回结果：**

导入完成后，Stream Load 会以 Json 格式返回这次导入的相关内容，示例如下：

~~~json
{
    "TxnId": 1003,
    "Label": "b6f3bc78-0d2c-45d9-9e4c-faa0a0149bee",
    "Status": "Success",
    "ExistingJobStatus": "FINISHED", // optional
    "Message": "OK",
    "NumberTotalRows": 1000000,
    "NumberLoadedRows": 1000000,
    "NumberFilteredRows": 1,
    "NumberUnselectedRows": 0,
    "LoadBytes": 40888898,
    "LoadTimeMs": 2144,
    "ErrorURL": "[http://192.168.1.1: 8042/api/_load_error_log?file =_ _shard_0/error_log_insert_stmt_db18266d4d9b4ee5-abb00ddd64bdf005_db18266d4d9b4ee5_abb00ddd64bdf005](http://192.168.1.1: 8042/api/_load_error_log?file =__shard_0/error_log_insert_stmt_db18266d4d9b4ee5-abb00ddd64bdf005_db18266d4d9b4ee5_abb00ddd64bdf005)"
}
~~~

* TxnId：导入的事务 ID。用户可不感知。
* Status: 导入最后的状态。
* Success：表示导入成功，数据已经可见。
* Publish Timeout：表述导入作业已经成功 Commit，但是由于某种原因并不能立即可见。用户可以视作已经成功不必重试导入。
* Label Already Exists：表明该 Label 已经被其他作业占用，可能是导入成功，也可能是正在导入。
* 其他：此次导入失败，用户可以指定 Label 重试此次作业。
* Message: 导入状态的详细说明。失败时会返回具体的失败原因。
* NumberTotalRows: 从数据流中读取到的总行数。
* NumberLoadedRows: 此次导入的数据行数，只有在 Success 时有效。
* NumberFilteredRows: 此次导入过滤掉的行数，即数据质量不合格的行。
* NumberUnselectedRows: 此次导入，通过 where 条件被过滤掉的行数。
* LoadBytes: 此次导入的源文件数据量大小。
* LoadTimeMs: 此次导入所用的时间(ms)。
* ErrorURL: 被过滤数据的具体内容，仅保留前 1000 条。如果导入任务失败，可以直接用以下方式获取被过滤的数据，并进行分析，以调整导入任务。

    ~~~bash
    wget http://192.168.1.1: 8042/api/_load_error_log?file =__shard_0/error_log_insert_stmt_db18266d4d9b4ee5-abb00ddd64bdf005_db18266d4d9b4ee5_abb00ddd64bdf005
    ~~~

### 取消导入

用户无法手动取消 Stream Load，Stream Load 在超时或者导入错误后会被系统自动取消。

---

## 最佳实践

### 应用场景

Stream Load 的最佳使用场景是原始文件在内存中或者存储在本地磁盘中。其次由于 Stream Load 是一种同步的导入方式，所以用户如果希望用同步方式获取导入结果，也可以使用这种导入。

### 数据量

由于 Stream Load 是由 BE 发起的导入并分发数据，建议的导入数据量在 1GB 到 10GB 之间。系统默认的最大 Stream Load 导入数据量为 10GB，所以如果要导入超过 10GB 的文件需要修改 BE 的配置项 streaming_load_max_mb。比如，待导入文件大小为 15G，则可修改 BE 的该配置项大于 15G, 例如设置为 `streaming_load_max_mb = 16000` 即可。

Stream Load 的默认超时为 300 秒，按照 StarRocks 目前最大的导入限速来看，导入超过 3GB 大小的文件就需要修改导入任务默认的超时时间了。

`导入任务超时时间 = 导入数据量 / 10M/s` （具体的平均导入速度需要用户根据自己的集群情况计算）

例如：导入一个 10GB 的文件，timeout 应该设为 1000s。

### 完整例子

**数据情况**：数据在客户端本地磁盘路径 /home/store-sales 中，导入的数据量约为 15GB，希望导入到数据库 bj-sales 的表 store-sales 中。

**集群情况**：Stream Load 的并发数不受集群大小影响。

* step1: 导入文件大小超过默认的最大导入大小 10GB，所以要修改 BE 的配置文件 BE.conf, 例如将最大导入大小设置为 16000：

`streaming_load_max_mb = 16000`

* step2: 计算大概的导入时间是否超过默认 timeout 值，导入时间 ≈ 15000 / 10 = 1500s，超过了默认的 timeout 时间，需要修改 FE 的配置 FE.conf：

`stream_load_default_timeout_second = 1500`

* step3：创建导入任务

~~~bash
curl --location-trusted -u user:password -T /home/store_sales \
    -H "label:abc" [http://abc.com:8000/api/bj_sales/store_sales/_stream_load](http://abc.com:8000/api/bj_sales/store_sales/_stream_load)
~~~

### 代码集成示例

* JAVA 开发 stream load，参考：[https://github.com/StarRocks/demo/MiscDemo/stream_load](https://github.com/StarRocks/demo/tree/master/MiscDemo/stream_load)
* Spark 集成 stream load，参考： [01_sparkStreaming2StarRocks](https://github.com/StarRocks/demo/blob/master/docs/01_sparkStreaming2StarRocks.md)

---

## 常见问题

* 数据质量问题报错：ETL-QUALITY-UNSATISFIED; msg: quality not good enough to cancel

可参考章节导入总览/常见问题。

* Label Already Exists

可参考章节导入总览/常见问题。由于 Stream Load 是采用 HTTP 协议提交创建导入任务，一般各个语言的 HTTP Client 均会自带请求重试逻辑。StarRocks 系统在接受到第一个请求后，已经开始操作 Stream Load，但是由于没有及时向 Client 端返回结果，Client 端会发生再次重试创建请求的情况。这时候 StarRocks 系统由于已经在操作第一个请求，所以第二个请求会遇到 Label Already Exists 的情况。排查上述可能的方法：使用 Label 搜索 FE Master 的日志，看是否存在同一个 Label 出现了两次的情况。如果有就说明 Client 端重复提交了该请求。

建议用户根据当前请求的数据量，计算出大致的导入耗时，并根据导入超时时间改大 Client 端的请求超时时间，避免请求被 Client 端多次提交。
