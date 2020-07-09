**一、SparkStreaming1.6 + kafka（Receiver模式）**

**二、SparkStreaming1.6 + kafka（Direct模式）**

**三、SparkStreaming 2.3 + kafka（Direct模式）**

1.丢弃了SparkStreaming + kafka的receiver模式，只有Direct模式。

2.采用了kafka新的consumer api实现，类似于1.6中SparkStreaming读取kafka点的Direct模式。并行度一样。

3.因为采用了kafka新的consumer api实现，所以相对于1.6中SparkStreaming的Direct模式（simple api实现），在api使用上有很大不同。未来这种api有可能继续变化（spark官网申明，推锅kafka）。

4.kafka中有两个参数：

session.timeout.ms：(默认30s)表示消费者与kafka之间的session会话的超时时间，如果在这个时间内，集群没有接受到消费者的心跳，那么就移除消费者。这个值位于group.min.session.timeout.ms(默认5s)和group.max.session.timeout.ms(默认300s)。

heartbeat.interval.ms：(默认3s)这个值代表kafka集群与消费者之间的心跳间隔时间，kafka集群确保消费者保持连接的心跳通信时间间隔。这个值必须比session.timeout.ms小，一般设置不会大于session.timeout.ms的1/3。

5.大多数情况下，SparkStreaming读取数据使用策略：

 PreferConsistent：大多数采用这种。会将分区均匀的分布在集群的Executor之间

 PreferBrokers：如果Executor在kafka集群中的某些节点上，可以使用这种策略。那么当前这个Executors中的数据会来自当前broker节点。

 PreferFixed：如果节点之间的分区有明显的分布不均，可以使用这种策略。可以通过一个map指定将topic分区分布在哪些节点中。

6.新的消费者api可以将kafka中的消息预读到缓存区中，默认大小是64k。默认缓存在Executor中，加快处理数据速度。

 spark.streaming.kafka.consumer.cache.maxCapacity来增带缓存大小。

 spark.streaming.kafka.consume.cache.enable设置成false关闭缓存机制。

注意：官网建议关闭，可能会有重复消费的问题。

7.关于消费者offset

 1).如果设置了checkpoint。那么offset将会存储在checkpoint中。可以利用checkpoint恢复offset，getOrCreate方法获取。

   存在的问题：1.当从checkpoint中恢复数据时，有可能会造成重复的消费。

​            		 2.当代码逻辑修改，无法从checkpoint中来恢复offset。

 2).依靠kafka来存储消费者offset。kafka中有一个特殊的topic来存储消费者offset。新的consumer api中，默认会定期自动提交offset。这也不是我们想要的。因为有可能消费者自动提交了offset，而SparkStreaming还没有将数据处理保存。可能会存在有数据漏掉没有消费处理的情况，所以将enable.auto.commit自动提交（默认5s）设置为false。我们自己手动异步提交，保证处理完业务后提交offset到kafka。

注意：这种模式也有弊端，这种将offset存储在kafka中的方式，参数offset.retention.minutes（默认是1440分钟，保存一天）控制offset过期删除的时间。如果在指定时间没有被消费掉，存在在kafka中offset会被清楚，存在消息漏消费的情况。

 3).自己存储offset。这样在处理逻辑时，保证数据处理的事务，如果处理数据失败，就不保存offset，处理数据成功则保存offset，这样就可以做到精准的处理一次处理数据。

SparkStreaming2.3 +kafka0.11代码：

**1.手动异步提交offset到kafka，offset由kafka管理**

