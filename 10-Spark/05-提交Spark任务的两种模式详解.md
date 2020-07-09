**一、基于standalone提交任务**

**1.基于standalone-client提交任务**

--deploy-mode：不写，默认就是client提交

也可以配置：--deploy-mode client

./spark-submit --master spark://node1:7077  --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 100 

![img](E:\Download\YoudaoNote\yangyh11@163.com\3c56e8219a9b429998a240e51433dea0\spark基于standalone-client模式提交任务流程.jpg)

**流程**

1.Worker节点向Master节点汇报资源。

2.在客户端提交Spark Application时，Driver在客户端启动。

3.客户端向Master申请资源。

4.Master通知Worker启动Executor。

5.

**2.基于standalone-cluster提交任务**

注意：在cluster模式下，jar包需要放到集群可以访问的目录，如果是放在本地，需要在每个节点上都放置一份

./spark-submit --master spark://node1:7077 --deploy-mode cluster --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 100 

![img](E:\Download\YoudaoNote\yangyh11@163.com\b7bd3658a33a4818a8a752e40ad624c2\spark基于standalone-cluster模式提交任务流程.jpg)

**基于yarn提交任务**

在Spark客户端加如下配置

spark-env.sh

export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop 

可能会报错虚拟内存不足.同时这里需要在每台NodeManager节点中将每台NodeManager的虚拟内存关闭，在每台NodeManager节点的$HADOOP_HOME/etc/hadoop/yarn-site.xml中加入如下配置：

<!-- 关闭虚拟内存检查 --> <property>    <name>yarn.nodemanager.vmem-check-enabled</name>    <value>false</value> </property> 

 

**1.yarn-client提交任务方式**

./spark-submit --master yarn  --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 100 

![img](E:\Download\YoudaoNote\yangyh11@163.com\94669cae3ced48ad8b6fb6b8f5f3bb66\spark基于yarn-client模式提交任务流程.jpg)

**2.yarn-cluster提交任务方式**

./spark-submit --master yarn --deploy-mode cluster --class org.apache.spark.examples.SparkPi ../examples/jars/spark-examples_2.11-2.3.1.jar 100 

![img](E:\Download\YoudaoNote\yangyh11@163.com\f108f2a4a7bd447499317b40906ecc8b\spark基于yarn-cluster模式提交任务流程.jpg)