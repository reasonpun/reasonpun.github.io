---
layout: post
title: Apache Flume-ng Structure
---

## Apache-flume NG 配置

### 简介

  Flume NG是一个分布式、可靠、可用的系统，它能够将不同数据源的海量日志数据进行高效收集、聚合、移动，最后存储到一个中心化数据存储系统中。

  由原来的Flume OG到现在的Flume NG，进行了架构重构，并且现在NG版本完全不兼容原来的OG版本。

  经过架构重构后，Flume NG更像是一个轻量的小工具，非常简单，容易适应各种方式日志收集，并支持failover和负载均衡。


### 架构设计要点

#### 核心概念

  * Event：一个数据单元，带有一个可选的消息头
  * Flow：Event从源点到达目的点的迁移的抽象
  * Client：操作位于源点处的Event，将其发送到Flume Agent
  * Agent：一个独立的Flume进程，包含组件Source、Channel、Sink
  * Source：用来消费传递到该组件的Event
  * Channel：中转Event的一个临时存储，保存有Source组件传递过来的Event
  * Sink：从Channel中读取并移除Event，将Event传递到Flow Pipeline中的下一个Agent（如果有的话）

#### 架构图

![flume-ng总体结构图]({{ site.url }}/assets/flume-ng/flume-ng-architecture.png)

#### 基本流程

外部系统产生日志，直接通过Flume的Agent的Source组件将事件（如日志行）发送到中间临时的channel组件，最后传递给Sink组件，HDFS Sink组件可以直接把数据存储到HDFS集群上。

#### 单Agent
  一个最基本Flow的配置，格式如下：

  ```
  # list the sources, sinks and channels for the agent
  <Agent>.sources = <Source1> <Source2>
  <Agent>.sinks = <Sink1> <Sink2>
  <Agent>.channels = <Channel1> <Channel2>

  # set channel for source
  <Agent>.sources.<Source1>.channels = <Channel1> <Channel2> ...
  <Agent>.sources.<Source2>.channels = <Channel1> <Channel2> ...

  # set channel for sink
  <Agent>.sinks.<Sink1>.channel = <Channel1>
  <Agent>.sinks.<Sink2>.channel = <Channel2>
  ```

  尖括号里面的，我们可以根据实际需求或业务来修改名称。

  下面详细说明：

  * <Agent> 表示配置一个Agent的名称，一个Agent肯定有一个名称。
  * <Source1>和<Source2>是Agent的Source组件的名称，消费传递过来的Event。
  * <Channel1>和<Channel2>是Agent的Channel组件的名称。
  * <Sink1>与<Sink2>是Agent的Sink组件的名称，从Channel中消费（移除）Event。

  上面配置内容中

  * 第一组中配置Source、Sink、Channel，它们的值可以有1个或者多个；
  * 第二组中配置Source将把数据存储（Put）到哪一个Channel中，可以存储到1个或多个Channel中，
    同一个Source将数据存储到多个Channel中，实际上是Replication；
  * 第三组中配置Sink从哪一个Channel中取（Task）数据，一个Sink只能从一个Channel中取数据。

#### 多个Agent顺序连接

![flume-ng 多个Agent顺序连接]({{ site.url }}/assets/flume-ng/flume-multiseq-agents.png)

  可以将多个Agent顺序连接起来，将最初的数据源经过收集，存储到最终的存储系统中。这是最简单的情况，一般情况下，应该控制这种顺序连接的Agent的数量，因为数据流经的路径变长了，如果不考虑failover的话，出现故障将影响整个Flow上的Agent收集服务。

#### 多个Agent的数据汇聚到同一个Agent

![flume-ng 多个Agent的数据汇聚到同一个Agent]({{ site.url }}/assets/flume-ng/flume-join-agent.png)

  这种情况应用的场景比较多，比如要收集Web网站的用户行为日志，Web网站为了可用性使用的负载均衡的集群模式，每个节点都产生用户行为日志，可以为每个节点都配置一个Agent来单独收集日志数据，然后多个Agent将数据最终汇聚到一个用来存储数据存储系统，如HDFS上。

#### 多路（Multiplexing）Agent

