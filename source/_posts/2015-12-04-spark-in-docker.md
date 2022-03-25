---
layout: post
title: Docker中搭建Spark计算框架
date: 2015-12-04 18:02:16
excerpt: Spark计算框架在Docker中的部署步骤
tag: [Spark, Docker]
---


#### 安装好Docker之后

#### 先拉取一个官方的基本镜像ubuntu

docker pull ubuntu

我们将在这个基础镜像上运行容器，将这个容器当成一个普通的ubuntu虚拟机来操作部署spark，最后将配置好的容器commit为一个镜像，之后就可以通过这个镜像运行n个节点来完成集群的搭建

<!--more-->

#### 下载完ubuntu镜像之后运行

```
[reason@localhost ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
mofun/spark         v2.0                b2dacac3e132        About an hour ago   1.508 GB
mofun/spark         v1.3                4d2f33ca61ee        18 hours ago        1.506 GB
ubuntu              latest              0a17decee413        6 days ago          188.3 MB
docker/whalesay     latest              ded5e192a685        4 months ago        247 MB
[reason@localhost ~]$
```

#### 运行ubuntu容器

```
[reason@localhost ~]$ docker run -v /home/docker/software/:/software -it ubuntu
root@f4c0a9d42852:/#
```

#### 在容器中安装ssh

```
[reason@localhost ~]$ docker run -v /home/docker/software/:/software -it ubuntu
root@3970c1e5466e:/# apt-get install ssh
Reading package lists... Done
Building dependency tree
Reading state information... Done

# ssh默认配置root无法登陆
root@3970c1e5466e:~/.ssh# vim /etc/ssh/sshd_config
root@3970c1e5466e:~/.ssh#
# 将 /etc/ssh/sshd_config中PermitRootLogin no/without_passwd 改为yes

# 生成访问密钥
root@3970c1e5466e:~# ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
b5:d1:e1:dd:98:7d:be:cc:55:69:c6:e7:67:80:d8:d3 root@3970c1e5466e
The key's randomart image is:
+--[ RSA 2048]----+
|            .    |
|           = =.=.|
|          + * E=*|
|         . o .o++|
|        S .     *|
|              o.+|
|               + |
|                 |
|                 |
+-----------------+
root@3970c1e5466e:~#
root@3970c1e5466e:~# cd ~/.ssh/
root@3970c1e5466e:~/.ssh# cat id_rsa.pub >> authorized_keys
root@3970c1e5466e:~/.ssh#
root@3970c1e5466e:~/.ssh# vim ~/.bashrc
#加入/usr/sbin/sshd
#如果在启动容器的时候还是无法启动ssh的话，
# 在/etc/rc.local文件中也加入
root@3970c1e5466e:~/.ssh# vim /etc/rc.local
#加入
root@3970c1e5466e:~/.ssh# /usr/sbin/sshd
Missing privilege separation directory: /var/run/sshd
root@3970c1e5466e:~/.ssh# mkdir /var/run/sshd
root@3970c1e5466e:~/.ssh# /usr/sbin/sshd
root@3970c1e5466e:~/.ssh#
# 开启ssh服务后验证是否可以使用，打印出当前时间
root@3970c1e5466e:~/.ssh# ssh localhost date
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is ab:43:27:e6:1c:44:be:2c:f1:17:27:90:6d:2c:68:86.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
Mon Oct 19 05:57:44 UTC 2015
root@3970c1e5466e:~/.ssh#
# ssh安装完毕
```

#### 安装JDK

可以使用apt-get方式直接下载安装jdk（不推荐，下载速度慢，有可能还会失败）
这里选择从网上下载完jdk-8u60-linux-xx.bin之后
将其传到Ubuntu宿主机中，在运行容器的时候使用-v参数将宿主机上的目录映射到容器中，这样在容器中就可以访问到宿主机中的文件了

```
如果提示不能安装.bin文件，使用以下命令即可解决
root@3970c1e5466e:~/.ssh# apt-get update
root@3970c1e5466e:~/.ssh# apt-getinstall g++-multilib
```
#### 安装Zookeeper

