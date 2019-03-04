---
layout: post
title: Apache Kafka Structure
---


#### Apache Kafka消息服务

  * 参考地址 [http://kafka.apache.org/documentation.html](http://kafka.apache.org/documentation.html#brokerconfigs)

  * 消息队列的分类

    * 点对点

      生产者生产消息发送到Queue中，消费者消费Queue中的消息，其中：
        * Queue中不再存储已经被消费的消息
        * Queue支持多个消费者，但是同一个消息，只能被一个消费者消费

    * 发布/订阅

      生产者（生产）将消息发布到topic中，同时多个消费者（消费）订阅该消息。和点对点方式不同的是，发布到topic的消息会被所有订阅者消费

  * 简介

    背景 Kafka使用Scala语言编写，是一个分布式，分区的，支持多副本，多订阅者的日志系统。

    目前支持Java，Python，C++， PHP等

    * 总体结构

![kafka总体结构图]({{ site.url }}/assets/kafka.0.9.0/structure.png)


  * 名词解释

    * Producer

        消息生产者，就是向kafka broker发消息的客户端

    * Consumer

        消息消费者，向kafka broker取消息的客户端

    * Topic

        是一个消息队列？

    * Consumer Group （CG）

      * 这是Kafka用来实现一个Topic消息的广播（发给所有的Consumer）和单播（发给任意一个Consumer）的手段
      * 一个Topic可以有多个CG
      * Topic的消息会复制（不是真的复制，是概念上的）到所有的CG，但每个CG只会把消息发给该CG中的一个consumer
      * 如果需要实现广播，只要每个Consumer有一个独立的CG就可以了
      * 要实现单播只要所有的Consumer在同一个CG
      * 用CG还可以将Consumer进行自由的分组而不需要多次发送消息到不同的topic

      * Broker

        * 一台Kafka服务器就是一个Broker
        * 一个集群由多个Broker组成。一个Broker可以容纳多个Topic

      * Partition

        为了实现扩展性，一个非常大的Topic可以分布到多个Broker（即服务器）上，一个Topic可以分为多个Partition，每个Partition是一个有序的队列。Prtition中的每条消息都会被分配一个有序的id（Offset）。Kafka只保证按一个Partition中的顺序将消息发给Consumer，不保证一个Topic的整体（多个Partition间）的顺序。

      * Offset

        * kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找
        * 例如你想找位于2049的位置，只要找到2048.kafka的文件即可，当然the first offset就是00000000000.kafka

  * 特性

    * 通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能
    * 高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数十万的消息
    * 支持 _同步_ 和 _异步_ 复制两种HA
    * Consumer客户端
      * pull
      * 随机读
      * 利用sendfile系统调用
      * zero-copy
      * 批量拉数据
    * 消费状态保存在客户端
    * 消息存储顺序写
    * 数据迁移、扩容对用户透明
    * 支持Hadoop并行数据加载
    * 支持online和offline的场景
    * 持久化：通过将数据持久化到硬盘以及replication防止数据丢失
    * scale out：无需停机即可扩展机器
    * 定期删除机制，支持设定partitions的segment file保留时间

  * 可靠性（一致性)

    传统的MQ系统通常都是通过broker和consumer间的确认（ack）机制实现的，并在broker保存消息分发的状态，即使这样一致性也是很难保证的。

    Kafka的做法是由consumer自己保存状态，也不要任何确认。这样虽然consumer负担更重，但其实更灵活了。因为不管consumer上任何原因导致需要重新处理消息，都可以再次从broker获得。

  * 可扩展性

    Kafka 使用Zookeeper实现动态的集群扩展，不需要更改客户端（生产者和消费者）的配置。broker会在ZK注册并保持相关的元数据更新。而客户端会在ZK上注册相关的watcher，一旦ZK发生变化，客户端能及时做出相应调整。这样可以保证变更broker时，各个broker之间能自动实现负载均衡。

  * 设计目标

    高吞吐量

      * 数据磁盘持久化：消息不在内存中cache，直接写入到磁盘，充分利用磁盘的顺序读写性能
      * zero-copy：减少IO操作步骤
      * 支持数据批量发送和拉取
      * 支持数据压缩
      * Topic划分为多个partition，提高并行处理能力

  * Producer负载均衡和HA机制
    * producer根据用户指定的算法，将消息发送到指定的partition。
    * 存在多个partiiton，每个partition有自己的replica，每个replica分布在不同的Broker节点上。
    * 多个partition需要选取出lead partition，lead partition负责读写，并由zookeeper负责fail over。
    * 通过zookeeper管理broker与consumer的动态加入与离开。

  * Consumer的pull机制

    由于broker会持久化数据，broker没有cache压力，因此，consumer比较适合才去pull的方式消费数据：
      * 简化kafka设计，降低了难度
      * Consumer根据消费能力自主控制消息拉取速度
      * Consumer根据自身情况自主选择消费模式，例如批量，重复消费，从制定partition或位置(offset)开始消费等

  * Consumer与Topic关系以及机制

    每个group包含多个consumer。对于topic中的一条特定消息，只会被订阅此Topic每个group中的一个consumer消费，那么一个group中的所有consumer将会交错的消费整个Topic。

    如果所有的consumer都具有相同的group（类似JMS queue），消息将有所有的consumer负载均衡

    如果所有的consumer都具有不同的group，那么这就是『发布-订阅』，消息将会广播给所有消费者

    在Kafka中，一个partition中的消息只会被group中的一个consumer消费（同一时刻）；每个group中consumer消息消费互相独立；
    一个group是一个『订阅』者，一个Topic中的每个partition只会被一个『订阅』者中的一个consumer消费，但是一个consumer可以同事消费多个partitions中的消息。

    Kafka只能保证一个partition中的消息被某个consumer消费是顺序的，但是从Topic角度，当有多个partitions时，消息仍不是全局有序的

    一个group中包含多个consumer，这样的话不仅能提高topic中消息的并发消费能力，还能提高『故障容错』性，如果group中的某个consumer失效，那么其消费的partition将会被其他consumer接管

    Kafka的设计原理决定，对于一个Topic，同一个group中不能有多于partition个数的consumer同时消费，否则将意味着某些consumer将无法得到消息

  * Producer均衡算法

    Kafka集群中的任何一个broker，都可以向producer提供metadata，这些metadata中包含『集群中存货的servers/partition leaders』，当producer获取到metadata后，会和topic下所有的partition leader保持socker连接；消息由producer直接通过socker发送到broker

    > 中间不会经过任何『路由层』，即，消息被路由到哪个partition上，是有producer决定的
    > 在producer端的配置文件中，可以指定partition的路由方式：『random』，『key-hash』等

  * Consumer均衡算法

    当一个group中，有consumer加入或者离开时，会触发partitions均衡。均衡的最终目的，是提升topic的并发消费能力。
      * 假如topic1,具有如下partitions: P0,P1,P2,P3
      * 加入group中,有如下consumer: C0,C1
      * 首先根据partition索引号对partitions排序: P0,P1,P2,P3
      * 根据consumer.id排序: C0,C1
      * 计算倍数: M = [P0,P1,P2,P3].size / [C0,C1].size,本例值M=2(向上取整)
      * 然后依次分配partitions: C0 = [P0,P1],C1=[P2,P3],即Ci = [P(i * M),P((i + 1) * M -1)]

  * Broker集群内broker之间replica机制

    replication策略是基于partiton，而不是topic

    > kafka将每个partition复制到多个server上
    > 任何一个partition有一个leader和任意数量的follower
    > 备份的数量可以由broker配置文件设定
    > leader处理所有的read-write请求，负责跟踪所有的follower状态，
    > 如果follower『落后』太多或者失效，leader会把它从replicas同步列表中删除
    > follower需要和leader保持同步，follower就像一个consumer，消费信息并保存在本地日志中
    > 当所有的follower都将一个消息保存成功，此消息才能被认为是『committed』，
    > 此时consumer才能消费它，这种策略要求leader和follower之间保持良好的网络环境
    > 只要ZK集群存活，即使只存活一个replica，仍可以保证消息的正常发送和接收

    * Kafka判定一个follower存活的条件
      * 和ZK保持良好的链接
      * 及时跟进leader，不能落后太多

    > 如果此replicas落后太多，它会继续在leader中fetch数据，然后加入同步列表中，
    > Kafka不会更换宿主，只有这样才能保证replicas足够快，才能保证producer发布消息时接收ACK的延迟较小

    * 当leader失效，需要考虑负载均衡，partition leader较少的broker更有可能成为新的leader，因为
      * 不能采用『投票多数派』的算法，因为这种算法对于『网络稳定性/投票参与者数量』要求较高
      * Kafka集群设计中，容忍N-1个replicas失效
      * 每个partiton中所有的replica信息都可以在ZK中获得，那么选择leader是非常简单的
      * 选择follower时需要注意：避免新的leader server上承载的partiton leader的个数过多，否则此server将承受更多的IO压力

  * 总结

    * Producer端直接连接broker列表，从列表中返回TopicMetadataResponse，该Metadata包含Topic下每个partition leader建立socket连接并发送消息。
    * Broker端使用ZK用来注册broker信息，以及监控partition leader存活性。
    * Consumer端使用ZK用来注册consumer信息，其中包括consumer消费的partition列表等，同时也用来发现broker列表，并和partition leader建立socket连接，并获取消息。

#### Kafka在Zookeeper中存储结构

  * 结构图

![kafka在ZK中的存储结构图]({{ site.url }}/assets/kafka.0.9.0/kafka_in_zk.png)

#### Kafka 安装和配置



#### 参考文献

  * http://blog.csdn.net/zhongwen7710/article/details/41252649
  * http://kafka.apache.org/documentation.html#brokerconfigs
