---
title: spark tips
date: 2019-09-01 22:18:19
tags:
    - spark
categories:
    - Distributed System
    - Bigdata Technology
---

# spark tips

### jar包上传到hdfs

* 分布式集群上跑spark时每次都要将${SPARK_HOME}/jars/*下所有的jar包上传到分布式系统中供调用，为了避免每次的上传时间可以将这些jar包存放到hdfs中，==注意：我在2.\*的版本中将所有jar包打包成.archive文件上传到hdfs，但是这样做导致spark在yarn上启动失败，后来尝试将这些jar包分散上传到hdfs的文件夹下就能正常在yarn上启动spark了。==具体做法如下：

```shell
将${SPARK_HOME}/jars下的所有jar包上传到hdfs中：
hadoop fs -put jars/* /spark
在${SPARK_HOME}/conf/spark-defaults.conf中添加配置项：
spark.yarn.jars  hdfs://localhost:9000/spark/jars/*
```

* 如果要使用hbase则需要将hbase的lib目录下hbase开头的jar包和guava-{version}.jar、protobuf-java-{version}.jar上传到hdfs上的同一目录jars，hbase.xml复制一份到hadoop的配置文件目录etc/hadoop下

## spark on yarn

### 1. 调参

#### 1.1 client mode

* client mode下，spark的driver与applicationmaster是分离的，要分别设置内存大小。任务提交的同时driver就开始运行了，所以driver内存不可以在程序里使用SparkConf类设置，可通过spark-defaults.conf或者CLI下--driver-memory配置，而applicationmaster的内存大小没有限制可以在SparkConf里面通过spark.yarn.am.memory来配置

#### 1.2 cluster mode

* cluster mode下，spark的driver和applicationmaster在同一个container里面，可以通过在SparkConf，spark-defaults.conf或者CLI下用spark.yarn.am.memory来对这个container内存大小进行配置

#### 1.3 CLI参数设置

> spark在yarn上运行能使用的container数目是由核数vcores和executor内存大小共同决定的，由于hadoop不像spark，hadoop可以设置一个结点的总内存和核数，所以当cpu或者内存资源超过设置的额度时会限制container到尽可能大的数目

* --num-executors设置executor数目
* --executor-memory设置executor内存
* --executor-cores设置executor核数
* --conf spark.yarn.am.memory=2g设置application master内存大小
* --conf spark.yarn.am.cores=2设置application master核数

#### 1.4 运行方式

* cluster

```shell
spark-submit --class <yourclass> --master yarn --deploy-mode cluster --conf spark.yarn.am.memory=2g --executor-memory 2g --executor-cores 2 <yourjar> <args>
```

* client

```shell
spark-submit --class <yourclass> --master yarn --deploy-mode client --driver-meemoty 2g --executor-memory 2g --executor-cores 2 <yourjar> <args>
```





## spark standalone

### 1.调参

* spark无需像mapreduce那样给每个节点设置总的内存限制，spark总内存使用会自动扩展。

* spark集群总核数无需配置，自动识别集群中所有机器的虚拟内核数

* spark的executor数目由集群中的总核数和exexutor的核数决定：
  $$
  num-executors = cores-in-cluster/executor.cores
  $$

* executor.memory不等于RDD可用的memory size，因为JVM等也需要占据一定内存资源，差不多只能用一半，driver.memory也一样

* 运行方式：

  ```shell
  spark-submit --class <yourclass> --master spark://slave103:7077 --conf spark.driver.memery=6g --conf spark.executor.memory=2g --conf spark.executor.cores=2 <yourjar> <args>
  ```