---
title: Ubuntu 17.10配置Hadoop+Spark环境
date: 2018-04-08 14:39:34
categories:
- 分布式与大数据
tags:
- Ubuntu 17.10 hadoop Spark 
---

## 一、前言
&nbsp;&nbsp;最近导师带的项目是与大数据相关，感觉这几年大数据技术还挺火的，就想着也去学一下，丰富自己的技能栈。本文主要讲的是hadoop+spark的环境搭建,然后使用自带的examples测试环境，这里不涉及原理介绍。<!--more-->


## 二、Hadoop的三种运行模式介绍
### 2.1、 单机模式也叫独立模式（Local或Standalone Mode）
+ 默认情况下，Hadoop即处于该模式，用于开发和调式。
+ 不对配置文件进行修改。
+ 使用本地文件系统，而不是分布式文件系统。
+ Hadoop不会启动NameNode、DataNode、JobTracker、TaskTracker等守护进程，Map()和Reduce()任务作为同一个进程的不同部分来执行的。
+ 用于对MapReduce程序的逻辑进行调试，确保程序的正确。


### 2.2、 伪分布式模式（Pseudo-Distrubuted Mode）
+ Hadoop的守护进程运行在本机机器上，模拟一个小规模的集群
+ 在一台主机上模拟多主机。
+ Hadoop启动NameNode、DataNode、JobTracker、TaskTracker这些守护进程都在同一台机器上运行，是相互独立的Java进程。
+ 在这种模式下，Hadoop使用的是分布式文件系统，各个作业也是由JobTraker服务，来管理的独立进程。在单机模式之上增加了代码调试功能，允许检查内存使用情况，HDFS输入输出，以及其他的守护进程交互。类似于完全分布式模式，因此，这种模式常用来开发测试Hadoop程序的执行是否正确。
+ 修改3个配置文件：core-site.xml（Hadoop集群的特性，作用于全部进程及客户端）、hdfs-site.xml（配置HDFS集群的工作属性）、mapred-site.xml（配置MapReduce集群的属性）
+ 格式化文件系统


### 2.3、 全分布式集群模式（Full-Distributed Mode）
+ Hadoop的守护进程运行在一个集群上　
+ Hadoop的守护进程运行在由多台主机搭建的集群上，是真正的生产环境。
+ 在所有的主机上安装JDK和Hadoop，组成相互连通的网络。
+ 在主机间设置SSH免密码登录，把各从节点生成的公钥添加到主节点的信任列表。
+ 修改3个配置文件：core-site.xml、hdfs-site.xml、mapred-site.xml，指定NameNode和JobTraker的位置和端口，设置文件的副本等参数
+ 格式化文件系统。



## 三、搭建伪分布式集群的前提条件
### 3.1、 实验环境
- [x]  **ubuntu 17.10-x64**
- [x]  **jdk_1.8.0_162**
- [x]  **Hadoop-3.0.0**
- [x]  **Spark-2.3.0**


### 3.2、 安装JDK，并配置环境变量
&nbsp;&nbsp;首先去官网下载对应系统版本的jdk,然后解压到`opt`目录下，命令如下：
>     sudo tar -zxvf jdk-8u162-linux-x64.tar.gz -C /opt


然后切换到`/opt`目录下，修改jdk的文件夹命名
>     mv jdk-8u162-linux-x64 ./jdk

最后，配置环境变量，
>     vim /etc/profile

在配置文件中加入：

```shell
export JAVA_HOME=/opt/jdk
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin
```
使配置文件生效，执行下面的命令：

    source /etc/profile
最后查看是否安装成功

    java -version


### 3.3、 安装Scala
+ 官网下载scala(scala-2.12.4.tgz)
+ 解压下载scala包
```
sudo tar -xzvf scala-2.12.4.tgz -C /opt/
# 修改文件名：
sudo mv scala-2.12.4 ./scala
```
+ 添加环境变量
```
sudo vim /etc/profile
# 在后面添加下面内容
export SCALA_HOME=/opt/scala
export PATH=$SCALA_HOME/bin:$PATH
```
+ 使配置生效并测试
```
source /etc/profile
# 测试是否安装成功
scala -version  # 输出scala版本号
```


