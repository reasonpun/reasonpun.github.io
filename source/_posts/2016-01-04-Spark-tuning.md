---
layout: post
title: Spark优化
date: 2016-01-04 18:02:16
---

## Spark 优化

由于Spark内存计算特性，Spark程序会由集群上的如下因素决定其性能
  * CPU
  * 网络带宽
  * 内存

通常来说，如果配置适当的内存，那么瓶颈就是带宽。但是有些时候，有需要做些优化，比如以序列化的形式存储RDD，从而降低内存的占用。

<!--more-->

此会从两方面分析
  * 数据序列化
  * 内存优化

### Data Serialization

序列化在分布式计算程序中占有非常重要的地位。迟缓的序列化格式，或者消费一个超大的字节数据，都会大大的减缓计算速度。所以，第一件事应该先尝试优化Spark程序。Spark试图达到在易用性（允许你定义任何的Java类型）和性能两方面达到一种平衡，并且提供两种序列化库：
  * Java序列化：缺省情况下，Spark使用Java的ObjectOutputStream框架序列化对象，
  从而可以和任何实现了java.io.Serializable接口的类一起玩耍。也可以更紧密的控制扩展了java.io.Externalizable的序列化的性能。Java的序列化是非常丰富的，但是速度奇慢，并且会导致很多类产生大量的序列化格式。

  * Kryo序列化：Spark也可以使用Kryo库（Version2）非常迅速的序列化对象。Kryo比Java序列化快很多并且更大的压缩率（通常是10x），但是并不支持所有的类型，另外也需要你在程序中注册类以获得最好的性能。

你可以选择通过设置SparkConf或者调用conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")来使用Kryo初始化Job。这样设置的话，不仅可以配置通过worker节点间shuffling数据，还能将RDD序列化到磁盘上。Kryo不是缺省配置的原因是由于自定义注册的要求决定的，同时我们也想在网络密集型应用中使用它。

Spark automatically includes Kryo serializers for the many commonly-used core Scala classes covered in the AllScalaRegistrar from the Twitter chill library.

使用Kryo注册自定义类，需要使用registerKryoClasses方法。

  ```
  val conf = new SparkConf().setMaster(...).setAppName(...)
  conf.registerKryoClasses(Array(classOf[MyClass1], classOf[MyClass2]))
  val sc = new SparkContext(conf)
  ```

Kryo文档描述了更多高级的注册选项，比如添加自定义序列代码。

如果你的对象非常大，则需要增加spark.kryoserializer.buffer配置参数。这个值缺省为2，但是这个值需要足够大足以保存序列号的对象。

最后，如果你没有注册自定义类，Kryo仍然会起作用，但是它不得不存储每个对象的全部类，这样是非常浪费的。

### 内存优化

内存优化的方法有3个方面需要考虑的：你的对象使用的总内存量（你可能希望全部数据都放到内存里），对象存取成本，GC的开销（如果你有非常高的对象交换频率）

默认情况下，Java对象可以被快速的访问，但是会轻易的耗尽它们字段中比“raw”数据多2到5倍的空间的因子。这取决于多个因素：
  * 每个独立的Java对象包含一个“对象头”，16字节长度并包含诸如只想它的类的一些信息。
  对于一个非常小的数据来说（比如Int），这个可能比数据本身都要大一些。
  * Java字符串有大约40字节的对象头，比“raw”数据要多（因为它们按照字符串数组的形式保存，并且还包含扩展数据，比如说数据的长度），
  由于采用了UTF-16编码格式，所以在字符串内部，存储一个字符需要占用2个字节的空间。因此10个字符就可以轻易的消耗掉60字节空间。
  * 普通的集合对象，比如说HashMap和LinkedList，使用的是链式数据结构，对于每个实体都存在一个“wrapper”对象（比如 Map.Entry）。这个对象不仅包含头，链表中还有指向下一个对象的指针（通常需要8个字节）。
  * 私有类型的集合经常以“装箱”对象的形式存储，比如java.lang.Integer。

这一章讨论的是如何确定对象的内存占用情况，和改进的方法－不止保护改变你的数据结构，另外还需要以一种序列号的形式存储数据。然后我们就可以覆盖到优化Spark缓存和Java GC的知识了。

#### 搞清楚内存消耗

测试一个数据集消耗内存的总量最好的方式创建一个RDD，并放到缓冲中，通过web页面查看存储情况。这个页面的内容可以显示RDD到底占有了多少内存。

估算一个分区数据消耗的内存，可以使用SizeEstimator’s estimate的方法，这是一种非常有用的方式去试验不同的数据结构去减少内存的使用，即可以确定广播变量占用空间可以消耗每个可执行堆的情况。

#### 优化数据结构

