**一、SparkStreaming简介**

SparkStreaming是流式处理框架，是Spark api的扩展。支持可扩展、高吞吐、容错的实时数据流处理。

**数据源**：Kafka、Flume、Twitter、ZeroMQ或者Tcp Socket。

**处理后的数据**：可以存放在文件系统，数据库等，方便实时展现。

**二、SparkStreaming与Storm的区别**

1.Storm是纯实时的流式处理框架，SparkStreaming是准实时的流式处理框架（微批处理）。因为是微批处理，SparkStreaming的吞吐量比Storm要高。

2.Storm的事务机制比SparkStreaming完善。

3.Storm支持动态资源调度（Spark1.2之后也开始支持）。

4.Storm不擅长复杂的业务处理，擅长简单的汇总型处理。SparkStream擅长复杂的业务处理。

**三、SparkStreaming处理数据流程**

![img](E:\Download\YoudaoNote\yangyh11@163.com\7ebc67f6e9a44568a111d8e4f20ac54a\clipboard.png)

这里演示的数据源是socket。receiver模式

SparkStreaming是7*24小时不间断运行的。程序启动之后，首先会启动一个job接收数据，receiver task一直处于接收数据的状态，将一段时间的数据接收到batch中，batch被封装到一个RDD中，RDD又被封装到DStream中。这里一段时间叫做batchInternal。SparkStreaming在batchInternal内接收到数据后，会启动新的job进行处理。DStream有Transformation算子，懒执行，需要DStream的outputOperation类算子触发执行。

假设batchInternal是5s，集群处理一批数据的时间是3s：

集群启动，0-5s集群接收数据，5-8s集群一边接收数据一边处理数据，8-10s只是接收数据。。。

问题：集群资源不能充分利用，资源浪费。

假设batchInternal是5s，集群处理一批数据的时间是8s：

集群启动，0-5s集群接收数据，5-10s集群一边接收数据一边处理数据，10-13s集群一边接收数据一边处理上一批的数据。。。

问题：集群接收的数据有堆积，如果数据存储在内存就会有OOM问题，如果放磁盘，加大数据处理的延迟。

**四、代码**

客户端，数据源

\# 启动socket nc –lk 9999 

SparkStreaming集群

def main(args: Array[String]): Unit = {   val conf = new SparkConf()  // receive模式下，local的模拟线程必须等于大于等于2，一个线程用来receive接收数据，一个线程用来处理数据  conf.setMaster("local[2]")  conf.setAppName("SparkStreaming demo1")   // batchDuration：流数据被分成批次的时间间隔。每隔5s就将这5s时间接受到的数据保存到一个batch中。  val ssc: StreamingContext = new StreamingContext(conf, Durations.seconds(5))  val lines: ReceiverInputDStream[String] = ssc.socketTextStream("node4", 9999)   val words: DStream[String] = lines.flatMap(_.split(" "))   val pairWord: DStream[(String, Int)] = words.map((_, 1))   val result: DStream[(String, Int)] = pairWord.reduceByKey(_ + _)  // DStream的outputoperator类算子触发  result.print()   ssc.start()  // 等待Spark程序被终止  ssc.awaitTermination()  ssc.stop() } 

1.StreamingContext .start()方法之后不能再添加业务逻辑。

2.StreamContext.stop()无参的stop方法将SparkContext一同关闭，stop(false)不会关闭SparkContext，。