# 手动部署

手动部署可以让用户快速体验 StarRocks, 积累 StarRocks 的系统运维经验.  生产环境部署, 请使用管理平台和自动部署。

## 获取二进制产品包

请您联系 StarRocks 的技术支持或者销售人员获取最新稳定版的 StarRocks 二进制产品包。

比如您获得的产品包为 starrocks-1.0.0.tar.gz, 解压(tar -xzvf starrocks-1.0.0.tar.gz)后内容如下:

```Plain Text
StarRocks-XX-1.0.0
├── be  # BE目录
│   ├── bin
│   │   ├── start_be.sh # BE启动命令
│   │   └── stop_be.sh  # BE关闭命令
│   ├── conf
│   │   └── be.conf     # BE配置文件
│   ├── lib
│   │   ├── starrocks_be  # BE可执行文件
│   │   └── meta_tool
│   └── www
├── fe  # FE目录
│   ├── bin
│   │   ├── start_fe.sh # FE启动命令
│   │   └── stop_fe.sh  # FE关闭命令
│   ├── conf
│   │   └── fe.conf     # FE配置文件
│   ├── lib
│   │   ├── starrocks-fe.jar  # FE jar包
│   │   └── *.jar           # FE 依赖的jar包
│   └── webroot
└── udf
```

## 环境准备

准备三台物理机, 需要以下环境支持：

* Linux (Centos 7+)
* Java 1.8+

CPU 需要支持 AVX2 指令集，cat /proc/cpuinfo |grep avx2 有结果输出表明 CPU 支持，如果没有支持，建议更换机器，StarRocks 使用向量化技术需要一定的指令集支持才能发挥效果。

将 StarRocks 的二进制产品包分发到目标主机的部署路径并解压，可以考虑使用新建的 StarRocks 用户来管理。

## 部署 FE

### FE 的基本配置

FE 的配置文件为 StarRocks-XX-1.0.0/fe/conf/fe.conf, 默认配置已经足以启动集群, 有经验的用户可以查看手册的系统配置章节, 为生产环境定制配置，为了让用户更好的理解集群的工作原理, 此处只列出基础配置。

### FE 单实例部署

```bash
cd StarRocks-XX-1.0.0/fe
```

第一步: 定制配置文件 conf/fe.conf：

```bash
JAVA_OPTS = "-Xmx4096m -XX:+UseMembar -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=7 -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSClassUnloadingEnabled -XX:-CMSParallelRemarkEnabled -XX:CMSInitiatingOccupancyFraction=80 -XX:SoftRefLRUPolicyMSPerMB=0 -Xloggc:$STARROCKS_HOME/log/fe.gc.log"
```

可以根据 FE 内存大小调整 -Xmx4096m，为了避免 GC 建议 16G 以上，StarRocks 的元数据都在内存中保存。
<br/>

第二步: 创建元数据目录:

```bash
mkdir -p meta (1.19.x及以前的版本需要使用mkdir -p doris-meta)
```

<br/>

第三步: 启动 FE 进程:

```bash
bin/start_fe.sh --daemon
```

<br/>

第四步: 确认启动 FE 启动成功。

* 查看日志 log/fe.log 确认。

```Plain Text
2020-03-16 20:32:14,686 INFO 1 [FeServer.start():46] thrift server started.

2020-03-16 20:32:14,696 INFO 1 [NMysqlServer.start():71] Open mysql server success on 9030

2020-03-16 20:32:14,696 INFO 1 [QeService.start():60] QE service start.

2020-03-16 20:32:14,825 INFO 76 [HttpServer$HttpServerThread.run():210] HttpServer started with port 8030

...
```

* 如果 FE 启动失败，可能是由于端口号被占用，修改配置文件 conf/fe.conf 中的端口号 http_port。
* 使用 jps 命令查看 java 进程确认 "StarRocksFe" 存在。
* 使用浏览器访问 8030 端口, 打开 StarRocks 的 WebUI, 用户名为 root, 密码为空。

### 使用 MySQL 客户端访问 FE

第一步: 安装 mysql 客户端(如果已经安装，可忽略此步)：

Ubuntu：sudo apt-get install mysql-client

Centos：sudo yum install mysql-client
<br/>

第二步: 使用 mysql 客户端连接：

```sql
mysql -h 127.0.0.1 -P9030 -uroot
```

注意：这里默认 root 用户密码为空，端口为 fe/conf/fe.conf 中的 query\_port 配置项，默认为 9030
<br/>

第三步: 查看 FE 状态：