def main(args: Array[String]): Unit = {  val conf = new SparkConf()  conf.setMaster("local")  conf.setAppName("SparkStreamingOnKafkaDirect")  val ssc = new StreamingContext(conf, Durations.seconds(5))  //设置日志级别  ssc.sparkContext.setLogLevel("ERROR")   /**   * auto.offset.reset的参数：   * earliest：当各分区下已有提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始   * latest：自动重置偏移量为最大的偏移量【默认】   * node：没有找到以前的offset，抛出异常   */  val kafkaParams = Map[String, Object](    ("bootstrap.servers", "node1:9092,node2:9092,node3:9093"),    ("key.deserializer", classOf[StringDeserializer]),    ("value.deserializer", classOf[StringDeserializer]),    ("group.id", "MyGroup1"),    ("auto.offset.reset", "earliest"),    // 为true，每隔一段时间就自动将offset提交。设置成false，我们手动去提交offset    ("enable.auto.commit", false: java.lang.Boolean) // 默认是true  )   // 可以读多个topic的消息  val topic = Array[String]("topic1220")  // 消费者读取topic消息  val stream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](    ssc,    PreferConsistent, // 消费策略    Subscribe[String, String](topic, kafkaParams)  )   val transStream: DStream[String] = stream.map(record => {    val key_value = (record.key(), record.value())    println("receive message key = " + key_value._1)    println("receive message value = " + key_value._2)    key_value._2  })   val wordsStream: DStream[String] = transStream.flatMap(_.split("\t"))  val result: DStream[(String, Int)] = wordsStream.map((_, 1)).reduceByKey(_ + _)  result.print()   /**   * 以上的业务完成后，异步的提交消费者offset。这里将enable.auto.commit设置成false，就是使用kafka自己来管理消费者offset。   * 注意：获取offsetRanges：Array[OffsetRange]每一批次的topic中的offset时，必须从源头读取过来的stream中获取，不能从经过   * 转换后的transStream中获取。   */  stream.foreachRDD(rdd => {    val offsetRanges: Array[OffsetRange] = rdd.asInstanceOf[HasOffsetRanges].offsetRanges     offsetRanges.foreach(offset => {      println(s"current topic = ${offset.topic},partition = ${offset.partition}," +        s"fromoffset = ${offset.fromOffset},untiloffset=${offset.untilOffset}")    })     // 异步提交，更新offset，kafka管理    stream.asInstanceOf[CanCommitOffsets].commitAsync(offsetRanges)  })   ssc.start()  ssc.awaitTermination()  ssc.stop() } 

**2.手动维护,toffset自己存起来,如使用redis**

def main(args: Array[String]): Unit = {  val conf = new SparkConf()  conf.setMaster("local")  conf.setAppName("ManageOffsetUseRedis")  //设置每个分区每秒读取多少条数据  conf.set("spark.streaming.kafka.maxRatePerPartition","10")   val ssc = new StreamingContext(conf,Durations.seconds(5))  //设置日志级别  ssc.sparkContext.setLogLevel("Error")   val topic = "myTopic"   /**    * 1.从Redis中获取消费者offset    */  val dbIndex = 3  val currentTopicOffset: mutable.Map[String, String] = getOffSetFromRedis(dbIndex,topic)  //初始读取到的topic offset:  currentTopicOffset.foreach(tp=>{println(s" 初始读取到的offset: $tp")})   //转换成需要的类型  val fromOffsets: Predef.Map[TopicPartition, Long] = currentTopicOffset.map { resultSet =>    // topic,partition,offset    new TopicPartition(topic, resultSet._1.toInt) -> resultSet._2.toLong  }.toMap   val kafkaParams = Map[String, Object](    "bootstrap.servers" -> "node1:9092,node2:9092,node3:9092",    "key.deserializer" -> classOf[StringDeserializer],    "value.deserializer" -> classOf[StringDeserializer],    "group.id" -> "MyGroupId11",    "auto.offset.reset" -> "latest",    "enable.auto.commit" -> (false: java.lang.Boolean)//默认是true  )   /**    * 2.将获取到的消费者offset传递给SparkStreaming,从指定的offset开始消费    */  val stream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](    ssc,    PreferConsistent,    ConsumerStrategies.Assign[String, String](fromOffsets.keys.toList, kafkaParams, fromOffsets)  )   stream.foreachRDD { (rdd:RDD[ConsumerRecord[String, String]]) =>     /** 3.消费到消息后，进行相应的业务逻辑处理 */    println("**** 业务处理完成  ****")     val offsetRanges: Array[OffsetRange] = rdd.asInstanceOf[HasOffsetRanges].offsetRanges     rdd.foreachPartition { iter =>      val o: OffsetRange = offsetRanges(TaskContext.get.partitionId)      println(s"topic:${o.topic}  partition:${o.partition}  fromOffset:${o.fromOffset}  untilOffset: ${o.untilOffset}")    }     /** 4.将当前批次最后的所有分区offsets保存到Redis中。更新redis中的offset */    saveOffsetToRedis(dbIndex,offsetRanges)  }   ssc.start()  ssc.awaitTermination()  ssc.stop() } 