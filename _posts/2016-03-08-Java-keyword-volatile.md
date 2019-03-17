---
layout: post
title: Java关键字volatile
excerpt: Java关键字volatile标识一个变量“被存储在主内存中”。更准确的说法是：每次volatile变量会从主内存中读取，而不是从CPU缓存；每次volatile变量的写操作会写入主内存，而不仅仅是CPU缓存。
tag: [Java]
---

翻译自：http://tutorials.jenkov.com/java-concurrency/volatile.html

### Java Volatile 关键字

Java关键字volatile标识一个变量“被存储在主内存中”。更准确的说法是：每次volatile变量会从主内存中读取，而不是从CPU缓存；每次volatile变量的写操作会写入主内存，而不仅仅是CPU缓存。

实际上，自Java5开始，volatile关键字保证的是volatile声明的变量都是写入和读取自主内存的。以下将会详细讲解。

### Java volatile保证可见性

Java中volatile关键字保证了线程之间变量修改的可见性。这个可能会有点抽象，因此让我详细讲解下。

在多线程程序中，由于性能的原因，在操作非volatile变量的时候，每个线程会将变量自主内存拷贝🈯️CPU缓存。如果你的计算机是多核的，每个线程就会运行在不同的CPU上。也就是说，每个线程会拷贝变量到不同的CPU缓存中。见插图：

![volatile]({{ site.url }}/assets/volatile/java-volatile-1.png)

使用非volatile变量，当JVM自主内存读取数据到CPU缓存，或者自CPU缓存写入主内存时，是没有保证的。这可能会引起很多问题，接下来章节中会降到。

设想一种情况，多个线程可以访问类似如下的同一个共享对象，这个对象包含一个计数器变量：

```java

public class SharedObject {
    public int counter = 0;
}

```

另一个设想，只有线程1改变计数变量，但是线程1和线程2都可能经常读取这个计数变量。

如果计数变量没有被声明为volatile，那么自CPU缓存回写到主内存的过程中，就无法保证这个计数变量的值了。也就是说，技术变量的值在通过CPU缓存写会主内存的过程中发生了改变。如图：

![volatile 2]({{ site.url }}/assets/volatile/java-volatile-2.png)

这个问题是因为线程并没有得到最新的值，另外那个线程并没有将最新的变量写回主内存，这就是所谓的“可见”问题。某一个线程的更新并没有被另外的线程获取。

通过声明计数变量为volatile，那么这个计数变量所有的更新都会立即被写回主内存。同时，所有的读操作都会直接通过主内存。如下既是如何声明计数变量为volatile类型：

```java

public class SharedObject {
    public volatile int counter = 0;
}

```

声明一个变量为volatile由此可以保证了变量的写操作对于其他线程都是可见的。

### The Java volatile Happens-Before Guarantee

自Java5开始，volatile关键字保证的不止是变量的主内存读和写，实际上还包含：

  * 如果线程A写入一个volatile变量，此时线程B读取了相同的volatile变量，那么此时所有的变量在写入volatile变量前对线程A是可见的，同时也对线程B在读取了volatile变量后也是可见的。

  * volatile变量的读写指令不能促使JVM重新排序（只要JVM发现排序不会改变程序的行为，基于性能的原因，JVM会进行排序）。前后的指令会被排序，但是volatile得读写不会混入这些指令。无论如何，指令都会follow一个volatile变量的读写，并且保证这种操作发生在读或者写之后。

这个需要更深入的解释。

当一个线程写入一个volatile变量时，并不止这个volatile变量自己写入了主内存。同时所有被线程修改的变量在写入volatile变量之前都会被回写到主内存。当一个线程读取一个volatile变量时，他会同时读取主内存中所有其他的变量，这些变量都是和volatile变量一同写入主内存的。

看例子：

```java

// Thread A:
    sharedObject.nonVolatile = 123;
    sharedObject.counter     = sharedObject.counter + 1;

// Thread B:
    int counter     = sharedObject.counter;
    int nonVolatile = sharedObject.nonVolatile;

```

因为线程A在写入volatile变量sharedObject.counter之前写入了非volatile变量sharedObject.nonVolatile，那么当线程A写入sharedObject.counter时，sharedObject.nonVolatile 和 sharedObject.counter 都会被写入主内存。

由于线程B开始的时候读取volatile类型的sharedObject.counter，那么sharedObject.counter and sharedObject.nonVolatile都会使用线程B自主内存读取到CPU缓存。等到线程B读取sharedObject.nonVolatile时，就会看到线程A写入的值。

开发者可能会使用这种扩展的可见性，保证了线程之间变量的可见性。只需要声明一个或者非常少的volatile变量替换掉每个变量都声明为volatile。如下是一个例子：

```java

public class Exchanger {

    private Object   object       = null;
    private volatile hasNewObject = false;

    public void put(Object newObject) {
        while(hasNewObject) {
            //wait - do not overwrite existing new object
        }
        object = newObject;
        hasNewObject = true; //volatile write
    }

    public Object take(){
        while(!hasNewObject){ //volatile read
            //wait - don't take old object (or null)
        }
        Object obj = object;
        hasNewObject = false; //volatile write
        return obj;
    }
}

```

