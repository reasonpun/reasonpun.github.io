---
layout: post
title: Oryx2 性能优化文档
date: 2016-04-07 18:02:16
---

这里收集了各种意见，经验法则和基准测试相关的性能：做这些不同的工作需要多少资源。

<!--more-->

## 硬件和集群设计

一般情况下，对硬件或者集群没有特别的要求。集群资源的需求主要取决于基于Spark的作业，这些往往是内存密集型和CPU密集型的，但是一般不会是I/O绑定的。如果数据的采集率非常的高，Kafka可能需要一些特殊的考虑。在这两种情况下，针对其他任何的Kafka或者Spark作业并没有不同的标准。

### 数据传输

因为Kafka是底层传输数据的，存储要求的采集和存放数据对于Kafka也就是这样的。查看[Kafka 性能](http://kafka.apache.org/performance.html)中提到的内容。一般情况下，Kafka并不会接近瓶颈，也能够像其他Kafka的使用一样调整资源占用大小。

### 批处理层

批处理层独特的地方是模型的构建, and the element that is of most interest to benchmark are likely the model building processes implemented in the app tier, on top of MLlib. Here again, the resources required to build a model over a certain amount of data are just that of the underlying MLlib implementations of ALS, k-means and decision forests.

MLlib的任何性能优化或者是基准测试都适用于这些基于MLlib预制的实现的批处理层，这对于Oryx也是一样的。

### JVM 优化

Choosing the number of Spark executors, cores and memory is a topic in its own right.

很自然的，更多的executors意味着更多的核心和内存。但是数量是不能超多集群上机器的总数量的；请查看：oryx.batch.streaming.num-executors。

更多的核，意味着可能会有更多的并发进程。在典型的模型构建进程中，如果能达到任务总数的三分之一或者2分之一的话，这将是非常有用的。你可以观察到任务的数量，以及Spark UI中批处理层固有的并发情况。在这数量之下，核数再多也无法增加更多的并发了。少一些是可以的，无非就是增加了点运行时间。当然，在批处理的时间间隔内，充足的核数可以保证批处理的顺利完成。核的数量可以通过oryx.batch.streaming.executor-cores进行配置。

如果你的作业发生了内存溢出，驱动器和executors可能需要更多的内存。如果你注意到在批处理层的”存储”标签页中并不是100%的RDD被缓存住了，那么更多的内存可能会有所帮助。查看：oryx.batch.streaming.executor-memory。

oryx-run.sh脚本的--jvm-args参数是用来为所有的JVM进程设置内存参数的。举个例子，通过设置 -XX:+UseG1GC ，会达到一个比较合理的效果。
### 服务层

REST API 后端服务器是Tomcat。配置文件是没有暴露给用户的，但是对于他的负载已经做了合理的调整。Tomcat容器本身开销很小，不需要担心性能问题。

最可能感兴趣的是项目中提供的关于CPU密集型程序实现的性能问题，而不是框架本身。

### 基准测试: 交替最小二乘法推荐

由于ALS实现的程序的服务层的大多数的操作是，会在内存中实时的计算一个非常大的矩阵，所以这个程序是最具有挑战性的了。大概其的规则如下：

如果需要运行类似的基准测试，可以使用LoadBenchmark，可以按照如下的配置：

```
mvn -DskipTests clean install
cd app/oryx-app-serving
...
mvn -Pbenchmark \
 -Doryx.test.als.benchmark.users=1000000 \
 -Doryx.test.als.benchmark.items=5000000 \
 -Doryx.test.als.benchmark.features=250 \
 -Doryx.test.als.benchmark.lshSampleRate=0.3 \
 -Doryx.test.als.benchmark.workers=2 \
 integration-test

```

### 内存

  * 内存需求是成线性的：(users + items) x features
  * -XX:+UseStringDeduplication 在 Java 8中是非常有用的 (reflected below)
  * At scale, 1M users or items ~= 500-1000M of heap required, depending on features

  Example steady-state heap usage (Java 8):


|Features|Users+Items (M)|Heap (MB)|
|--|--|--|
|50|2|1400|
|50|6|2600|
|50|21|7500|
|250|2|3000|
|250|6|7500|
|250|21|25800|

### 请求延迟，吞吐量

  * Recommend and similarity computation time scales linearly with items x features
  * A single request is parallelized across CPUs; max throughput and minimum latency is already achieved at about 1-2 concurrent requests
  * Locality sensitive hashing decreases processing time roughly linearly; 0.33 ~= 1/0.33 ~= 3x faster (setting too low adversely affects result quality)

  Below are representative throughput / latency measurements for the /recommend endpoint using
  a 32-core Intel Xeon 2.3GHz (Haswell), OpenJDK 8 and flags -XX:+UseG1GC -XX:+UseStringDeduplication. Heap size was comfortably large enough for the data set in each case. The tests were run with 1-3 concurrent request at a time, as necessary to achieve near-full CPU utilization.

_With LSH (sample rate = 0.3)_

|Features|Items (M)|Throughput (qps)|Latency (ms)|
|--|--|--|--|
|50|1|437|7|
|250|1|151|13|
|50|5|84|24|
|250|5|36|56|
|50|20|14|69|
|250|20|6|162|

_Without LSH (sample rate = 1.0)_

|Features|Items (M)|Throughput (qps)|Latency (ms)|
|--|--|--|--|
|50|1|74|27|
|250|1|23|44|
|50|5|13|80|
|250|5|5|191|
|50|20|4|282|
|250|20|1|708|

### JVM 优化

机器上运行（多个）服务层，使用越多的可用的核，意味着可以提供更多的并发请求处理能力。在ALS中，一些请求，比如/recommend 可以在一个请求中通过多核计算完成。

内存的需求主要是由加载进内存的模型的需求控制的。对于大的模型，比如ALS，这就意味着需要确保服务层有足够的内存以保证不会导致GC崩溃。查看oryx.serving.memory。

-XX:+UseG1GC remains a good garbage collection setting to supply with --jvm-args. In Java 8, -XX:+UseStringDeduplication can reduce memory requirements by about 20%.

### 实时计算层

实时计算层驱动进程类似服务层那样也是需要大量内存的，因为他也是需要加载一个模型到内存中的。驱动进程的内存是通过oryx.speed.streaming.driver-memory控制的，也需要和服务层内存一样设置，并且也需要JVM的一些flags支持。

这也是一个Spark Streaming 作业，也需要想批处理层那样配置executors。一般情况下，需要更少的处理和更低的时间延迟。

Executors will have to be sized to consume input Kafka partitions fully in parallel; the number of cores times number of executors should be at least the number of Kafka partitions.