首选的降低内存消耗的方法是避免使用具有Java的特性导致的开销，比如基于指针的数据结构核包装类。有多种方式可以做到：
  * 设计你的数据结构以提升对象数组和原始类型，用来替换标准的Java或者Scala的集合类（比如：HashMap）。
  fastutil类库为原始类型提供了适当的集合类型，可以兼容Java的标准库。
  * 尽量避免包含大量小对象核指针的嵌套数据结构。
  * 考虑使用数字类型的ID或者枚举对象来替代字符串行的key。
  * 如果你的RAM不足32GB，可以通过设置JVM的-XX:+UseCompressedOops参数，修改指针默认占用8字节为4个字节。也可以讲这些参数加到spaspark-env.sh中。

#### 序列化的RDD存储

通过以上的优化，你的对象仍然很大从而影响到高效的存储的话，一种更加简单的减少内存使用的方法是通过RDD的持久化API序列化StorageLevels，从而使他们以序列化的方式存储，比如MEMORY_ONLY_SER。Spark这时就可以将每个RDD作为一个大字节数组分区存储。唯一的不足是，序列化存储的数据每次读取的时候会很慢，这个取决于每个对象的反序列化（on the fly）。我们强烈建议使用Kryo作为缓存序列化数据的方法，这样可以比Java序列化占用更少的空间（甚至是原始的Java对象）

#### GC优化
JVM 的GC可能会是一个问题，当你的程序在存储一个RDD方面存在一个很大的『churn』。（他不会是一个大问题，如果只是每次读取一个RDD，并多次操作）当Java需要逐渐的用新对象替换旧对象时，GC就会追踪你的所有Java对象，找到并替换掉。这里的重点指出的，GC的执行成本是与Java对象的个数成正比的，因此使用更少对象的数据结构可以大大降低GC的成本（比如一个Int类型的数组替换LinkedList的数组）。更好的方法是持久化被序列化之后的对象，如上述，在每个RDD分区里只会存在一个对象（一个字节数组）。在尝试其他技术之前，如果GC是一个问题，那么第一件事就是尝试序列化的缓存。

你的任务的占用的活动内存核缓存在你节点上的RDD之间的干扰也会导致GC出现问题（需要执行任务所需的内容总量）。我们将会讨论怎样控制分配给RDD缓存的空间以减少这种影响。

##### 衡量GC的影响

GC优化的第一步需要先手机统计数据：GC发生的频率和发费的时间。这个可以通过添加设置java参数-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps实现。（详情见传递Java参数到Spark任务的指南）下一次你的Spark任务执行时，你就可以在每个worker的日志中看到关于GC事件的信息。注意，这些日志时存放到worker节点上的，并不是在driver 程序上。

##### 缓冲大小优化

GC一个重要的配置参数是分配给缓存RDD使用的内存总量。缺省情况下，Spark会使用配置给executor memory (spark.executor.memory) 60%的内存量缓冲RDD。也就是说40%的内存是用于在任务执行过程中任何其它的对象。

当你的任务速度缓慢，并且发现JVM GC频率特别频繁或者出现内存溢出的问题时，降低这个值会对降低内存消耗提供帮助。修改这个值可以通过设置spark.storage.memoryFraction，比如修改成50%，则可以conf.set("spark.storage.memoryFraction", "0.5")。结合使用序列化缓存，使用更少的缓存就可以充分的减轻大部分的GC问题。

##### 高级GC优化

更进一步的GC优化，我们首先需要理解一些基本的JVM管理内存管理知识：
  * Java堆空间被划分为两个部分Young 和 Old。Young generation用来保存短周期的对象，同时Old generation是用来保持长周期对象。
  * Young generation被进一步划分为3部分：Eden, Survivor1, Survivor2。
  * 一个简单的关于垃圾回收程序的描述：当Eden满了的时候，a minor GC is run on Eden and objects that are alive from Eden and Survivor1 are copied to Survivor2。Survivor部分是可以交换的。如果一个对象已经旧了或者Survivor2满了，他就会被移动到旧的部分。最后，当旧的部分接近满的时候，一个满的GC就会被唤起。