线程A通过调用put()持续的写入对象。线程B通过take()持续的获取对象。这个类只有在线程A调用put()和线程B调用take()时，通过使用volatile变量才能工作正常。

如果JVM在不改变排序指令的语义的基础上实现，那么JVM则会通过记录JAVA指令优化性能。

```java

while(hasNewObject) {
    //wait - do not overwrite existing new object
}
hasNewObject = true; //volatile write
object = newObject;

```

注意到volatile变量hasNewObject在被实际设置前已经被执行。对于JVM，这看上去完全合法。这两个写入变量的值相对于另外一个是独立的。

然而，排序执行的指令会有损对象变量的可见性。首先，线程B可能在线程A设置新值之前将hasNewObject设置为true。其次，甚至没法保证当新值写入对象后回写到主内存。

为了防止如上述情况的发生，volatile关键字采用“发生前保证”机制。”发生前保证“机制保证读和写volatile变量的指令不能被排序。前后指令可以被排序，但是volatile读或者写指令发生前后不能被排序。

举个例子：

```java

sharedObject.nonVolatile1 = 123;
sharedObject.nonVolatile2 = 456;
sharedObject.nonVolatile3 = 789;

sharedObject.volatile     = true; //a volatile variable

int someValue1 = sharedObject.nonVolatile4;
int someValue2 = sharedObject.nonVolatile5;
int someValue3 = sharedObject.nonVolatile6;

```

JVM会排序前3个指令，只要他们在volatile写指令之前。

同样的，只要volatile写入质量发生在后三个指令之前，那么后三个指令就会被排序。在此之前，这三个指令中任何一个指令都不会被排序。
That is basically the meaning of the Java volatile happens before guarantee.

### volatile 并不总是适用的

即使volatile关键字保证了所有的volatile变量直接从主内存读取，所有的写操作都是直接写入主内存，但是存在声明了volatile而仍然不够的情况。

之前提过的，只有线程1写入共享的counter变量，声明为volatile的时候才能确保线程2总是能获取最新的写入值。

事实上，多线程甚至能写入一个共享的volatile变量，如果这个新写入的值不依赖于旧数据的话就将正确的值存入主内存。换句话说, 如果一个线程写入一个值到共享的volatile变量，并不会马上将读取的值用于下一个值得话。

一旦一个线程首次读取一个volatile变量的值，并且需要基于这个值生成一个新的共享volatile变量值，那么这个volatile变量就不能保证正确的可见性。读取volatile变量的值并且写入新值得很短的间隙，就会创建一个竞争的条件：多个线程会读取同一个volatile变量的值，并且生成一个新的值，当回写进主内存后就会覆盖掉其他的值。

多线程增加相同的计数器值得这种情况，准确的说对于volatile变量是不适用的。一下片段讲话详细讲述这种情况。

设想，如果线程1读取共享计数变量，他的值是0，并放入CPU的缓存，使其加1，但是这样的改变并不会回写如主内存。线程2此时可以自主内存读取相同的计数器变量，但是读到的值仍然是0，并将这个值放入了自己的CPU缓存。线程2也可以对其加1，并且可以回写到主内存。这种情况详见下图：

![volatile 3]({{ site.url }}/assets/volatile/java-volatile-3.png)

线程1和线程2此时并不是同步的。这个共享计数器的值应该是2，但是每个线程在自己的CPU缓存中存放的值却是1，在主内存中的值是0.这已经完全乱了！即使最后每个线程回写这个共享计数变量的值到主内存，这个值也是错误的。

### volatile在什么时候是适用的?

就像之前提到过的，如果是两个线程都需要读取和写入一个共享变量，那么此时适用volatile关键字是不适用的。那么就需要使用synchronized来保证读和写的原子性。读写volatile变量不会阻塞线程的读写。为了实现这一点，你必须使用synchronized关键字。

作为synchronized可供选择，需要采用java.util.concurrent包中的某一个。比如AtomicLong，AtomicReference或者其他的任意一个。

在这种情况下，只有一个线程读写volatile变量的值，其他线程只是读取变量，那么读取线程就可以通过volatile变量保证可以读取到最新的写入值。如果没有volatile变量，则这个过程就不能被保证。

volatile关键字可以被用在32和64位变量中。

### volatile 性能注意事项

volatile变量的读写会触发变量在主内存中的读写。主内存中的读写会比CPU缓存消耗的代价高。使用volatile变量同样会阻止普通性能增强指令的排序。因此，只能在确实需要增加变量可见性基础上使用volatile变量。

### 参考文献
  * http://www.ibm.com/developerworks/cn/java/j-jtp06197.html
  * http://www.infoq.com/cn/articles/ftf-java-volatile
  * http://www.infoq.com/cn/articles/java-memory-model-4
  * http://sakyone.iteye.com/blog/668091
  * https://zh.wikipedia.org/wiki/Volatile%E5%8F%98%E9%87%8F
  * http://www.cnblogs.com/aigongsi/archive/2012/04/01/2429166.html
