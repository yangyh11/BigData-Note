**一、Master高可用的原理**

Standalone集群只有一个Master，如果Master挂了就无法提交应用程序，需要给Master进行高可用配置。

Master的高可用可以使用fileSystem(文件系统)和Zookeeper（分布式协调服务）。

1.fileSyatem只有存储功能，可以存储Master的元数据信息，用fileSyatem搭建的Master HA，在Master失败时，需要我们手动启动另外的备用的Master，这种方式不推荐使用。

2.zookeeper由选举和存储功能，可以存储Master的元数据信息，使用zookeeper搭建的Master HA，当master挂掉时，备用的Master会自动切换，推荐使用这种方式搭建Master的HA。

**二、使用Zookeeper搭建Master HA**

**1.在Spark配置文件spark-env.sh增加如下配置（****所有节点****）**

export SPARK_DAEMON_JAVA_OPTS=" -Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=node2:2181,node3:2181,node4:2181  -Dspark.deploy.zookeeper.dir=/sparkmaster" 

**2.备用Master，找一台节点（非Master节点）修改Master（****node2****）**

export SPARK_MASTER_HOST=node2 

**三、启动**

启动zookeeper集群

zkServer.sh start 

启动Spark集群

./start-all.sh 

启动备用Master节点（node2）

./start-master.sh 

主Master

![img](E:\Download\YoudaoNote\yangyh11@163.com\ad3e8246391f406eb16a735f66b8535e\clipboard.png)

备用Master

![img](E:\Download\YoudaoNote\yangyh11@163.com\0e551a3c1dd24764a99f0a1888b18c60\clipboard.png)