```Plain Text
mysql> SHOW PROC '/frontends'\G

************************* 1. row ************************
             Name: 172.16.139.24_9010_1594200991015
               IP: 172.16.139.24
         HostName: starrocks-sandbox01
      EditLogPort: 9010
         HttpPort: 8030
        QueryPort: 9030
          RpcPort: 9020
             Role: FOLLOWER
         IsMaster: true
        ClusterId: 861797858
             Join: true
            Alive: true
ReplayedJournalId: 64
    LastHeartbeat: 2020-03-23 20:15:07
         IsHelper: true
           ErrMsg:
1 row in set (0.03 sec)
```

<br/>

Role 为 FOLLOWER 说明这是一个能参与选主的 FE；IsMaster 为 true，说明该 FE 当前为主节点。
<br/>

如果 MySQL 客户端连接不成功，请查看 log/fe.warn.log 日志文件，确认问题。由于是初次启动，如果在操作过程中遇到任何意外问题，都可以删除并重新创建 FE 的元数据目录，再从头开始操作。
<br/>

### FE 的高可用集群部署

FE 的高可用集群采用主从复制架构, 可避免 FE 单点故障. FE 采用了类 raft 的 bdbje 协议完成选主, 日志复制和故障切换. 在 FE 集群中, 多实例分为两种角色: follower 和 observer; 前者为复制协议的可投票成员, 参与选主和提交日志, 一般数量为奇数(2n+1), 使用多数派(n+1)确认, 可容忍少数派(n)故障; 而后者属于非投票成员, 用于异步订阅复制日志, observer 的状态落后于 follower, 类似其他复制协议中的 learner 角色。
<br/>

FE 集群从 follower 中自动选出 master 节点, 所有更改状态操作都由 master 节点执行, 从 FE 的 master 节点可以读到最新的状态. 更改操作可以从非 master 节点发起, 继而转发给 master 节点执行,  非 master 节点记录最近一次更改操作在复制日志中的 LSN, 读操作可以直接在非 master 节点上执行, 但需要等待非 master 节点的状态已经同步到最近一次更改操作的 LSN, 因此读写非 Master 节点满足顺序一致性. Observer 节点能够增加 FE 集群的读负载能力, 时效性要求放宽的非紧要用户可以读 observer 节点。
<br/>

FE 节点之间的时钟相差不能超过 5s, 使用 NTP 协议校准时间。

一台机器上只可以部署单个 FE 节点。所有 FE 节点的 http\_port 需要相同。
<br/>

集群部署按照下列步骤逐个增加 FE 实例。

第一步: 分发二进制和配置文件, 配置文件和单实例情形相同。
<br/>

第二步: 使用 MySQL 客户端连接已有的 FE,  添加新实例的信息，信息包括角色、ip、port：

```sql
mysql> ALTER SYSTEM ADD FOLLOWER "host:port";
```

或

```sql
mysql> ALTER SYSTEM ADD OBSERVER "host:port";
```

host 为机器的 IP，如果机器存在多个 IP，需要选取 priority\_networks 里的 IP，例如 priority\_networks = 192.168.1.0/24 可以设置使用 192.168.1.x 这个子网进行通信。port 为 edit\_log\_port，默认为 9010。

