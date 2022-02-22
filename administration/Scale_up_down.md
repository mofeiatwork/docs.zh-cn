# 扩容缩容

## FE 扩缩容

StarRocks 有两种 FE 节点：Follower 和 Observer。Follower 参与选举投票和写入，Observer 只用来同步日志，扩展读性能。

FE 扩缩容时要注意：

* Follower FE(包括 Master)的数量必须为奇数，建议部署 3 个，组成高可用(HA)模式即可。
* 当 FE 处于高可用部署时（1 个 Master，2 个 Follower），建议通过增加 Observer FE 来扩展 FE 的读服务能力。当然也可以继续增加 Follower FE，但几乎是不必要的。
* 通常一个 FE 节点可以应对 10-20 台 BE 节点。建议总的 FE 节点数量在 10 个以下。而 3 个即可满足绝大部分需求。

### FE 扩容

部署好 FE 节点，启动完成服务。

~~~sql
bin/start_be.sh --helper "fe_host: edit_log_port" --daemon ;
--fe_host 为 master 节点的 ip
~~~

通过命令扩容 FE 节点。

~~~sql
alter system add follower "fe_host:edit_log_port";
alter system add observer "fe_host:edit_log_port";
~~~

### FE 缩容

缩容和扩容命令类似

~~~sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
~~~

扩缩容完成后可以通过 `show proc '/frontends';` 查看节点信息

## BE 扩缩容

BE 扩缩容后，StarRocks 会自动根据负载情况，进行数据均衡，期间不影响使用。

### BE 扩容

* 运行命令进行扩容

~~~sql
alter system add backend 'be_host:be_heartbeat_service_port';
~~~

* 运行命令查看 BE 状态

~~~sql
show proc '/backends';
~~~

### BE 缩容

缩容 BE 有两种方式： DROP 和 DECOMMISSION。

DROP 会立刻删除 BE 节点，丢失的副本由 FE 调度补齐；DECOMMISSION 先保证副本补齐，然后再下掉 BE 节点。DECOMMISSION 方式更加友好一点，建议采用这种方式进行缩容。

二者的命令类似：

* `alter system decommission backend "be_host:be_heartbeat_service_port";`
* `alter system drop backend "be_host:be_heartbeat_service_port";`

Drop backend 是一个危险操作所以需要二次确认后执行

* `alter system drop backend "be_host:be_heartbeat_service_port";`

FE 和 BE 扩容之后的状态，也可以通过查看 [集群状态](../administration/Cluster_administration.md#确认集群健康状态) 一节中的页面进行查看。