```
将下载好的zookeeper-3.4.6.tar.gz上传
root@3970c1e5466e:~/.ssh# mv /software/zookeeper-3.4.6.tar.gz /usr/local/zookeeper-3.4.6
root@3970c1e5466e:~/.ssh# tar -zxvf zookeeper-3.4.6.tar.gz
root@3970c1e5466e:~/.ssh# cd /usr/local/zookeeper-3.4.6/conf/
root@3970c1e5466e:~/.ssh# cp zoo_sample.cfgzoo.cfgvim zoo.cfg
root@3970c1e5466e:~/.ssh# vim zoo.cfg
#修改：dataDir=/root/zookeeper/tmp
#在最后添加：
server.1=cloud4:2888:3888
server.2=cloud5:2888:3888
server.3=cloud6:2888:3888

#保存退出，然后创建一个tmp文件夹
root@3970c1e5466e:~/.ssh# mkdir /data/zookeeper/tmp
#再创建一个空文件
root@3970c1e5466e:~/.ssh# touch /data/zookeeper/tmp/myid
#最后向该文件写入ID
root@3970c1e5466e:~/.ssh# echo 1> /data/zookeeper/tmp/myid
```

#### 安装Hadoop

```
root@3970c1e5466e:~/.ssh# mv /software/hadoop-2.6.1.tar.gz /usr/local/
root@3970c1e5466e:~/.ssh# tar -zxvf hadoop-2.2.0-64bit.tar.gz
root@3970c1e5466e:~/.ssh# cd /usr/local/hadoop/etc/hadoop
```

更改hadoop-env.sh

```
root@3970c1e5466e:/data/test# cd /usr/local/hadoop-2.6.1/
root@3970c1e5466e:/usr/local/hadoop-2.6.1# ls
LICENSE.txt  NOTICE.txt  README.txt  bin  etc  include  lib  libexec  logs  sbin  share
root@3970c1e5466e:/usr/local/hadoop-2.6.1# cd etc/hadoop/
root@3970c1e5466e:/usr/local/hadoop-2.6.1/etc/hadoop# vim hadoop-env.sh
#加入java环境变量
export JAVA_HOME=/usr/local/jdk1.8.0_60
```

修改core-site.xml

```
root@3970c1e5466e:/usr/local/hadoop-2.6.1/etc/hadoop# vim core-site.xml

<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

<property>
<name>fs.defaultFS</name>
<value>hdfs://ns1</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/data/hadoop/tmp</value>
</property>
<property>
<name>ha.zookeeper.quorum</name>
<value>cloud4:2181,cloud5:2182,cloud6:2183</value>
</property>
</configuration>
```

修改hdfs-site.xml, mapred-site.xml, yarn-site.xml

```
root@3970c1e5466e:/usr/local/hadoop-2.6.1/etc/hadoop# vim hdfs-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
<property>
<name>dfs.nameservices</name>
<value>ns1</value>
</property>
<property>
<name>dfs.ha.namenodes.ns1</name>
<value>nn1,nn2</value>
</property>
<property>
<name>dfs.namenode.rpc-address.ns1.nn1</name>
<value>cloud1:9000</value>
</property>
<property>
<name>dfs.namenode.http-address.ns1.nn1</name>
<value>cloud1:50070</value>
</property>
<property>
<name>dfs.namenode.rpc-address.ns1.nn2</name>
<value>cloud2:9000</value>
</property>
<property>
<name>dfs.namenode.http-address.ns1.nn2</name>
<value>cloud2:50070</value>
</property>
<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://cloud4:8485;cloud5:8485;cloud6:8485/ns1</value>
</property>
<property>
<name>dfs.journalnode.edits.dir</name>
<value>/data/hadoop/journal</value>
</property>
<property>
<name>dfs.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
<property>
<name>dfs.client.failover.proxy.provider.ns1</name>
<value>
org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider
</value>
</property>
<property>
<name>dfs.ha.fencing.methods</name>
<value>
sshfence
</value>
</property>
<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>/root/.ssh/id_rsa</value>
</property>
<property>
<name>dfs.ha.fencing.ssh.connect-timeout</name>
<value>30000</value>
</property>
<property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///data/hadoop/workspace/hdfs/name</value>
</property>

<property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///data/hadoop/workspace/hdfs/data</value>
</property>

<property>
       <name>dfs.replication</name>
       <value>2</value>
</property>
</configuration>
```