### 3.4、 安装ssh并设置免密登录
- 安装ssh。如果已安装则跳过这一步。
```
sudo apt-get install openssh-server
```
- 配置ssh无密登录
```
ssh-keygen -t rsa # 然后一直回车
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
- 测试ssh无密登录
```
ssh localhost
# 如果不提示输入密码则配置成功
```

[TOC]

## 四、搭建伪分布式集群

### 4.1、 hadoop下载安装
* 下载Hadoop(笔者下载的是hadoop-3.0.0.tar.gz)
* 解压并重命名
```
# 解压
sudo tar -xzvf hadoop-3.0.0.tar.gz -C /opt/
# 重命名
cd /opt
sudo hadoop-3.0.0 hadoop
```
* 修改文件权限
```
cd /opt
sudo chown -R yourname:yourname hadoop  
# yourname替换成你的用户名  -R表示逐级往下授权
```
* 配置环境变量
```
sudo vim /etc/profile
# 在最后添加下面代码
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_COMMON_LIB_NATIVE_DIR"
```
* 使配置生效
```
source /etc/profile
```
* 测试
```
hadoop version  # output the information of the version of the hadoop
```


### 4.2、 Hadoop伪分布式配置
+ 修改配置文件`hadoop-env.sh`
```
# 切换到工作目录
cd /opt/hadoop/etc/hadoop
# 打开配置文件
vim hadoop-env.sh
# 直接加上下面代码
export JAVA_HOME=/opt/jdk
```
+ 修改配置文件`core-site.xml`
```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://219.223.243.131:9000</value>
</property>
```
>    219.223.243.131是我的节点所在主机的ip,9000为默认的端口，不用更改

+ 修改配置文件`hdfs-site.xml`
```
<property>
    <name>dfs.nameservices</name>
    <value>hadoop-cluster</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///data/hadoop/hdfs/nn</value>
</property>
<property>
    <name>dfs.namenode.checkpoint.dir</name>
    <value>file:///data/hadoop/hdfs/snn</value>
</property>
<property>
    <name>dfs.namenode.checkpoint.edits.dir</name>
    <value>file:///data/hadoop/hdfs/snn</value>
</property>
<property>
    <name>fs.datanode.data.dir</name>
    <value>file:///data/hadoop/hdfs/dn</value>
</property>

