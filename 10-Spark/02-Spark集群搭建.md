**集群规划**

| master | node1       |
| ------ | ----------- |
| worker | node2,node3 |

## 一、安装(node1)

1.解压

tar -zxvf spark-2.3.1-bin-hadoop2.6.tgz -C /opt 

2.重命名

mv spark-2.3.1-bin-hadoop2.6/ spark-2.3.1  

## 二、配置文件

进入conf目录

**1.配置worker节点** 

拷贝一份出来

cp slaves.template slaves 

修改slaves文件

node2 node3 

**2.Spark相关配置**

拷贝一份出来

cp spark-env.sh.template spark-env.sh 

修改spark-env.sh文件

export SPARK_MASTER_HOST=node1 export SPARK_MASTER_PORT=7077 export SPARK_WORKER_CORES=2 export SPARK_WORKER_MEMORY=2g export JAVA_HOME=/usr/java/jdk1.8.0_221 

## 三、分发配置

scp -r spark-2.3.1 node2:`pwd` scp -r spark-2.3.1 node3:`pwd` 

## 四、启动集群

进入Spark目录sbin目录下

./start-all.sh 

## 五、访问WebUi

默认是8080端口，可以在spark-env.sh通过SPARK_MASTER_WEBUI_PORT修改

node1:8080 

## 六、提交任务

**1.基于standalone-client提交任务**

Spark集群任意节点或者单独的Spark的客户端

Standalone提交命令：

./spark-submit --master spark://node1:7077  --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 100 

**2.基于yarn提交任务**

在Spark客户端加如下配置

spark-env.sh

export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop 

可能会报错虚拟内存不足。同时这里需要在每台NodeManager节点中将每台NodeManager的虚拟内存关闭，在每台NodeManager节点的$HADOOP_HOME/etc/hadoop/yarn-site.xml中加入如下配置：

<!-- 关闭虚拟内存检查 --> <property>    <name>yarn.nodemanager.vmem-check-enabled</name>    <value>false</value> </property> 

Spark on YARN提交命令：

./spark-submit --master yarn --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 100 

## 七、如何动态添加Spark的worker节点(node4)

1.将spark集群中任意一台配置好的拷贝发送到node4

```shell
scp -r spark-2.3.1/ node4:`pwd` 
```

2.在新的节点启动worker进程，命令在sbin目录下

```shell
./start-slave.sh spark://node1:7077 
```