```
root@3970c1e5466e:/usr/local/hadoop-2.6.1/etc/hadoop# mv mapred-site.xml.template mapred-site.xml
root@3970c1e5466e:/usr/local/hadoop-2.6.1/etc/hadoop# vim mapred-site.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
</configuration>
```

```
root@3970c1e5466e:/usr/local/hadoop-2.6.1/etc/hadoop# vim yarn-site.xml
<?xml version="1.0"?>
<configuration>

<property>
<name>yarn.resourcemanager.hostname</name>
<value>cloud3</value>
</property>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

</configuration>
```

```
root@3970c1e5466e:/usr/local/hadoop-2.6.1/etc/hadoop# vim slaves
cloud1
cloud2
cloud3
cloud4
cloud5
cloud6
```

#### 安装Spark

```
root@3970c1e5466e:~/.ssh# mv /software/scala-2.11.7 /usr/local/
root@3970c1e5466e:~/.ssh# tar -zxvf scala-2.11.7.tgz
root@3970c1e5466e:~/.ssh# vim ~/.bashrc
export JAVA_HOME=/usr/local/jdk1.8.0_60
export HADOOP_HOME=/usr/local/hadoop-2.6.1
export SCALA_HOME=/usr/local/scala-2.11.7
export SPARK_HOME=/usr/local/spark-1.5.1-bin-hadoop2.6
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$SCALA_HOME/bin:$SPARK_HOME/bin

root@3970c1e5466e:~/.ssh# mv /software/spark-1.5.1-bin-hadoop2.6.tgz /usr/local/
root@3970c1e5466e:~/.ssh# tar -zxvf spark-1.5.1-bin-hadoop2.6
```

编辑配置文件

```
root@3970c1e5466e:~/.ssh# cd /usr/local/spark-1.5.1-bin-hadoop2.6/
root@3970c1e5466e:/usr/local/spark-1.5.1-bin-hadoop2.6# cd conf/
root@3970c1e5466e:/usr/local/spark-1.5.1-bin-hadoop2.6# vim slaves
cloud1
cloud2
cloud3
cloud4
cloud5
cloud6
root@3970c1e5466e:/usr/local/spark-1.5.1-bin-hadoop2.6# mv spark-env.sh.template spark-env.sh
root@3970c1e5466e:/usr/local/spark-1.5.1-bin-hadoop2.6# vim ~/spark/conf/spark-env.sh
export SPARK_MASTER_IP=cloud1
export SPARK_WORKER_MEMORY=128m
export JAVA_HOME=/usr/local/jdk1.8.0_60
export SCALA_HOME=/usr/local/scala-2.11.7
export SPARK_HOME=/usr/local/spark-1.5.1-bin-hadoop2.6
export HADOOP_CONF_DIR=/usr/local/hadoop-2.6.1/etc/hadoop
export SPARK_LIBRARY_PATH=$$SPARK_HOME/lib
export SCALA_LIBRARY_PATH=$SPARK_LIBRARY_PATH
export SPARK_WORKER_CORES=1
export SPARK_WORKER_INSTANCES=1
export SPARK_MASTER_PORT=7077
```
此时配置已经基本完成，所以需要__保存镜像__

```
[reason@localhost ~]$ sudo docker commit -m "mofun spark first commit" -a "reason" cloud1 mofun/spark:v1.0
```

查看执行过的镜像

```
[reason@localhost ~]$ sudo docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                         PORTS               NAMES
d4e581ba6af8        mofun/spark:v1.0   "/bin/bash"              11 seconds ago      Exited (0) 7 seconds ago                           hungry_pasteur
```

启动容器