![flume-ng 多路（Multiplexing）Agent]({{ site.url }}/assets/flume-ng/flume-multiplexing-agent.png)

  这种模式，有两种方式

  * 一种是用来复制（Replication）
    * Replication方式，可以将最前端的数据源复制多份，分别传递到多个channel中，每个channel接收到的数据都是相同的，配置格式

      ```
        # List the sources, sinks and channels for the agent
        <Agent>.sources = <Source1>
        <Agent>.sinks = <Sink1> <Sink2>
        <Agent>.channels = <Channel1> <Channel2>

        # set list of channels for source (separated by space)
        <Agent>.sources.<Source1>.channels = <Channel1> <Channel2>

        # set channel for sinks
        <Agent>.sinks.<Sink1>.channel = <Channel1>
        <Agent>.sinks.<Sink2>.channel = <Channel2>

        <Agent>.sources.<Source1>.selector.type = replicating
      ```

      使用的Replication方式，Source1会将数据分别存储到Channel1和Channel2，这两个channel里面存储的数据是相同的，然后数据被传递到Sink1和Sink2。

  * 另一种是用来分流（Multiplexing）
    * Multiplexing方式，selector可以根据header的值来确定数据传递到哪一个channel

      ```
        # Mapping for multiplexing selector
        <Agent>.sources.<Source1>.selector.type = multiplexing
        <Agent>.sources.<Source1>.selector.header = <someHeader>
        <Agent>.sources.<Source1>.selector.mapping.<Value1> = <Channel1>
        <Agent>.sources.<Source1>.selector.mapping.<Value2> = <Channel1> <Channel2>
        <Agent>.sources.<Source1>.selector.mapping.<Value3> = <Channel2>
        #...

        <Agent>.sources.<Source1>.selector.default = <Channel2>
      ```

      上面selector的type的值为multiplexing，同时配置selector的header信息，还配置了多个selector的mapping的值，即header的值：如果header的值为Value1、Value2，数据从Source1路由到Channel1；如果header的值为Value2、Value3，数据从Source1路由到Channel2。

#### 实现load balance功能

![实现load balance功能]({{ site.url }}/assets/flume-ng/flume-load-balance-agents.png)

  Load balancing Sink Processor能够实现load balance功能，上图Agent1是一个路由节点，
  负责将Channel暂存的Event均衡到对应的多个Sink组件上，而每个Sink组件分别连接到一个独立的Agent上

  ```
    a1.sinkgroups = g1
    a1.sinkgroups.g1.sinks = k1 k2 k3
    a1.sinkgroups.g1.processor.type = load_balance
    a1.sinkgroups.g1.processor.backoff = true
    a1.sinkgroups.g1.processor.selector = round_robin
    a1.sinkgroups.g1.processor.selector.maxTimeOut=10000
  ```

#### 实现failover能

  Failover Sink Processor能够实现failover功能，具体流程类似load balance，
  但是内部处理机制与load balance完全不同：Failover Sink Processor维护一个优先级Sink组件列表，只要有一个Sink组件可用，
  Event就被传递到下一个组件。如果一个Sink能够成功处理Event，则会加入到一个Pool中，否则会被移出Pool并计算失败次数，设置一个惩罚因子

  ```
    a1.sinkgroups = g1
    a1.sinkgroups.g1.sinks = k1 k2 k3
    a1.sinkgroups.g1.processor.type = failover
    a1.sinkgroups.g1.processor.priority.k1 = 5
    a1.sinkgroups.g1.processor.priority.k2 = 7
    a1.sinkgroups.g1.processor.priority.k3 = 6
    a1.sinkgroups.g1.processor.maxpenalty = 20000
  ```

### 安装配置

```
# 下载二进制包
[mofun_mining@i-tev02vc1 ~]$ wget "http://apache.arvixe.com/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz"
[mofun_mining@i-tev02vc1 ~]$ tar xvzf apache-flume-1.6.0-bin.tar.gz
[mofun_mining@i-tev02vc1 ~]$ mv apache-flume-1.6.0-bin /usr/local/
# 修改配置文件
[mofun_mining@i-qe32ajmq conf]$ pwd
/usr/local/apache-flume-1.6.0-bin/conf
[mofun_mining@i-qe32ajmq conf]$ sudo cp flume-conf.properties.template flume-conf.properties
```

