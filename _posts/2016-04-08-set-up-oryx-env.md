---
layout: post
title: 玩转 Oryx2 （一）
---

<!--  -->
<!--  -->
<!--  _ __    __     __      ____    ___     ___   _____   __  __    ___     -->
<!-- /\`'__\/'__`\ /'__`\   /',__\  / __`\ /' _ `\/\ '__`\/\ \/\ \ /' _ `\   -->
<!-- \ \ \//\  __//\ \L\.\_/\__, `\/\ \L\ \/\ \/\ \ \ \L\ \ \ \_\ \/\ \/\ \  -->
<!--  \ \_\\ \____\ \__/.\_\/\____/\ \____/\ \_\ \_\ \ ,__/\ \____/\ \_\ \_\ -->
<!--   \/_/ \/____/\/__/\/_/\/___/  \/___/  \/_/\/_/\ \ \/  \/___/  \/_/\/_/ -->
<!--                                                 \ \_\                   -->
<!--                                                  \/_/                   -->
<!--  -->

## 准备

### 环境
  * CDH 5.5.2, Parcel
    * HDFS
    * YARN
    * Zookeeper
    * Kafka
    * Spark (on YARN)

### 下载最新版本

  * 进入[下载页面](https://github.com/OryxProject/oryx/releases/tag/oryx-2.1.2)，分别下载

    * compute-classpath.sh
    * oryx-batch-2.1.2.jar
    * oryx-run.sh
    * oryx-serving-2.1.2.jar
    * oryx-speed-2.1.2.jar

  * 下载conf文件

    * 以ALS为例，在源码目录oryx/app/conf下[als-example.conf](https://github.com/OryxProject/oryx/blob/master/app%2Fconf%2Fals-example.conf)

    * 并将其重命名为oryx.conf（注：文件需要和每个层的JAR文件放在同一个目录下）

    * 修改conf文件

```
  # 现阶段只需要修改这么几个字段就OK了
  kafka-brokers = "data-mining-46.slave:9092,data-mining-47.slave:9092,data-mining-48.slave:9092,data-mining-49.master:9092"
  zk-servers = "data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka"
  hdfs-base = "hdfs:///Oryx"

  oryx {
  id = "ALSExample"
  als {
    rescorer-provider-class = null
  }
  input-topic {
    broker = ${kafka-brokers}
    lock = {
      master = ${zk-servers}
    }
  ...
```

### 开始

```
[root@data-mining-49 oryx]# ./oryx-run.sh kafka-setup
Input   ZK      data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
        Kafka   data-mining-46.slave:9092,data-mining-47.slave:9092,data-mining-48.slave:9092,data-mining-49.master:9092
        topic   OryxInput
Update  ZK      data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
        Kafka   data-mining-46.slave:9092,data-mining-47.slave:9092,data-mining-48.slave:9092,data-mining-49.master:9092
        topic   OryxUpdate

All available topics:


Input topic OryxInput does not exist. Create it? y
Creating topic OryxInput
Error while executing topic command : org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /brokers/ids
[2016-04-08 16:53:09,383] ERROR org.I0Itec.zkclient.exception.ZkNoNodeException: org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode =
NoNode for /brokers/ids
        at org.I0Itec.zkclient.exception.ZkException.create(ZkException.java:47)
        at org.I0Itec.zkclient.ZkClient.retryUntilConnected(ZkClient.java:995)
        at org.I0Itec.zkclient.ZkClient.getChildren(ZkClient.java:675)
        at org.I0Itec.zkclient.ZkClient.getChildren(ZkClient.java:671)
        at kafka.utils.ZkUtils.getChildren(ZkUtils.scala:537)
        at kafka.utils.ZkUtils.getSortedBrokerList(ZkUtils.scala:172)
        at kafka.admin.AdminUtils$.createTopic(AdminUtils.scala:243)
        at kafka.admin.TopicCommand$.createTopic(TopicCommand.scala:107)
        at kafka.admin.TopicCommand$.main(TopicCommand.scala:60)
        at kafka.admin.TopicCommand.main(TopicCommand.scala)
Caused by: org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /brokers/ids
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:111)
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:51)
        at org.apache.zookeeper.ZooKeeper.getChildren(ZooKeeper.java:1468)
        at org.apache.zookeeper.ZooKeeper.getChildren(ZooKeeper.java:1496)
        at org.I0Itec.zkclient.ZkConnection.getChildren(ZkConnection.java:114)
        at org.I0Itec.zkclient.ZkClient$4.call(ZkClient.java:678)
        at org.I0Itec.zkclient.ZkClient$4.call(ZkClient.java:675)
        at org.I0Itec.zkclient.ZkClient.retryUntilConnected(ZkClient.java:985)
        ... 8 more
 (kafka.admin.TopicCommand$)
Status of topic OryxInput:
y
Update topic OryxUpdate does not exist. Create it?
Creating topic OryxUpdate
Error while executing topic command : org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /brokers/ids
[2016-04-08 16:53:11,094] ERROR org.I0Itec.zkclient.exception.ZkNoNodeException: org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /brokers/ids
        at org.I0Itec.zkclient.exception.ZkException.create(ZkException.java:47)
        at org.I0Itec.zkclient.ZkClient.retryUntilConnected(ZkClient.java:995)
        at org.I0Itec.zkclient.ZkClient.getChildren(ZkClient.java:675)
        at org.I0Itec.zkclient.ZkClient.getChildren(ZkClient.java:671)
        at kafka.utils.ZkUtils.getChildren(ZkUtils.scala:537)
        at kafka.utils.ZkUtils.getSortedBrokerList(ZkUtils.scala:172)
        at kafka.admin.AdminUtils$.createTopic(AdminUtils.scala:243)
        at kafka.admin.TopicCommand$.createTopic(TopicCommand.scala:107)
        at kafka.admin.TopicCommand$.main(TopicCommand.scala:60)
        at kafka.admin.TopicCommand.main(TopicCommand.scala)
Caused by: org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /brokers/ids
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:111)
        at org.apache.zookeeper.KeeperException.create(KeeperException.java:51)
        at org.apache.zookeeper.ZooKeeper.getChildren(ZooKeeper.java:1468)
        at org.apache.zookeeper.ZooKeeper.getChildren(ZooKeeper.java:1496)
        at org.I0Itec.zkclient.ZkConnection.getChildren(ZkConnection.java:114)
        at org.I0Itec.zkclient.ZkClient$4.call(ZkClient.java:678)
        at org.I0Itec.zkclient.ZkClient$4.call(ZkClient.java:675)
        at org.I0Itec.zkclient.ZkClient.retryUntilConnected(ZkClient.java:985)
        ... 8 more
 (kafka.admin.TopicCommand$)
Error while executing topic command : Topic OryxUpdate does not exist on ZK path data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
[2016-04-08 16:53:11,833] ERROR java.lang.IllegalArgumentException: Topic OryxUpdate does not exist on ZK path data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
        at kafka.admin.TopicCommand$.alterTopic(TopicCommand.scala:119)
        at kafka.admin.TopicCommand$.main(TopicCommand.scala:62)
        at kafka.admin.TopicCommand.main(TopicCommand.scala)
 (kafka.admin.TopicCommand$)
Status of topic OryxUpdate:

[root@data-mining-49 oryx]#
```

### 此时出现了一个问题：

```
Error while executing topic command : org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /brokers/ids
```

这个问题应该是：CDH没有启动Kafka服务没有启动。通过CDH后台启动Kafka服务（如果没有添加Kafka服务，则需要先添加）。

![Kafka Service]({{ site.url }}/assets/oryx/add-kafka-service.png)

此时重新执行setup命令：

```
[root@data-mining-49 oryx]# ./oryx-run.sh kafka-setup
Input   ZK      data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
        Kafka   data-mining-47.slave:9092,data-mining-48.slave:9092
        topic   OryxInput
Update  ZK      data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
        Kafka   data-mining-47.slave:9092,data-mining-48.slave:9092
        topic   OryxUpdate

All available topics:


Input topic OryxInput does not exist. Create it? y
Creating topic OryxInput
Created topic "OryxInput".
Status of topic OryxInput:
Topic:OryxInput PartitionCount:4        ReplicationFactor:1     Configs:
        Topic: OryxInput        Partition: 0    Leader: 140     Replicas: 140   Isr: 140
        Topic: OryxInput        Partition: 1    Leader: 141     Replicas: 141   Isr: 141
        Topic: OryxInput        Partition: 2    Leader: 140     Replicas: 140   Isr: 140
        Topic: OryxInput        Partition: 3    Leader: 141     Replicas: 141   Isr: 141

Update topic OryxUpdate does not exist. Create it? y
Creating topic OryxUpdate
Created topic "OryxUpdate".
Updated config for topic "OryxUpdate".
Status of topic OryxUpdate:
Topic:OryxUpdate        PartitionCount:1        ReplicationFactor:1     Configs:retention.ms=86400000,max.message.bytes=16777216
        Topic: OryxUpdate       Partition: 0    Leader: 140     Replicas: 140   Isr: 140

[root@data-mining-49 oryx]#
```

OK！

### 需要将如上的二进制脚本都同步到其他集群的节点上了

```
[root@data-mining-49 data]# scp -r oryx/ root@192.168.1.48:/data/
compute-classpath.sh    100% 1893     1.9KB/s   00:00
oryx-run.sh             100%   13KB  13.2KB/s   00:00
oryx-batch-2.1.2.jar    100%   26MB  26.0MB/s   00:00
oryx-serving-2.1.2.jar  100%   33MB  33.0MB/s   00:01
oryx-speed-2.1.2.jar    100%   26MB  26.0MB/s   00:00
oryx.conf               100%   1884  1.8KB/s    00:00
```

### 接下来我们需要找一个可用的数据集：

[MovieLens 100K Dataset](http://files.grouplens.org/datasets/movielens/ml-100k.zip)，

将这个数据集的格式修改下：

```
[root@data-mining-49 ml-100k]# tr '\t' ',' < u.data > data.csv
[root@data-mining-49 ml-100k]# tail data.csv
806,421,4,882388897
676,538,4,892685437
721,262,3,877137285
913,209,2,881367150
378,78,3,880056976
880,476,3,880175444
716,204,5,879795543
276,1090,1,874795795
13,225,2,882399156
12,203,3,879959583
[root@data-mining-49 ml-100k]#
```

数据准备完毕，我们可以通过

```
[root@data-mining-49 oryx]# ./oryx-run.sh kafka-input --input-file data.csv
```
命令导入系统， 同时打开另一个终端窗口，通过命令

```
[root@data-mining-49 oryx]# ./oryx-run.sh kafka-tail
Input   ZK      data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
        Kafka   data-mining-47.slave:9092,data-mining-48.slave:9092
        topic   OryxInput
Update  ZK      data-mining-46.slave:2181,data-mining-47.slave:2181,data-mining-48.slave:2181/kafka
        Kafka   data-mining-47.slave:9092,data-mining-48.slave:9092
        topic   OryxUpdate

279,64,1,875308510
646,750,3,888528902
654,370,2,887863914
617,582,4,883789294
913,690,3,880824288
660,229,2,891406212
421,498,4,892241344
495,1091,4,888637503
806,421,4,882388897
676,538,4,892685437
721,262,3,877137285
913,209,2,881367150
378,78,3,880056976
880,476,3,880175444
716,204,5,879795543
276,1090,1,874795795
13,225,2,882399156
12,203,3,879959583
```

实时跟踪输入导入情况，当数据全部导入完毕后，用户可以手动的通过Ctrl-C关闭这个命令。

如果以上全部成功了，可以关闭这些进程。集群至此已经准备好运行Oryx了。

### 启动服务

```
./oryx-run.sh batch
./oryx-run.sh speed
./oryx-run.sh serving

```

blablabla.....好多输出，先不要管他。

确认服务层启动成功以后，可以导入数据（难道刚才导数据是测试用的？）

```
wget --quiet --post-file data.csv --output-document -  --header "Content-Type: text/csv" http://192.168.1.49:8080/ingest
```
批处理层已经开始触发一个新的计算了。

此时看看HDFS中的数据是这样的：

```
[root@data-mining-49 oryx]# hdfs dfs -ls /Oryx
Found 2 items
drwxr-xr-x   - hdfs supergroup          0 2016-04-08 18:40 /Oryx/data
drwxr-xr-x   - hdfs supergroup          0 2016-04-08 18:40 /Oryx/model
^[[A[root@data-mining-49 oryx]# hdfs dfs -ls /Oryx/data
Found 1 items
drwxr-xr-x   - hdfs supergroup          0 2016-04-08 18:40 /Oryx/data/oryx-1460112000000.data
[root@data-mining-49 oryx]# hdfs dfs -ls /Oryx/model
Found 3 items
drwxr-xr-x   - hdfs supergroup          0 2016-04-08 18:41 /Oryx/model/.checkpoint
drwxr-xr-x   - hdfs supergroup          0 2016-04-08 18:40 /Oryx/model/.temporary
drwxr-xr-x   - hdfs supergroup          0 2016-04-08 18:40 /Oryx/model/1460112018440
[root@data-mining-49 oryx]#
```

表明数据已经ok了，通过API确认计算是否完成

```
[root@data-mining-49 ~]# wget --quiet --output-document - --server-response http://192.168.1.49:8080/ready
  HTTP/1.1 200 OK
  Content-Length: 0
  Date: Fri, 08 Apr 2016 10:43:27 GMT
  Server: Oryx
```

OK！

### 我们来查看下推荐结果

```
[root@data-mining-49 ~]# wget --quiet --output-document -  http://192.168.1.49:8080/recommend/17
50,0.8234792673029006
127,0.7889597890898585
181,0.7442612769082189
275,0.7263787714764476
258,0.7164891492575407
121,0.7006391943432391
15,0.697495789732784
288,0.6787936894688755
285,0.6696474105119705
25,0.6603603088587988
[root@data-mining-49 ~]# wget --quiet --output-document -  http://192.168.1.49:8080/recommend/806
173,1.084704713546671
64,1.0142679414711893
7,0.9939510896801949
183,0.9918235178338364
22,0.9644620651379228
69,0.945814429782331
202,0.9420167971402407
11,0.9270770070143044
135,0.9227914617804345
191,0.9215727103874087
[root@data-mining-49 ~]#
```

完美结束。

这貌似是走的批处理逻辑，中途实时计算层挂鸟

```
16/04/08 18:42:38 INFO SparkContext: Successfully stopped SparkContext
16/04/08 18:42:38 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
Exception in thread "main" org.apache.spark.SparkException: Job aborted due to stage failure: Task 0 in stage 0.0 failed 4 times, most recent failure: Lost task 0.3 in stage 0.0 (TID 3, data-mining-47.slave): java.io.IOException: org.apache.spark.SparkException: Failed to get broadcast_0_piece0 of broadcast_0
        at org.apache.spark.util.Utils$.tryOrIOException(Utils.scala:1177)
        at org.apache.spark.broadcast.TorrentBroadcast.readBroadcastBlock(TorrentBroadcast.scala:165)
        at org.apache.spark.broadcast.TorrentBroadcast._value$lzycompute(TorrentBroadcast.scala:64)
        at org.apache.spark.broadcast.TorrentBroadcast._value(TorrentBroadcast.scala:64)
        at org.apache.spark.broadcast.TorrentBroadcast.getValue(TorrentBroadcast.scala:88)
        at org.apache.spark.broadcast.Broadcast.value(Broadcast.scala:70)
        at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:62)
        at org.apache.spark.scheduler.Task.run(Task.scala:88)
        at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:214)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
Caused by: org.apache.spark.SparkException: Failed to get broadcast_0_piece0 of broadcast_0
        at org.apache.spark.broadcast.TorrentBroadcast$$anonfun$org$apache$spark$broadcast$TorrentBroadcast$$readBlocks$1$$anonfun$2.apply(TorrentBroadcast.scala:138)
        at org.apache.spark.broadcast.TorrentBroadcast$$anonfun$org$apache$spark$broadcast$TorrentBroadcast$$readBlocks$1$$anonfun$2.apply(TorrentBroadcast.scala:138)
        at scala.Option.getOrElse(Option.scala:120)

```

咱们下一篇再找找原因吧。

## 参考文献
  * http://oryx.io/docs/admin.html
  * http://oryx.io/docs/endusers.html
  * http://oryx.io/docs/developer.html
  * http://oryx.io/docs/performance.html
  * http://oryx.io/apidocs/index.html