```
# 后台模式运行
[reason@localhost ~]$ docker run -d --name cloud2 -h cloud2 -it mofun/spark:v2.0 /bin/bash

需要使用刚才制作的镜像启动6个容器
[reason@localhost ~]$ docker run --name cloud1 -h cloud1 -it mofun/spark:v2.0 /bin/bash
[reason@localhost ~]$ docker run --name cloud2 -h cloud2 -it mofun/spark:v2.0 /bin/bash
[reason@localhost ~]$ docker run --name cloud3 -h cloud3 -it mofun/spark:v2.0 /bin/bash
[reason@localhost ~]$ docker run --name cloud4 -h cloud4 -it mofun/spark:v2.0 /bin/bash
[reason@localhost ~]$ docker run --name cloud5 -h cloud5 -it mofun/spark:v2.0 /bin/bash
[reason@localhost ~]$ docker run --name cloud6 -h cloud6 -it mofun/spark:v2.0 /bin/bash

```

做最后的修改

```
#在cloud5~cloud6中分别手动修改myid
root@cloud5:~# echo 2 >  /data/zookeeper/myid
root@cloud5:~# echo 2 > /usr/local/zookeeper-3.4.6/tmp/myid
root@cloud6:~# echo 3 >  /data/zookeeper/myid
root@cloud6:~# echo 3 > /usr/local/zookeeper-3.4.6/tmp/myid
```

启动zookeeper集群

```
# 启动zookeeper集群（分别在cloud4、cloud5、cloud6上启动zk）
# 全部节点启动后，再执行zkServer.sh status
# 返回正常
# root@cloud6:/usr/local/zookeeper-3.4.6/bin# ./zkServer.sh status
# JMX enabled by default
# Using config: /usr/local/zookeeper-3.4.6/bin/../conf/zoo.cfg
# Mode: follower
root@cloud5:~# /usr/local/zookeeper-3.4.6/bin/zkServer.sh start
# 当3个节点服务正常启动后
# 使用status查看是否启动
root@cloud5:~# /usr/local/zookeeper-3.4.6/bin/zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
root@cloud5:~#
```

进入cloud1，开启hadoop和spark服务

```
# 启动journalnode（在cloud1上启动所有journalnode，注意：是调用的hadoop-daemons.sh这个脚本，注意是复数s的那个脚本）
# 运行jps命令检验，cloud4、cloud5、cloud6上多了JournalNode进程
root@cloud1:/usr/local/hadoop-2.6.1/sbin# pwd
/usr/local/hadoop-2.6.1/sbin
root@cloud1:/usr/local/hadoop-2.6.1/sbin#
root@cloud1:/usr/local/hadoop-2.6.1/sbin# hadoop-daemons.sh start journalnode

# 格式化ZK(在cloud1上执行即可，在bin目录下)
root@cloud1:/usr/local/hadoop-2.6.1/bin# hdfs zkfc -formatZK

# 进入节点cloud4，查看zookeeper信息
root@cloud4:/usr/local/zookeeper-3.4.6/bin# pwd
/usr/local/zookeeper-3.4.6/bin
root@cloud4:/usr/local/zookeeper-3.4.6/bin# ls
README.txt  zkCleanup.sh  zkCli.cmd  zkCli.sh  zkEnv.cmd  zkEnv.sh  zkServer.cmd  zkServer.sh  zookeeper.out
root@cloud4:/usr/local/zookeeper-3.4.6/bin#
root@cloud4:/usr/local/zookeeper-3.4.6/bin# ./zkCli.sh
Connecting to localhost:2181
2015-10-21 09:34:34,485 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
2015-10-21 09:34:34,487 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=cloud4
2015-10-21 09:34:34,487 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_...
...

[zk: localhost:2181(CONNECTED) 2] ls /hadoop-ha
[ns1]
[zk: localhost:2181(CONNECTED) 3]

# 格式化HDFS(在bin目录下),在cloud1上执行命令:
root@cloud1:/usr/local/hadoop-2.6.1/bin# hdfs namenode -format

# 首先启动active节点，执行如下命令(在cloud1上执行)
root@cloud1:/usr/local/hadoop-2.6.1/sbin# hadoop-daemon.sh start namenode  

# 进入cloud2，需要启动standy模式
root@cloud2:/usr/local/hadoop-2.6.1/bin# pwd
/usr/local/hadoop-2.6.1/bin
root@cloud2:/usr/local/hadoop-2.6.1/bin# ./hdfs namenode -bootstrapStandby
root@cloud2:/usr/local/hadoop-2.6.1/sbin# pwd
/usr/local/hadoop-2.6.1/sbin
# 这条命令可以等到hadoop-daemons.sh start zkfc 成功以后执行
# 貌似是zkfc的启动慢导致standby模式启动出错？

root@cloud1:/usr/local/spark-1.5.1-bin-hadoop2.6/bin# jps
1522 NameNode
2546 Jps
1859 DFSZKFailoverController
1109 Worker
104 JournalNode
426 DataNode
558 NodeManager
943 Master
# 仔细观察DFSZKFailoverController 这个进程的存在情况

root@cloud2:/usr/local/hadoop-2.6.1/sbin# ./hadoop-daemon.sh start namenode

# 重新进入cloud1 ，启动datanode
root@cloud4:/usr/local/hadoop-2.6.1/sbin# pwd
/usr/local/hadoop-2.6.1/sbin
root@cloud4:/usr/local/hadoop-2.6.1/sbin# ./hadoop-daemons.sh start datanode


# 在cloud3上执行start-yarn.sh
root@cloud3:/usr/local/hadoop-2.6.1/sbin# start-yarn.sh

# 启动ZKFC
root@cloud1:/usr/local/hadoop-2.6.1/sbin# ./hadoop-daemons.sh start zkfc
# 启动spark集群
root@cloud1:/usr/local/hadoop-2.6.1/sbin# cd /usr/local/spark-1.5.1-bin-hadoop2.6/sbin/
root@cloud1:/usr/local/spark-1.5.1-bin-hadoop2.6/sbin# start-all.sh

```