采用 Avro Source+Memory Channel+HDFS Sink 方式

  * 服务器（日志汇总服务器agent）端配置文件

  ```
  [mofun_mining@i-tev02vc1 ~]$ cd /usr/local/apache-flume-1.6.0-bin/conf/
  [mofun_mining@i-tev02vc1 conf]$ ls
  flume-conf.properties  flume-conf.properties.template  flume-env.ps1.template  flume-env.sh  flume-env.sh.template  log4j.properties
  [mofun_mining@i-tev02vc1 conf]$ pwd
  /usr/local/apache-flume-1.6.0-bin/conf
  [mofun_mining@i-tev02vc1 conf]$ sudo vim flume-conf.properties
  # example.conf: A single-node Flume configuration

  # Name the components on this agent
  agent1.sources = r1
  agent1.sinks = k1
  agent1.channels = c1

  # Describe/configure the source
  agent1.sources.r1.type = avro
  agent1.sources.r1.bind = 192.168.1.33
  agent1.sources.r1.port = 41414
  agent1.sources.r1.channels = c1

  # Describe the sink
  agent1.sinks.k1.type = hdfs
  agent1.sinks.k1.channel = c1
  agent1.sinks.k1.hdfs.fileType = DataStream
  agent1.sinks.k1.hdfs.useLocalTimeStamp = true
  agent1.sinks.k1.hdfs.path = /flume/events/%Y-%m-%d
  #agent1.sinks.k1.hdfs.round = true
  #agent1.sinks.k1.hdfs.roundValue = 10
  #agent1.sinks.k1.hdfs.roundUnit = minute
  agent1.sinks.k1.hdfs.rollCount = 5000
  agent1.sinks.k1.hdfs.rollSize = 0
  agent1.sinks.k1.hdfs.rollInterval= 0

  # Use a channel which buffers events in memory
  agent1.channels.c1.type = memory
  agent1.channels.c1.capacity = 10000
  agent1.channels.c1.transactionCapacity = 1000
  ```

  * 客户端（日志收集agent）

  ```
  [reason@i-qunray9x conf]$ cd /usr/local/apache-flume-1.6.0-bin/conf/
  [reason@i-qunray9x conf]$ pwd
  /usr/local/apache-flume-1.6.0-bin/conf
  [reason@i-qunray9x conf]$ sudo vim flume-conf.properties

  # example.conf: A single-node Flume configuration

  # Name the components on this agent
  agent1.sources = r1
  agent1.sinks = k1
  agent1.channels = c1

  # Describe/configure the source
  agent1.sources.r1.type = exec
  agent1.sources.r1.command = tail -n 0 -F /home/reason/1.txt
  agent1.sources.r1.channels = c1

  # Describe the sink
  agent1.sinks.k1.type = avro
  agent1.sinks.k1.channel = c1
  agent1.sinks.k1.hdfs.useLocalTimeStamp = true
  agent1.sinks.k1.hdfs.path = /flume/events/%Y-%m-%d
  agent1.sinks.k1.hostname=192.168.1.33
  agent1.sinks.k1.port = 41414

  # Use a channel which buffers events in memory
  agent1.channels.c1.type = memory
  agent1.channels.c1.capacity = 5000
  agent1.channels.c1.transactionCapacity = 500
  ```

  * 启动服务器

  ```
    [mofun_mining@i-tev02vc1 conf]$
    /usr/local/apache-flume-1.6.0-bin/bin/flume-ng agent -c ./conf/ -f /usr/local/apache-flume-1.6.0-bin/conf/flume-conf.properties -n agent1 -Dflume.root.logger=INFO,console
  ```

  * 启动客户端

  ```
    [reason@i-qunray9x conf]$
    /usr/local/apache-flume-1.6.0-bin/bin/flume-ng agent -c conf -f /usr/local/apache-flume-1.6.0-bin/conf/flume-conf.properties -n agent1
  ```

  * 测试

  ```
  [mofun_mining@i-r6cuv8iq ~]$ hdfs dfs -ls /flume/events/2015-12-15
  Found 40 items
  -rw-r--r--   2 mofun_mining supergroup      34844 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340281
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340282
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340283
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340284
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340285
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340286
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340287
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340288
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39 /flume/events/2015-12-15/FlumeData.1450172340289
  -rw-r--r--   2 mofun_mining supergroup      34850 2015-12-15 17:39
  ...
  ```
  此时，通过nginx实时产生的日志，即可实时插入到hdfs中了。

### 参考文献

  * http://shiyanjun.cn/archives/915.html
  * http://my.oschina.net/leejun2005/blog/288136
  * http://tech.meituan.com/mt-log-system-optimization.html
  * http://www.ixirong.com/2015/05/18/how-to-install-flume-ng/
  * https://flume.apache.org/FlumeUserGuide.html#setting-up-an-agent
  * http://m.blog.csdn.net/blog/xueliang1029/24039459