Spark中GC优化的目标是确保只有需要长久存在的RDD才会被存储在Old generation，Young generation有足够的空间处处短周期的对象。这就可以帮助避免full GCs在任务执行过程中收集临时任务。这里的一些步骤可能会比较有用：
  * 通过检查GC的状态检查是否存在多个垃圾回收。如果一个full GC在一个任务结束之前被多次唤醒，这就意味着没有足够的可用内存涌来执行任务。
  * In the GC stats that are printed，如果OldGen接近满的状态的话，就减少缓存的内存总量。这个可以通过设置spark.storage.memoryFraction property来达成。缓存更少的对象总比减慢任务执行速度要好的多！
  * 如果存在很多的次要collections而不是很多的主要的GCs，给Eden分配更多的内存会有所帮助。你可以设置Eden稍微高些的内存给每个任务。如果Eden表示成E，那么可以通过设置Young generation的参数大小为-Xmn=4/3*E。（The scaling up by 4/3 is to account for space used by survivor regions as well.）
  * 举个例子，如果你的任务正在从HDFS中读取数据，任务使用的内存被标记为数据的块大小。
  注意，解压后的数据块大约是之前数据的2～3倍。因此我们希望有3或者4个任务的工作空间，并且HDFS块的大小是64M，我们就可以估算出Eden大小大约是4*3*64MB。
  * 通过修改新的设置来监测垃圾回收的频率和时间

我的经验得出GC优化的改进取决于你的程序和可用的内存总量。在网上有很多优化选项的描述，但是再进一步，管理full GC的频率可以有助于降低顶层限制。

### 其它需要考虑的事情

#### 并行的水平

节点不会被充分利用，除非喂每个操作设置了足够高的并行水平。Spark会自动设置“map”任务的数量执行每个文件以和他的大小保持一致（尽管你可以通过SparkContext.textFile的可选参数控制），并且对于分布式的“reduce”操作，比如groupByKey 和 reduceByKey，会使用最大的父RDD的分区数。你可以传递并行的level作为第二个参数，活着设置spark.default.parallelism配置属性改变默认值。通常情况下，我们建议在节点上每个CPU内核执行2～3个任务。

#### 降低任务的内存使用

有时，你可能得到OutOfMemoryError错误，但是这并不是因为你的Rdd不适合你的内存，而是因为任务中的其中一个的设置，比如其中一个reduce任务执行groupByKey时太大。Spark的shuffle操作（sortByKey, groupByKey, reduceByKey, join等）会创建一个哈希表并且每个任务会执行grouping操作，这个通常情况下也会很大。此时最简单的解决办法是增加并行的水平，以降低任务的输入集大小。Spark可以高效的支持任务，因为它可以重复利用一个executor JVM，并且有很低的任务加载消耗，所以你可以安全的增加并发水平，甚至超过你节点上核心的数目。

#### 广播大变量

使用SparkContext上可用的广播功能可以明显的减少每个序列化任务的大小，和通过节点加载任务的消耗。如果你的任务使用来自driver程序的大的对象（比如：静态查找表），试着转化成一个广播变量。Spark打印出主节点上每个任务的序列化后的大小，从而你可以通过这个决定你的任务是否过大；通常的任务大于20KB就值得优化了。

#### 数据的位置

数据的位置可能是影响Spark作业的最主要的一个影响因素。如果数据和代码被放置在一起的话，计算速度会明显加快。但是如果代码和数据是分开的，不同的两块数据会被移动到一起。特别是，传输序列化的代码比数据块要快很多，这是因为代码的大小比数据小很多。Spark构建调度的主要原则就是由数据的位置决定的。

数据的位置到底距离操作他的代码多近的距离才合适呢？这里存在多个级别（自近及远）：
  * PROCESS_LOCAL  数据和代码在同一个JVM中，这种方式是最好的一种情况
  * NODE_LOCAL 数据在同一个节点。比如说在HDFS的同一个节点，或者相同节点的不同executor。
  这种方式比PROCESS_LOCAL稍微慢些，因为这种情况下，数据需要在不同进程中传输
  * NO_PREF data is accessed equally quickly from anywhere and has no locality preference
  * RACK_LOCAL 数据在同一个服务器机架。数据在同一个机架的不同的服务器上的话，数据需要通过网络进行传输，typically through a single switch
  * ANY 数据分布在网络的各个地方，并且不在同一个机架

Spark更倾向于调度最好的locality级别的所有任务，但是这并不是经常发生的。在这种情况下，不在处理数据的任何空闲executor，Spark都会选择更低的locality级别。这有两种选择：
  * 等待同一个服务器上的任务，直到对应的CPU闲下来才执行
  * 立即执行一个新任务，并通过移动数据到更远的节点执行

Spark通常会稍微等一下，以等待CPU执行完毕。一旦超时，他就会将远端的数据传递给空闲的CPU。不同级别的等待超时回退可以单独配置，或者在一个参数中集中修改；前往配置页查看spark.locality参数详情。如果你的任务are long and see poor locality，你也可以修改设置，但是缺省值是完全可以满足要求的。

### 总结

这是一个简短的介绍，其中指出了你可能需要了解的优化Spark应用的最主要的要点，数据序列化和内存优化。对于大多数的程序来说，选择Kryo序列化，并且以序列化形式保存数据可以解决大部分一般的性能问题。Feel free to ask on the Spark mailing list about other tuning best practices.
