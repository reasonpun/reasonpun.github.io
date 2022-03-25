---
layout: post
title: Oryx2 简介
date: 2015-12-21 18:02:16
---


### 简介

![Oryx2]({{ site.url }}/assets/images/posts/oryx/OryxLogoMedium.png)


Oryx2是专注于进行大规模，实时机器学习框架，遵循lambda规则，基于Apache Spark和Apache Kafka构建。

Oryx 不仅是构建应用程序的框架，而且包含 协同过滤，分类，回归和聚类的打包的端到端的应用。

<!--more-->

包含3层
  * lambda层
    * 批量处理
    * 快速处理
    * 服务
  * ML抽象层
  * 端到端实现层

从另一个角度，可以看成是一系列链接的元素
  * 批处理层：依据历史数据进行离线处理
  * 实时处理层：通过增量数据流，实时更新结果
  * 服务层：通过模型的传递实现异步查询API
  * 数据传输层：在外部数据源与处理层之间传输数据

The project
  * may be reused tier by tier for example, the packaged app tier can be ignored, and it can be a framework for building new ML applications.
  * It can be reused layer by layer too: for example, the Speed Layer can be omitted if a deployment does not need incremental updates.
  * It can be modified piece-by-piece too: the collaborative filtering application’s model-building batch layer could be swapped for a custom implementation based on a new algorithm outside Spark MLlib while retaining the serving and speed layer implementations.

![Architecture]({{ site.url }}/assets/images/posts/oryx/Architecture.png)

### Lambda层实现

#### 数据传输

  数据传输机制其实就是一个Kafka的Topic。任何一个进程（包含且不局限与服务层）都可以向topic中写入数据，并通过实时处理和批处理层查看。
  Kafka Topic也可以用来模型和模型之间的更新，并被实时处理层和服务层消费。

#### 批处理层

  批处理层是以Spark Streaming进程的方式实现的，运行在Hadoop Cluster节点上，并读取来自Kafka topic的输入数据。 Streaming 进程会有一个很长的运行周期-若干小时甚至一天。会使用Spark存储当前会话数据到HDFS中，然后合并HDFS上的所有历史数据，之后重新初始化构建新的结果数据。并将新的结果重新写入HDFS，同时发不到Kafka更新topic中。

#### 实时处理层

  实时处理层也是由Spark Streaming进程实现的，同样读取Kafka topic输入数据。但是他存在比较短的运行周期，比如秒级别。会持续消费更新topic中的新模型，并生产新的模型。也会回写更新topic。

#### 服务层

  服务层监听更新topic上的模型以及模型更新。在内存中持久化模型状态。
  会暴露顶层方法的 HTTP REST API 用于查询内存中的模型。大部分接口都支持大规模的部署。
  每个接口都可以接收新的数据并写入Kafka，以此在实时处理层和批处理层可见。

#### 配置和部署

  程序是基于Java实现的，依赖
    * Spark 1.3.x+
    * Hadoop 2.6.x+
    * Tomcat 8.x+
    * Kafka 0.8.2+
    * Zookeeper 等。

  配置文件通过 [Typesafe Config](https://github.com/typesafehub/config) 的方式实现整个系统的部署配置。
  包括： 批处理，实时处理，服务层逻辑关键的接口类的实现

  每个层的二进制形式分开进行打包和部署的，每个都是以可执行的Java的jar包的形式存在并包含所有必须的服务。

### ML层实现

  ML层对上述通用接口方法做了简单的专一话的实现，实现了通用ML需求，并且对应用暴露了机器学习特有的接入接口。

  举个例子，实现了批量处理层，用于自动更新测试集和训练集进程。可以调用应用提供的函数来评估测试机模型。通过尝试不同的超参数值，选择出最佳结果。通过PMML管理模型的序列号。

### 端到端应用实现

  除了作为一种框架，Oryx2 包含完整的三中机器学习需要的批处理层，实时处理层，服务层。
  开箱即用，或者作为自定义程序的基础：

    * 基于最小二乘法的协同过滤/推荐
    * 基于k-means的聚类
    * 基于随机决策森林的分类和回归

### 参考文献

  * http://oryx.io/index.html
  * http://www.ivanopt.com/oryx-document%E7%BF%BB%E8%AF%91/
  * http://jameskinley.tumblr.com/post/37398560534/the-lambda-architecture-principles-for
  * http://dmg.org/pmml/v4-1/GeneralStructure.html
  * http://blog.csdn.net/nxcjh321/article/details/24796879
  * http://youngfor.me/post/recsys/oryx-tui-jian-xi-tong-chu-ti-yan
  * https://github.com/OryxProject/oryx
