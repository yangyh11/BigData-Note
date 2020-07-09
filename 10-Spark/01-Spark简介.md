# Spark简介

Spark是专为大规模数据处理而设计的快速通用的计算引擎。

Spark拥有MapReduce都具有的优点；但不同的是Spark的job中间处理结果可以保存在内存中，从而不再需要读取HDFS。

Spark能更好的适用与数据挖掘与机器学习等需要迭代的算法。

## 一、Spark特点

1.速度快

2.简单易用

3.通用

4.无处不运行 -- Yarn | Mesos | Standlone | k8s

## 二、Spark技术栈

相关：HDFS， yarn， hive， storm

SparkCore， SparkSQL， SparkStreaming

## 三、Spark与MapReduce的区别

都是分布式计算框架，Spark基于内存，MR基于HDFS。

Spark：

1.Spark基于内存计算。

2.Spark有DAG(有向无环图)执行引擎，来切分任务的执行先后顺序。

## 四、Spark运行模式

1.Local

多用于本地测试，如再Eclipse，Idea中写程序测试等。

2.Standalone

Standalone是是Spark自带的一个资源调度框架，它支持完全分布式。

3.Yarn

Hadoop生态圈里面的一个资源调度框架，Spark也是可以基于Yarn来计算的。

4.Mesos

Apache的资源调度框架。

注意：要基于Yarn来进行资源调度，必须实现ApplicationMaster接口，Spark实现了这个接口，所以可以基于Yarn。

## 五、术语解释

1.Master

资源管理的主节点（进程） 

2.Worker

资源管理的从节点（进程） 

3.Application

基于Spark的用户程序，包含了Driver程序和运行在集群上的Executor程序 

4.Driver

用来连接Worker进程的程序 

5.Executor

是在一个Worker进程所管理的节点上为Application启动的一个进程。该进程负责运行任务，并且负责将数据存在内存或者磁盘上， 每一Application都有各自独立的executor 

6.Job

包含很多Task的并行计算。可以看作和Action算子对应，一个Application中有多少个Action算子，就有多少个job 

7.Stage

一个Job会被拆分很多组任务，每组任务被称为Stage。Stage切割规则，从后往前，遇到宽依赖就切割。 

8.Task

分发到某个Executor上的工作单元 