此时可以通过CURL访问服务了，如果宿主机中的hosts文件没有配置docker容器的主机名和IP地址映射关系的话要换成用IP访问

```
[reason@localhost ~]$ curl http://172.17.0.56:50070
[reason@localhost ~]$
```

#### 其他

```
# 删除镜像
[reason@localhost ~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
mofun/spark         v2.0                b2dacac3e132        3 hours ago         1.508 GB
mofun/spark         v1.3                4d2f33ca61ee        21 hours ago        1.506 GB
ubuntu              latest              0a17decee413        6 days ago          188.3 MB
docker/whalesay     latest              ded5e192a685        4 months ago        247 MB
[reason@localhost ~]$ sudo docker rmi 4d2f
```

```
# 删除容器
[reason@localhost ~]$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
8258875263be        ubuntu              "/bin/bash"         3 minutes ago       Exited (127) 3 minutes ago                        mad_mestorf
[reason@localhost ~]$ sudo docker rm 8258875263be
8258875263be
[reason@localhost ~]$
```

```
# 删除(NULL) 容器
root@iZ28ikebrg6Z:/var/run# docker images  
REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE  
<none>                 <none>              def2e0b08cbc        About an hour ago   1.37 GB  
sameersbn/redmine      latest              f0bec095f291        2 hours ago         614.6 MB  
root@iZ28ikebrg6Z:/var/run# docker ps -a  
CONTAINER ID        IMAGE                    COMMAND                CREATED             STATUS                     PORTS               NAMES  
5d6373cb79e6        224b40d4b89f             "/bin/sh -c 'apt-get   25 hours ago        Exited (0) 25 hours ago                        distracted_blackwell    
root@iZ28ikebrg6Z:/var/run# docker rm 5d63  
5d63
```

```
# 镜像导出
[reason@localhost ~]$ sudo docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                         PORTS               NAMES
d4e581ba6af8        mofun/spark_rc:v1.0   "/bin/bash"              11 seconds ago      Exited (0) 7 seconds ago                           hungry_pasteur
[reason@localhost ~]$ sudo docker export d4e581ba6af8 > mofunspark_v1.0.tar

# 和导入恢复
[reason@localhost ~]$ cat mofunspark_v1.0.tar | sudo docker import - mofun/spark:v1.0

```