```

+ 修改配置文件`mapred-site.xml`
```
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
<property>
    <name>mapreduce.admin.user.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
    <name>mapreduce.application.classpath</name>
    <value>
         /opt/hadoop/etc/hadoop,
         /opt/hadoop/share/hadoop/common/*,
         /opt/hadoop/share/hadoop/common/lib/*,
         /opt/hadoop/share/hadoop/hdfs/*,
         /opt/hadoop/share/hadoop/hdfs/lib/*,
         /opt/hadoop/share/hadoop/mapreduce/*,
         /opt/hadoop/share/hadoop/mapreduce/lib/*,
         /opt/hadoop/share/hadoop/yarn/*,
         /opt/hadoop/share/hadoop/yarn/lib/*
    </value>
</property>
```
+ 修改配置文件`yarn-site.xml`
```
<!-- 指定ResourceManager的地址-->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>219.223.243.131</value>
</property>
<!-- 指定reducer获取数据的方式-->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.local-dirs</name>
    <value>file:///data/hadoop/yarn/nm</value>
</property>

<property>
    <name>yarn.nodemanager.vmem-pmem-ratio</name>
    <value>4</value>
</property>
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
```
+ 创建相关目录
```
sudo mkdir -p /data/hadoop/hdfs/nn
sudo mkdir -p /data/hadoop/hdfs/dn
sudo mkdir -p /data/hadoop/hdfs/snn
sudo mkdir -p /data/hadoop/yarn/nm
# 然后给这些目录设置读写权限，
# 如果使用当前的用户（非root用户）启动相关进程，`/data`必须具有相应的读写权限
# 给`/data`目录及其子目录设置读写权限。 -R 递归设置权限
sudo chmod -R 777 /data
```
+ 对HDFS集群进行格式化，HDFS集群是用来存储数据的
```
hdfs namenode -format
```


### 4.3、 启动集群
1. 启动HDFS集群
```
# 启动主节点
hdfs --daemon start namenode 
# 启动从节点
hdfs --daemon start datanode 

# 验证节点是否启动，输入一下命令
jps # 看是否出现namenode和datanode
```
2. 启动YARN集群
```
# 启动资源管理器
yarn --daemon start resourcemanager
# 启动节点管理器
yarn --daemon start nodemanager

# 验证是否启动，同样是采用`jps`命令，看是否出想相关进程
```
3. 启动作业历史服务器
```
mapred --daemon start historyserver
# 同样采用`jps`命令查看是否启动成功
```
4. HDFS和YARN集群都有对应的WEB监控页面。
>  HDFS: http://ip:9870 或者 localhost:9870


<img src="https://wx2.sinaimg.cn/mw1024/e0db46edgy1fq6ro1zxiuj20so0pvq5f.jpg" alt="图片名称" align=center />

<img src="https://wx2.sinaimg.cn/mw1024/e0db46edgy1fq6ro1u9z0j20sh0ntgmn.jpg"  alt="图片名称" align=center />

>  YARN: http://ip:8088

<img src="https://wx4.sinaimg.cn/mw1024/e0db46edgy1fq6ro1zitoj21ag0gstbh.jpg"  alt="图片名称" align=center />

5. HDFS集群的简单操作命令
```
hdfs dfs -ls /   # 相当于shell中的 ll
hdfs dfsadmin -safemode leave  # 关闭安全模式
hdfs dfs -mkdir -p /user/input  # 在hdfs文件系统上级联创建文件/user/input，
```
6. YARN集群examples测试
```
# 计算PI值的作业
yarn jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 4 100

# wordcount例子
# 首先创建一个输入文件，并上传到hdfs文件系统上
hdfs dfs -put input /user/heany
# 然后执行
yarn jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount input output
# 用命令行查看结果
hdfs dfs -cat /user/output/part-r-00000
# 或者在网页上查看
```



## 五、 安装Spark
### 5.1、 下载安装Spark
+ 下载Spark(笔者下载的是spark-2.3.0-bin-hadoop2.7.tgz)
+ 解压下载的Spark包
```
sudo tar -zxvf spark-2.3.0-bin-hadoop2.7.tgz -C /opt
```
+ 重命名
```
# 切换到工作目录
cd /opt
sudo mv spark-2.3.0-bin-hadoop2.7 spark
```
+ 添加环境变量
```
sudo vim /etc/profile
# 在最后添加下面的代码
export SPARK_HOME=/opt/spark
export PATH=$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH
```
+ 修改文件的权限
```
cd /opt
sudo chown -R yourname:yourname ./spark
```

### 5.2、 修改配置文件
```
# 拷贝配置文件
cd /opt/spark
cp ./conf/spark-env.sh.template ./conf/spark-env.sh

# 修改配置文件
vim ./conf/spark-env.sh
# 在最后添加下面的代码
export SPARK_DIST_CLASSPATH=$(/opt/hadoop/bin/hadoop classpath)
export JAVA_HOME=/opt/jdk

```

### 5.3、运行examples验证安装

```
/opt/spark/bin/run-example SparkPi 2>&1 | grep "Pi is roughly"
# output
# Pi is roughly 3.143635718178591
```

### 5.4、 脚本启动Hadoop和Spark
- 启动spark
```
/opt/spark/sbin/start-all.sh
```
- 通过WEB页面查看
>  浏览器输入地址：http://localhost:8080

- 编写自动化脚本
```
# 启动Hadoop以及Spark脚本

#!/bin/bash
# 启动Hadoop以及yarn
start-dfs.sh
start-yarn.sh
# 启动历史服务器
#mr-jobhistory-daemon.sh start historyserver
mapred --daemon start historyserver
# 启动Spark
/opt/spark/sbin/start-all.sh



# 停止Hadoop以及Spark脚本

#!/bin/bash
# 停止Hadoop以及yarn
stop-dfs.sh
stop-yarn.sh
# 停止历史服务器
#mr-jobhistory-daemon.sh stop historyserver
mapred --daemon stop historyserver
# 停止Spark
/opt/spark/sbin/stop-all.sh
```