---
layout: post
title: Oryx2 终端用户文档
---


### 运行

注意：你必须已经按照[管理员文档](http://reasonpun.com/2015/12/21/Oryx2-Admin-Docs/)中提到的配置好了你的集群。

下载最新的Oryx版本，包括批处理层，实时计算层和服务层的jar文件和sh脚本。

或者，源码编译他们并从deploy/bin/获取最新的脚本。

拷贝二进制和脚本到hadoop集群的机器上。他们可以会被部署到不同的机器，或者是被部署到一个测试机器上。实时和批处理层应该运行且只能运行在一台机器上（The Speed and Batch Layers should run on at most one machine, each），服务层则可以运行于多个节点上。

创建一个配置文件，可以简单的拷贝例子中的conf/als-example.conf。并修改host名称，端口和目录。实际上，选择hdfs上已经存在的数据和模型目录便于用户运行Oryx 二进制命令。

拷贝该配置文件，并重命名为oryx.conf，将他放到每个机器的上二进制命令和脚本相同的目录下。

执行如下命令开始这3个层的运行：

```
./oryx-run.sh batch
./oryx-run.sh speed
./oryx-run.sh serving
```

参数--layer-jar your-layer.jar and --conf your-config.conf可以指定某一层特定位置的jar文件和配置文件。也可以使用--jvm-args直接传递更多的参数给Spark驱动程序，比如：--jvm-args="-Dkey=value"

这都不需要在同一台机器上，但是也不一定（如果配置特殊指定批处理和实时处理层，服务层API不同的端口）。服务层可以运行在多个机器上。

举个例子，批处理层SparkUI运行在启动脚本所在的机器的4040端口（除非通过配置更改）。一个简单的基于web端的控制台的服务层默认是运行在8080端口的。

完美！


### 尝试下ALS的例子吧

如果你已经使用了上述的配置，你就已经可以运行一个基于ALS的推荐程序实例。
自获取GroupLens 100K的数据集，并且找到u.data文件，这个文件的内容需要转换成csv格式：

```
tr '\t' ',' < u.data > data.csv
```

将这些数据放入服务层，使用本地的命令行工具，如下：

```
wget --quiet --post-file data.csv --output-document - \
  --header "Content-Type: text/csv" \
  http://your-serving-layer:8080/ingest
```

如果你使用tail命令查看输入的内容，可以看到如下数据：

```
196,242,3.0,881250949186
196,242,3.0,881250949
186,302,3.0,891717742
22,377,1.0,878887116
244,51,2.0,880606923
166,346,1.0,886397596
298,474,4.0,884182806
...
```

很快的，你也可以看到批处理层已经开始触发一个新的计算了。这个例子被配置为5分钟一个周期。

数据首先会被写入HDFS。默认配置被写入hdfs:///user/example/Oryx/data/目录下。且目录以时间戳命名，每一部分都包含Hadoop part-r-* 文件，都是以文本的序列话文件的方式存储。虽然不是纯文本，打印出来的话，还是有一部分是可以识别的，因为这其实真的是文本。

```
SEQorg.apache.hadoop.io.Textorg.apache.hadoop.io.Text����^�]�XسN�22,377,1.0,87888711662...
```

模型计算开始。这些批处理层会以大量的新的分布式的作业形式展现。在这个例子中，Spark UI可以通过http://your-batch-layer:4040访问。

模型计算是非常快的，执行完毕以后会合并PMML和支持数据文件并存储到目录hdfs:///user/example/Oryx/model/下。举个例子，model.pmml 的内容如下：


```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<PMML xmlns="http://www.dmg.org/PMML-4_2" version="4.2.1">
    <Header>
        <Application name="Oryx"/>
        <Timestamp>2014-12-18T04:48:54-0800</Timestamp>
    </Header>
    <Extension name="X" value="X/"/>
    <Extension name="Y" value="Y/"/>
    <Extension name="features" value="10"/>
    <Extension name="lambda" value="0.001"/>
    <Extension name="implicit" value="true"/>
    <Extension name="alpha" value="1.0"/>
    <Extension name="XIDs">56 168 222 343 397 ...
     ...

```

The X/ and Y/ subdirectories next to it contain feature vectors, like:

```
[56,[0.5746282834154238,-0.08896614131333057,-0.029456222765775263,
  0.6039821219690552,0.1497901814774658,-0.018654312114339863,
  -0.37342063488340266,-0.2370768843521807,1.148260034028485,
  1.0645643656769153]]
[168,[0.8722769882777296,0.4370416943031704,0.27402044461549885,
  -0.031252701117490456,-0.7241385753098256,0.026079081002582338,
  0.42050973702065714,0.27766923396205817,0.6241033215856671,
  -0.48530795198811266]]
...
```

如果使用tail命令查看更新的内容。
这些数据会很快放入服务层，此时访问/ready会返回200 OK。

```
wget --quiet --output-document - --server-response \
  http://your-serving-layer:8080/ready
...
  HTTP/1.1 200 OK
  Content-Length: 0
  Date: Tue, 1 Sep 2015 13:26:53 GMT
  Server: Oryx
```

```
wget --quiet --output-document -  http://your-serving-layer:8080/recommend/17
...
50,0.7749542842056966
275,0.7373013861581563
258,0.731818692628511
181,0.7049967175706345
127,0.704518989947498
121,0.7014631029793741
15,0.6954683387287907
288,0.6774889711024022
25,0.6663619887033064
285,0.6398968471343595
```

恭喜！实时推荐系统搭建完毕！可以通过Ctrl-C关闭。

### API手册

Oryx 支持多种端到端的程序，包括服务层的REST 接口。

### 协同过滤和推荐

  * [/recommend](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/Recommend.html)
  * [/recommendToMany](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/RecommendToMany.html)
  * [/recommendToAnonymous](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/RecommendToAnonymous.html)
  * [/recommendWithContext](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/RecommendWithContext.html)
  * [/similarity](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/Similarity.html)
  * [/similarityToItem](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/SimilarityToItem.html)
  * [/knownItems](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/KnownItems.html)
  * [/estimate](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/Estimate.html)
  * [/estimateForAnonymous](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/EstimateForAnonymous.html)
  * [/because](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/Because.html)
  * [/mostSurprising](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/MostSurprising.html)
  * [/popularRepresentativeItems](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/PopularRepresentativeItems.html)
  * [/mostActiveUsers](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/MostActiveUsers.html)
  * [/mostPopularItems](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/MostPopularItems.html)
  * [/mostActiveUsers](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/MostActiveUsers.html)
  * [/item/allIDs](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/AllItemIDs.html)
  * [/ready](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/Ready.html)
  * [/pref](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/Preference.html)
  * [/ingest](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/als/Ingest.html)

### 分类 / 回归

  * [/predict](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/rdf/Predict.html)
  * [/classificationDistribution](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/rdf/ClassificationDistribution.html)
  * [/ready](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/rdf/Ready.html)
  * [/train](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/rdf/Train.html)

### 聚类

  * [/assign](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/kmeans/Assign.html)
  * [/distanceToNearest](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/kmeans/DistanceToNearest.html)
  * [/ready](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/kmeans/Ready.html)
  * [/add](http://oryx.io/apidocs/com/cloudera/oryx/app/serving/kmeans/Add.html)

### 配置

  * [app/conf/als-example.conf](https://github.com/OryxProject/oryx/blob/master/app/conf/als-example.conf)
  * [app/conf/kmeans-example.conf](https://github.com/OryxProject/oryx/blob/master/app/conf/kmeans-example.conf)
  * [app/conf/rdf-example.conf](https://github.com/OryxProject/oryx/blob/master/app/conf/rdf-example.conf)