执行Scala交互模式时出错时，需要检查

```
root@cloud1:/usr/local/jdk1.8.0_60/jre/lib/ext# ls -la
total 25632
drwxr-xr-x.  3  501 staff     4096 Oct 19 02:35 .
drwxr-xr-x. 15  501 staff     4096 Oct 18 03:34 ..
-rw-r--r--.  1  501 staff  3860522 Aug  4 19:29 cldrdata.jar
-rw-r--r--.  1  501 staff     8286 Aug  4 19:29 dnsns.jar
-rw-r--r--.  1  501 staff    44516 Aug  4 19:29 jaccess.jar
-rwxr-xr-x.  1  501 staff 18464934 Aug  3 17:58 jfxrt.jar
-rw-r--r--.  1  501 staff  1178935 Aug  4 19:29 localedata.jar
-rw-r--r--.  1  501 staff     1269 Aug  4 19:29 meta-index
-rw-r--r--.  1  501 staff  2014239 Aug  4 19:29 nashorn.jar
-rw-r--r--.  1  501 staff    39771 Aug  4 19:29 sunec.jar
-rw-r--r--.  1  501 staff   278680 Aug  4 19:29 sunjce_provider.jar
-rw-r--r--.  1  501 staff   250826 Aug  4 19:29 sunpkcs11.jar
drwxr-xr-x.  2 root root      4096 Oct 19 02:35 tmp
-rw-r--r--.  1  501 staff    68848 Aug  4 19:29 zipfs.jar
root@cloud1:/usr/local/jdk1.8.0_60/jre/lib/ext#
# 该目录下出现了很多类似._jfxrt.jar 的包，直接予以删除即可。
```

#### 参考文献
  * [http://eksliang.iteye.com/blog/2226986](http://eksliang.iteye.com/blog/2226986)
  * [http://dockerpool.com/static/books/docker_practice/container/daemon.html](http://dockerpool.com/static/books/docker_practice/container/daemon.html)
  * [http://dockerpool.com/static/books/docker_practice/install/ubuntu.html](http://dockerpool.com/static/books/docker_practice/install/ubuntu.html)
  * [http://dockerpool.com/static/books/docker_practice/image/create.html](http://dockerpool.com/static/books/docker_practice/image/create.html)
  * [http://dockerpool.com/static/books/docker_practice/image/save_load.html](http://dockerpool.com/static/books/docker_practice/image/save_load.html)
  * [http://dockerpool.com/static/books/docker_practice/container/rm.html](http://dockerpool.com/static/books/docker_practice/container/rm.html)
  * [http://blog.csdn.net/qq1010885678/article/details/46353101](http://blog.csdn.net/qq1010885678/article/details/46353101)
  * [http://cn.soulmachine.me/blog/20131027/](http://cn.soulmachine.me/blog/20131027/)
  * [http://my.oschina.net/zjzhai/blog/225112](http://my.oschina.net/zjzhai/blog/225112)
  * [http://blog.csdn.net/minimicall/article/details/40188251](http://blog.csdn.net/minimicall/article/details/40188251)
  * [http://www.scala-lang.org/documentation/](http://www.scala-lang.org/documentation/)  
  * [https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/NetworkWordCount.scala](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/NetworkWordCount.scala)
  * [http://spark.apache.org/docs/latest/](http://spark.apache.org/docs/latest/)
  * [https://docs.sigmoidanalytics.com/index.php/Error:_Failed_to_initialize_compiler:_object_scala_not_found.](https://docs.sigmoidanalytics.com/index.php/Error:_Failed_to_initialize_compiler:_object_scala_not_found.)
  * [http://docs.docker.com/linux/step_one/](http://docs.docker.com/linux/step_one/)
  * [http://blog.sequenceiq.com/blog/2015/01/09/spark-1-2-0-docker/](http://blog.sequenceiq.com/blog/2015/01/09/spark-1-2-0-docker/)
