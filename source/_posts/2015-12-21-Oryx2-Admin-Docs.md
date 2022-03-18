---
layout: post
title: Oryx2 管理员文档
---


### Oryx 2.1.0 系统要求

* Java 7 or later (JRE only is required)
* A Hadoop cluster running the following components:
  * Apache Hadoop 2.6.0 or later
  * Apache Zookeeper 3.4.5 or later
  * Apache Kafka 0.8.2 or later (in 0.8.x line)
  * Apache Spark 1.5.0 or later

<!--more-->

### 服务

Hadoop cluster 服务
  * HDFS
  * YARN
  * Zookeeper
  * Kafka
  * Spark (on YARN)

Note that for CDH, Kafka is available as a parcel from [Cloudera Labs](http://www.cloudera.com/content/cloudera/en/developers/home/cloudera-labs/apache-kafka.html).

Kafka brokers 需要配置在集群中，根据实例，需要注意hosts和端口。端口一般会设置为9092，此处和ZK server的端口保持一致，缺省设置为2181。缺省端口会在随后的例子中使用。

多个hosts需要通过逗号分割，并需要提供host:port 这种方式，比如 ： your-zk-1:2181,your-zk-2:2181。

同时需要注意，你的ZK实例是否使用了chroot path。这是一个简单的路径前缀，比如 your-zk:2181/your-chroot
/kafka 是经常用到作为前缀的。如果没有使用chroot，就可以忽略这个。注意：如果存在多个ZK server，和一个chroot，只需要在最后添加一次chroot即可，比如 your-zk-1:2181,your-zk-2:2181/kafka

### 配置Kafka

Oryx 使用2个Kafka topics 做数据传输。
  * 一个传输输入数据到批量处理，和实时计算层
  * 另一个同步模型更新到服务层

缺省的topics的名称分别为OryxInput 和 OryxUpdate，只有Oryx服务启动后才能创建这两个topics。

输入topic的分区数目会影响消费数据的Spark Streaming 作业的分区数，甚至是并发数。比如，批量处理层读取HDFS上的历史数据分区和Kafka数据。

如果输入topic只有一个分区，且在每个时间间隔内有大量的数据涌入，这时Kafka基于的输入分区相对的需要很长的时间去处理。一个比较合适的经验值是选择一些topic分区，在一个批处理时间间隔内到达的大量数据，大约是一个HDFS块大小，缺省值是128MB。

提供的oryx-run.sh kafka设置脚本，缺省设置为4个分区，当然了，这个值之后是可以修改的。必须注意不能设置更新topic多于1个分区。

重复因子可以设置为任何值，但是建议最少是2。注意：重复因子数目不能超过Kafka brokers在集群上的数目。所以提供的设置脚本里缺省设置重复因子为1. 之后你可以通过kafka-topics --zookeeper ... --alter --topic ... --replication-factor N 等修改这些缺省值。

你需要为其中一个或者两个topic配置持续时间。尤其重要的是需要限制更新topic的持续时间，因为实时计算层和服务层需要从启动开始的起点获取整个topic。这个机制并不如输入数据重要，输入数据不会再次从头读取数据。

设置这个值为批量处理层更新间隔的2倍是比较合适。比如设置该值为1天（24 * 60 * 60 * 1000 = 86400000 ms），设置topic的为86400000ms。这个可以通过oryx-run.sh设置脚本自动设置。

上述两个topics会包含大量信息；尤其是更新topic包含整个序列化的PMML模型。很有可能他会超过Kafka缺省最大消息的大小（kafka消息最大1Mib）。如果有更大的数据则需要设置topic’s max.message.bytes。oryx-run.sh Kafka设置脚本设置更新topic缺省为16Mib。这也是Oryx试图吸入更新topic里的模型的最大值默认；更大的模型只会以文件的形式保存到HDFS中的路径中。请查看属性oryx.update-topic.message.max-size。

Kafka代理的message.max.bytes属性可以控制这个，但是设置这个值会影响到代理管理的所有的topics，甚至包括不良状态的topics。可以通过查看性能和资源部分了解更多。尤其是需要注意的，必须设置代理的replica.fetch.max.bytes属性，以防止重复任何非常大的消息。

> There is no per-topic equivalent to this.

### Kafka配置自动设置

提供的oryx-run.sh脚本可以打印ZK的当前配置，列出已经存在的Kafka中的topics，如果需要，会创建配置好的输入topics和更新topics。

你需要先创建Oryx配置文件，或者可以拷贝conf/als-example.conf。需要按照要求修改Kafka和ZK的配置文件，比如topic名称。

oryx.conf文件需要和每个层的JAR文件放在同一个目录下，然后执行：

```
./oryx-run.sh kafka-setup

Input  ZK:    your-zk:2181
Input  Kafka: your-kafka:9092
Input  topic: OryxInput
Update ZK:    your-zk:2181
Update Kafka: your-kafka:9092
Update topic: OryxUpdate

All available topics:


Input topic OryxInput does not exist. Create it? y
Creating topic OryxInput
Created topic "OryxInput".
Status of topic OryxInput:
Topic:OryxInput	PartitionCount:4	ReplicationFactor:1	Configs:
	Topic: OryxInput	Partition: 0	Leader: 120	Replicas: 120,121	Isr: 120,121
	Topic: OryxInput	Partition: 1	Leader: 121	Replicas: 121,120	Isr: 121,120
	Topic: OryxInput	Partition: 2	Leader: 120	Replicas: 120,121	Isr: 120,121
	Topic: OryxInput	Partition: 3	Leader: 121	Replicas: 121,120	Isr: 121,120

Update topic OryxUpdate does not exist. Create it? y
Creating topic OryxUpdate
Created topic "OryxUpdate".
Updated config for topic "OryxUpdate".
Status of topic OryxUpdate:
Topic:OryxUpdate	PartitionCount:1	ReplicationFactor:1	Configs:retention.ms=86400000,max.message.bytes=16777216
	Topic: OryxUpdate	Partition: 0	Leader: 120	Replicas: 120,121	Isr: 120,121
```
查看发送到输入和更新topic，监控应用的动作，可以执行：

```
./oryx-run.sh kafka-tail
Input  ZK:    your-zk:2181
Input  Kafka: your-kafka:9092
Input  topic: OryxInput
Update ZK:    your-zk:2181
Update Kafka: your-kafka:9092
Update topic: OryxUpdate

...output...
```

接着在另外一个窗口，可以接受输入数据，比如将来自终端用户的文档data.csv加入到输入队列，并验证：

```
./oryx-run.sh kafka-input --input-file data.csv
```

如果以上全部成功了，可以关闭这些进程。集群至此已经准备好运行Oryx了。

### HDFS和数据层

在Oryx中，Kafka是数据传输的途径，因此数据在Kafka中只是需要暂时存放的。然而输入数据也会持久化到HDFS中以备之后使用。同样的，模型和更新用来为Kafka的更新topic提供数据，模型也会持久化到HDFS以备之后引用。

oryx.batch.storage.data-dir定义了输入数据存放在HDFS中的位置。在这个目录下，子目录标题会以oryx-[timestamp].data形式创建，每一个会通过Spark Straming在批量处理层执行。在这里，时间戳格式格式与Unix相同，且以毫秒为单位。

实际上，和大多数Hadoop中分布式进程输出的『文件』一样，存在这样的一个子目录，包含了很多以part-开头的文件。每个文件都是序列化文件，通过Writable类序列化Kafka输入topics的键值，Writable类实现自oryx.batch.storage.key-writable-class 和 oryx.batch.storage.message-writable-class类。默认情况下，这是TextWritable，并且keys和消息是以字符串形式被记录下来的。

该目录下的数据会被删除。也就不会再次被批处理层计算使用。尤其是，设置oryx.batch.storage.max-age-data-hours为一个非负数，将会使批处理层自动删除大于给定时间的数据。

同样的，在每个批处理间隔内被批处理层选中的模型会被原始的机器学习应用（扩展子MLUpdate）输出。也会被持久化到oryx.batch.storage.model-dir定义的目录下的子目录中。在这个目录下，子目录的命名都是以时间戳形式实现的，同Unix毫秒形式。

子目录下的内容取决于应用，但是一般会包含以model.pmml命名的PMML模块，并且和模块一起存在的可选的追加文件。

这个目录之所以存在是因为需要记录PMML模块用来归档用或者被其他工具使用。也可按照规则删除其内容。

### 捕获错误

最后，你可能希望停止其中一个或者几个层的运行，或者重启。服务也可能挂鸟。这到底发生了神马？为啥会这样捏？

#### 数据丢失

历史数据存放在HDFS中，理论上会存放多个副本。HDFS会确保数据被靠谱的存放着。当设置了副本，Kafka也被设计为采用副本方式应对故障。

这并没有啥鸟用，这样不能确保数据不会丢失，只能依靠HDFS和Kafka能正常可用罢了。

#### 服务器挂鸟

通常情况下，所有的三层服务进程应该会持续的工作，如果不得不停止或者挂鸟的话，服务自己会立即重启。这个可以通过初始化脚本完美的完成或者类似机制（尚未实现鸟）

##### 服务层

服务层是无状态的。启动后，他会读取更新topics中的所有的模型和可用更新。当首个可用的模型就绪后，就可以开始应答请求。基于这个原因，需要适当的限制更新topic的持续时间。

服务层的操作不是分布式的，每个实例是独立的，启动和停止不会影响到其他部分。

##### 实时计算层

实时计算层同样也不存在状态，也会在读取全部的模型和更新topic的更新。只要存在合法的模型，就可以生成更新。同时，从最后一次偏移位置开始读取输入topics。

实时计算层使用Spark和Spark Streaming 做计算。Spark会响应计算过程中失败情况，并重试任务。

Spark Streaming的Kafka集成模块在某些情况下可以恢复接收的故障。
如果是整个进程死掉并被重新启动，oryx.id的值被设定以后，系统会自动从上一次Kafka记录的偏移地址开始读取。（否则，将会从上次偏移地址开始，这就意味着实时计算层没有运行的时候，到达的数据就不会生成任何更新。），同样的，如果实时计算层的模型还没有准备好的话，收到的数据也会被忽略。It effectively adopts “at most once” semantics.

由于实时计算层的作用是为最后发布的模型提供approximate, “best effort”的更新。这种行为由于其间接性一般是没有问题，且令人满意的。

##### 批处理层
批处理层是最复杂的，因为他并生成某些状态：

  * 历史数据，总是持久化到HDFS
  * 如果应用选择的话，模型的扩展状态和topics都可以被持久化到HDFS上

对于多次或者根本不读取数据是非常敏感的，因为他本来就是生产官方下一代模型的组件

与实时计算层一起，Spark和Spark Streaming在计算过程中可以捕获很多错误情况。也可以管理存储到HDFS中的数据，负责避免两次写入相同数据。

应用负责回复各自的「状态」，一般情况下，建立在Oryx ML层的应用会将状态写入唯一的子目录中，并且重启后会在新的目录简单的产生一个新状态。前一个状态如果存在的话，也会被完整写入或者被完全忽视。

批处理层也和实时计算层一样，符合『至多一次』的规则。综上，如果整个进程死掉或者被重启，oryx.id被设置的话，则会从Kafka记录的最后一次偏移重新读取，否则会在最后一次偏移处重新读取数据。
