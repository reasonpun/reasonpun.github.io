---
layout: post
title: Oryx2 开发者文档
---


### 环境需求
  * [git](http://git-scm.com/), 或者一个支持git的IDE
  * [Apache Maven](http://maven.apache.org/) 3.2.1 或者更新版本
  * [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) (不能只有JRE) 7 或者更新版本

以上需要已经安装到了你的开发环境中。

## Building

克隆代码到你本地，并且编译：

```
git clone https://github.com/OryxProject/oryx.git oryx
cd oryx
mvn -DskipTests package
```

会编译出如下的二进制的jar文件：

  * 批处理层: deploy/oryx-batch/target/oryx-batch-2.1.2.jar
  * 实时处理层: deploy/oryx-speed/target/oryx-speed-2.1.2.jar
  * 服务层: deploy/oryx-serving/target/oryx-serving-2.1.2.jar

友情提醒，如果你对开发Oryx感兴趣，可以根据这个[分支](https://help.github.com/articles/fork-a-repo)克隆自己的分支，然后就可以提交修改了。

### Java 8

如果使用Java8编译，需要添加参数-Pjava8 并且测试这里的指令。

### Platform Only

默认的编译包括基于Spark MLlib和其他库的端到端的ML程序。如果只是编译lambda和ML层，需要通过参数-P!app-tier关闭其他的选项。注意，在bash中，！需要转义： -P\!app-tier。

### 测试

mvn 测试会执行所有的但愿测试用例。也同时会执行所有的集成测试，这个可能会需要稍微长点的时间。

## 模型对应表

主要的模型和他们对应的层：

||Serving|Speed|Batch|
|--|--|--|--|
|Binary| oryx-serving| oryx-speed| oryx-batch|
|App|oryx-app-serving|oryx-app-mllib oryx-app|oryx-app-mllib oryx-app|
|ML||oryx-ml|oryx-ml|
|Lambda|oryx-lambda-serving|oryx-lambda|oryx-lambda|

支持的模型，比如：[oryx-common](https://github.com/OryxProject/oryx/tree/master/framework/oryx-common), [oryx-app-common](https://github.com/OryxProject/oryx/tree/master/app/oryx-app-common), [oryx-api](https://github.com/OryxProject/oryx/tree/master/framework/oryx-api), and [oryx-app-api](https://github.com/OryxProject/oryx/tree/master/app/oryx-app-api) 没有在这里列出了。

## 实现一个Oryx 程序

Oryx 中的“app 层”，是实现了推荐的真实的批处理，实时，服务层逻辑，集群和分类。然而，任何实现都需要使用到Oryx。他们也可以混合和匹配。举个例子，你可以重新实现ALS-related推荐的批处理层，但是仍然使用原来的ALS的服务层和实时计算层。

### 创建一个程序

在每个例子中，创建一个自定义的批处理层，实时计算层或者服务层的程序都需要实现com.cloudera.oryx.api中的几个关键的Java接口或者Scala的特性。这些接口/特性可以在项目的oryx-api模型中找到。

||Java|Scala|
|--|--|--|
|Batch|batch.BatchLayerUpdate|batch.ScalaBatchLayerUpdate|
|Speed|speed.SpeedModelManager|speed.ScalaSpeedModelManager|
|Serving|serving.ServingModelManager|serving.ScalaServingModelManager|

com.cloudera.oryx.api也包含大量的关键的类和接口，举个例子，[serving.OryxResource](https://github.com/OryxProject/oryx/blob/master/framework/oryx-api/src/main/java/com/cloudera/oryx/api/serving/OryxResource.java)  是一个入口，用来编译自定义的JAX-RS 端点，但是不需要使用。

### 编译程序

进入你的程序的这些接口/特性，添加一个com.cloudera.oryx:oryx-api的依赖，scope字段需要填写“provided”，在Maven中，需要添加如下依赖：
In Maven, this would mean adding a dependency like:

```
<dependencies>
  <dependency>
    <groupId>com.cloudera.oryx</groupId>
    <artifactId>oryx-api</artifactId>
    <scope>provided</scope>
    <version>2.1.2</version>
  </dependency>
</dependencies>
```

这些artifacts被放在了[Cloudera](https://repository.cloudera.com/artifactory/cloudera-repos/)这个分支下，因此在编译的时候需要引用这个分支：

```
<repositories>
  <repository>
    <id>cloudera</id>
    <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
  </repository>
</repositories>
```

一个最小的实例可以访问[example/](https://github.com/OryxProject/oryx/tree/master/app/example) 这里。

”Word Count” 这个程序是按照空格将行分割成独立的单词，然后统计出排重后的单词出现的次数。

编译代码后生成一个JAR文件，包含了程序实现和所有第三方的diam，如果使用Maven，可以通过mvn package命令。

### 编译 Word Count 例子

举例：编译样例代码如下：
```
cd app/example
mvn package
```

生成的JAR包在target/example-2.1.2.jar。

### 自定义Oryx程序

当发布一个源自Oryx的预打包程序，在某些情况下，有可能会提供一个扩展的实现，从而可以自定义他们的行为。举例，ALS推荐程序暴露了com.cloudera.oryx.app.als.RescorerProvider接口。这些特定程序API类可以在模块oryx-app-api中找到。这些接口的实现也可以在独立模式下被编译，打包，部署。

```
<dependencies>
  <dependency>
    <groupId>com.cloudera.oryx</groupId>
    <artifactId>oryx-app-api</artifactId>
    <scope>provided</scope>
    <version>2.1.2</version>
  </dependency>
</dependencies>

```

## 发布程序

拷贝生成的JAR文件--myapp.jar，放到需要执行的Oryx 二进制JAR文件相同的目录下。

修改Oryx的配置文件，以便于引用自定义的批处理，实时计算和服务层的实现。
当执行批处理，实时计算和服务层时，需要添加--app-jar myapp.jar到oryx-run.sh命令行中。

### 发布 Word Count 例子

举例，如果已经编译好了上述的“word count”的程序，你可以执行这个程序，直接引用wordcount-example.conf这个配置文件：

```
./oryx-run.sh batch --conf wordcount-example.conf --app-jar example-2.1.2.jar
```

... 对于实时计算和服务层也是同样的。

```
curl -X POST http://.../add/foo%20bar%20baz
...
curl http://.../distinct
{"foo":2,"bar":2,"baz":2}
```

配置文件本身已经配置好了主机名称和[Cloudera Quickstart VM](http://www.cloudera.com/content/www/en-us/downloads/quickstart_vms.html)的参数。事实上，这个例子可以作为一个集群配置的例子：[Cloudera Quickstart VM Setup](http://oryx.io/docs/admin.html#cloudera_quickstart_vm_setup)。