> StarRocks 的 FE 和 BE 因为安全考虑都只会监听一个 IP 来进行通信，如果一台机器有多块网卡，可能 StarRocks 无法自动找到正确的 IP，例如 ifconfig 命令能看到  eth0 ip 为 192.168.1.1, docker0:  172.17.0.1 ，我们可以设置 192.168.1.0/24 这一个子网来指定使用 eth0 作为通信的 IP，这里采用是 [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 的表示方法来指定 IP 所在子网范围，这样可以在所有的 BE，FE 上使用相同的配置。
> priority\_networks 是 FE 和 BE 相同的配置项，写在 fe.conf 和 be.conf 中。该配置项用于在 FE 或 BE 启动时，告诉进程应该绑定哪个 IP。示例如下：
> `priority_networks=10.1.3.0/24`

如出现错误，需要删除 FE，应用下列命令：

```sql
alter system drop follower "fe_host:edit_log_port";
alter system drop observer "fe_host:edit_log_port";
```

具体参考 [扩容缩容](../administration/Scale_up_down.md)。
<br/>

第三步: FE 节点之间需要两两互联才能完成复制协议选主, 投票，日志提交和复制等功能。 FE 节点首次启动时，需要指定现有集群中的一个节点作为 helper 节点, 从该节点获得集群的所有 FE 节点的配置信息，才能建立通信连接，因此首次启动需要指定--helper 参数：

```shell
./bin/start_fe.sh --helper host:port --daemon
```

host 为 helper 节点的 IP，如果有多个 IP，需要选取 priority\_networks 里的 IP。port 为 edit\_log\_port，默认为 9010。

当 FE 再次启动时，无须指定--helper 参数，因为 FE 已经将其他 FE 的配置信息存储于本地目录, 因此可直接启动：

```shell
./bin/start_fe.sh --daemon
```

<br/>

第四步: 查看集群状态, 确认部署成功：

```Plain Text
mysql> SHOW PROC '/frontends'\G

********************* 1. row **********************
    Name: 172.26.108.172_9010_1584965098874
      IP: 172.26.108.172
HostName: starrocks-sandbox01
......
    Role: FOLLOWER
IsMaster: true
......
   Alive: true
......
********************* 2. row **********************
    Name: 172.26.108.174_9010_1584965098874
      IP: 172.26.108.174
HostName: starrocks-sandbox02
......
    Role: FOLLOWER
IsMaster: false
......
   Alive: true
......
********************* 3. row **********************
    Name: 172.26.108.175_9010_1584965098874
      IP: 172.26.108.175
HostName: starrocks-sandbox03
......
    Role: FOLLOWER
IsMaster: false
......
   Alive: true
......
3 rows in set (0.05 sec)
```

节点的 Alive 显示为 true 则说明添加节点成功。以上例子中，

172.26.108.172\_9010\_1584965098874 为主 FE 节点。

## 部署 BE

### BE 的基本配置

BE 的配置文件为 StarRocks-XX-1.0.0/be/conf/be.conf, 默认配置已经足以启动集群, 不建议初尝用户修改配置, 有经验的用户可以查看手册的系统配置章节, 为生产环境定制配置. 为了让用户更好的理解集群的工作原理, 此处只列出基础配置。

### BE 部署

用户可使用下面命令添加 BE 到 StarRocks 集群, 一般至少部署 3 个 BE 实例, 每个实例的添加步骤相同。

```shell
cd StarRocks-XX-1.0.0/be
```

第一步: 创建数据目录：

```shell
mkdir -p storage
```

<br/>

第二步: 通过 mysql 客户端添加 BE 节点：

```sql
mysql> ALTER SYSTEM ADD BACKEND "host:port";
```

这里 host 为与 priority_networks 设置相匹配的 IP，port 为 BE 配置文件中的 heartbeat_service_port，默认为 9050。
<br/>

如出现错误，需要删除 BE 节点，应用下列命令：

* `alter system decommission backend "be_host:be_heartbeat_service_port";`

具体参考 [扩容缩容](../administration/Scale_up_down.md)。
<br/>

第三步: 启动 BE：

```shell
bin/start_be.sh --daemon
```

<br/>

第四步: 查看 BE 状态, 确认 BE 就绪:

```Plain Text
mysql> SHOW PROC '/backends'\G

********************* 1. row **********************
            BackendId: 10002
              Cluster: default_cluster
                   IP: 172.16.139.24
             HostName: starrocks-sandbox01
        HeartbeatPort: 9050
               BePort: 9060
             HttpPort: 8040
             BrpcPort: 8060
        LastStartTime: 2020-03-23 20:19:07
        LastHeartbeat: 2020-03-23 20:34:49
                Alive: true
 SystemDecommissioned: false
ClusterDecommissioned: false
            TabletNum: 0
     DataUsedCapacity: .000
        AvailCapacity: 327.292 GB
        TotalCapacity: 450.905 GB
              UsedPct: 27.41 %
       MaxDiskUsedPct: 27.41 %
               ErrMsg:
              Version:
1 row in set (0.01 sec)
```

<br/>

如果 isAlive 为 true，则说明 BE 正常接入集群。如果 BE 没有正常接入集群，请查看 log 目录下的 be.WARNING 日志文件确定原因。
<br/>

如果日志中出现类似以下的信息，说明 priority\_networks 的配置存在问题。

```Plain Text
W0708 17:16:27.308156 11473 heartbeat\_server.cpp:82\] backend ip saved in master does not equal to backend local ip127.0.0.1 vs. 172.16.179.26
```

<br/>

此时需要，先用以下命令 drop 掉原来加进去的 be，然后重新以正确的 IP 添加 BE。

```sql
mysql> ALTER SYSTEM DROPP BACKEND "172.16.139.24:9050";
```

<br/>

由于是初次启动，如果在操作过程中遇到任何意外问题，都可以删除并重新创建 storage 目录，再从头开始操作。

## 部署 Broker

配置文件为 apache\_hdfs\_broker/conf/apache\_hdfs\_broker.conf

> 注意：Broker 没有也不需要 priority\_networks 参数，Broker 的服务默认绑定在 0.0.0.0 上，只需要在 ADD BROKER 时，填写正确可访问的 Broker IP 即可。

如果有特殊的 hdfs 配置，复制线上的 hdfs-site.xml 到 conf 目录下

启动：

```shell
./apache_hdfs_broker/bin/start_broker.sh --daemon
```

添加 broker 节点到集群中：

```sql
MySQL> ALTER SYSTEM ADD BROKER broker1 "172.16.139.24:8000";
```

查看 broker 状态：

```plain text
MySQL> SHOW PROC "/brokers"\G
*************************** 1. row ***************************
          Name: broker1
            IP: 172.16.139.24
          Port: 8000
         Alive: true
 LastStartTime: 2020-04-01 19:08:35
LastUpdateTime: 2020-04-01 19:08:45
        ErrMsg: 
1 row in set (0.00 sec)
```

Alive 为 true 代表状态正常。

## 参数设置

* **Swappiness**

关闭交换区，消除交换内存到虚拟内存时对性能的扰动。

```shell
echo 0 | sudo tee /proc/sys/vm/swappiness
```

* **Compaction 相关**

当使用聚合表或更新模型，导入数据比较快的时候，可在配置文件 `be.conf` 中修改下列参数以加速 compaction。

```shell
cumulative_compaction_num_threads_per_disk = 4
base_compaction_num_threads_per_disk = 2
cumulative_compaction_check_interval_seconds = 2
```

* **并行度**

在客户端执行命令，修改 StarRocks 的并行度(类似 clickhouse set max_threads = 8)。并行度可以设置为当前机器 CPU 核数的一半。

```sql
set global parallel_fragment_exec_instance_num =  8;
```

## 使用 MySQL 客户端访问 StarRocks

### Root 用户登录

使用 MySQL 客户端连接某一个 FE 实例的 query_port(9030), StarRocks 内置 root 用户，密码默认为空：

```shell
mysql -h fe_host -P9030 -u root
```

<br/>

清理环境：

```sql
mysql > drop database if exists example_db;

mysql > drop user test;
```

### 创建新用户

通过下面的命令创建一个普通用户：

```sql
mysql > create user 'test' identified by '123456';
```

### 创建数据库

StarRocks 中 root 账户才有权建立数据库，使用 root 用户登录，建立 example\_db 数据库:

```sql
mysql > create database example_db;
```

  <br/>

数据库创建完成之后，可以通过 show databases 查看数据库信息：

```Plain Text
mysql > show databases;

+--------------------+
| Database           |
+--------------------+
| example_db         |
| information_schema |
+--------------------+
2 rows in set (0.00 sec)
```

information_schema 是为了兼容 mysql 协议而存在，实际中信息可能不是很准确，所以关于具体数据库的信息建议通过直接查询相应数据库而获得。

### 账户授权

example_db 创建完成之后，可以通过 root 账户 example_db 读写权限授权给 test 账户，授权之后采用 test 账户登录就可以操作 example\_db 数据库了：

```sql
mysql > grant all on example_db to test;
```

<br/>

退出 root 账户，使用 test 登录 StarRocks 集群：

```sql
mysql > exit

mysql -h 127.0.0.1 -P9030 -utest -p123456
```

### 建表

StarRocks 支持支持单分区和复合分区两种建表方式。

<br/>

在复合分区中：

* 第一级称为 Partition，即分区。用户可以指定某一维度列作为分区列（当前只支持整型和时间类型的列），并指定每个分区的取值范围。
* 第二级称为 Distribution，即分桶。用户可以指定某几个维度列（或不指定，即所有 KEY 列）以及桶数对数据进行 HASH 分布。

以下场景推荐使用复合分区：

* 有时间维度或类似带有有序值的维度：可以以这类维度列作为分区列。分区粒度可以根据导入频次、分区数据量等进行评估。
* 历史数据删除需求：如有删除历史数据的需求（比如仅保留最近 N 天的数据）。使用复合分区，可以通过删除历史分区来达到目的。也可以通过在指定分区内发送 DELETE 语句进行数据删除。
* 解决数据倾斜问题：每个分区可以单独指定分桶数量。如按天分区，当每天的数据量差异很大时，可以通过指定分区的分桶数，合理划分不同分区的数据, 分桶列建议选择区分度大的列。

用户也可以不使用复合分区，即使用单分区。则数据只做 HASH 分布。

<br/>  

下面分别演示两种分区的建表语句：

1. 首先切换数据库：mysql > use example_db;
2. 建立单分区表建立一个名字为 table1 的逻辑表。使用全 hash 分桶，分桶列为 siteid，桶数为 10。这个表的 schema 如下：

* siteid：类型是 INT（4 字节）, 默认值为 10
* city_code：类型是 SMALLINT（2 字节）
* username：类型是 VARCHAR, 最大长度为 32, 默认值为空字符串
* pv：类型是 BIGINT（8 字节）, 默认值是 0; 这是一个指标列, StarRocks 内部会对指标列做聚合操作, 这个列的聚合方法是求和（SUM）。这里采用了聚合模型，除此之外 StarRocks 还支持明细模型和更新模型，具体参考 [数据模型介绍](../table_design/Data_model.md)。

建表语句如下:

```sql
mysql >
CREATE TABLE table1
(
    siteid INT DEFAULT '10',
    citycode SMALLINT,
    username VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(siteid, citycode, username)
DISTRIBUTED BY HASH(siteid) BUCKETS 10
PROPERTIES("replication_num" = "1");
```

1. 建立复合分区表

建立一个名字为 table2 的逻辑表。这个表的 schema 如下：

* event_day：类型是 DATE，无默认值
* siteid：类型是 INT（4 字节）, 默认值为 10
* city_code：类型是 SMALLINT（2 字节）
* username：类型是 VARCHAR, 最大长度为 32, 默认值为空字符串
* pv：类型是 BIGINT（8 字节）, 默认值是 0; 这是一个指标列, StarRocks 内部会对指标列做聚合操作, 这个列的聚合方法是求和（SUM）

我们使用 event_day 列作为分区列，建立 3 个分区: p1, p2, p3

* p1：范围为 \[最小值, 2017-06-30)
* p2：范围为 \[2017-06-30, 2017-07-31)
* p3：范围为 \[2017-07-31, 2017-08-31)

每个分区使用 siteid 进行哈希分桶，桶数为 10。

<br/>  

建表语句如下:

```sql
CREATE TABLE table2
(
event_day DATE,
siteid INT DEFAULT '10',
citycode SMALLINT,
username VARCHAR(32) DEFAULT '',
pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, siteid, citycode, username)
PARTITION BY RANGE(event_day)
(
PARTITION p1 VALUES LESS THAN ('2017-06-30'),
PARTITION p2 VALUES LESS THAN ('2017-07-31'),
PARTITION p3 VALUES LESS THAN ('2017-08-31')
)
DISTRIBUTED BY HASH(siteid) BUCKETS 10
PROPERTIES("replication_num" = "1");
```

  <br/>

表建完之后，可以查看 example\_db 中表的信息:

```Plain Text
mysql> show tables;

+-------------------------+
| Tables_in_example_db    |
+-------------------------+
| table1                  |
| table2                  |
+-------------------------+
2 rows in set (0.01 sec)

  <br/>

mysql> desc table1;

+----------+-------------+------+-------+---------+-------+
| Field    | Type        | Null | Key   | Default | Extra |
+----------+-------------+------+-------+---------+-------+
| siteid   | int(11)     | Yes  | true  | 10      |       |
| citycode | smallint(6) | Yes  | true  | N/A     |       |
| username | varchar(32) | Yes  | true  |         |       |
| pv       | bigint(20)  | Yes  | false | 0       | SUM   |
+----------+-------------+------+-------+---------+-------+
4 rows in set (0.00 sec)

  <br/>

mysql> desc table2;

+-----------+-------------+------+-------+---------+-------+
| Field     | Type        | Null | Key   | Default | Extra |
+-----------+-------------+------+-------+---------+-------+
| event_day | date        | Yes  | true  | N/A     |       |
| siteid    | int(11)     | Yes  | true  | 10      |       |
| citycode  | smallint(6) | Yes  | true  | N/A     |       |
| username  | varchar(32) | Yes  | true  |         |       |
| pv        | bigint(20)  | Yes  | false | 0       | SUM   |
+-----------+-------------+------+-------+---------+-------+
5 rows in set (0.00 sec)
```

## 使用 Docker 进行编译

具体参考 [在容器内编译](../administration/Build_in_docker.md)。
