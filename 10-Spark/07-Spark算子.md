**一、Tranformation算子**

Transformations类算子叫做转换算子，该类算子是延迟加载，也叫懒加载。

**filter:**过滤保留符合条件的数据。true保留，false过滤掉

rdd.filter(_ % 2 == 0) 

**filterByRange：**作用KV格式的RDD上，过滤保留Key值符合范围的数据

val rdd1 = sc.parallelize(List(("a", 2), ("b", 1), ("d", 4), ("c", 1), ("e", 4))) // 结果b，c，d都留了下来 val rdd2 = rdd1.filterByRange("b", "d") 

\---------------------------------------------------------------------------------------------------------------------------

**map:**真正的Executor中执行时，是一条一条的将数据拿出来处理。输入一条记录输出一条记录。

words.map((_, 1)) 

**flapMap:**将RDD中的每一条数据映射为0到多个输出项。输入一条记录，输出n条记录。n可以为0

var lines = sc.textFile("/root/hello.txt") lines.flatMap(_.split(" ")) 

**flapMapValues：**作用在KV格式的RDD上，对value进行处理，一对多

val rdd1 = sc.parallelize(List(("a", "2 2"), ("b", "1 5"), ("d", "4 6"))) // (a,2)(a,2)(b,1)(b,5)(d,4)(d,6) val rdd2 = rdd1.flatMapValues(_.split(" ")) 

**mapValues：**作用在KV格式的RDD上，对value进行处理，一对一

// (a,2 21)(b,1 51)(d,4 61) rdd1.mapValues(_+1) 

**mapPartitions**：与map类似，遍历的单位是每个partition上的数据。

words.map((_, 1)) 

**mapPartitionsWithIndex**：一次拿出一个分区。除此之外，还会携带分区的编号。

val rdd1 = sc.parallelize(List(1, 2, 3, 4, 5, 6, 7, 8, 9), 2) val value = rdd1.mapPartitionsWithIndex((index, it) => {  // 拿到一个分区，然后通过分区再取里面的数据  it.map(x => s"part:$index,ele:$x") }) 

\---------------------------------------------------------------------------------------------------------------------------

**sample:**随机抽样，根据传进去的小数按比例有放回或无放回的抽样。

\# 抽取50%的数据 rdd.sample(true, 0.5) 

\---------------------------------------------------------------------------------------------------------------------------

**reduceByKey:**将相同的key的数据根据相应的逻辑进行处理

pairWords.reduceByKey(_ + _) 

**aggregateByKey：**先在分区内对key相同的数据进行聚合处理

val rdd1 = sc.parallelize(List(("dog", 2), ("cat", 1), ("mouse", 4), ("cat", 1), ("mouse", 4), ("dog", 2)),2) val result = rdd1.aggregateByKey(0)(_ + _, _ + _) 

\---------------------------------------------------------------------------------------------------------------------------

**groupBy**：根据指定元素分组。返回（K，Iterable<V>）。

val subTeacherAndCount: RDD[((String, String), Int)] = subTeacherAndOne.reduceByKey(_ + _) val grouped: RDD[(String, Iterable[((String, String), Int)])] = subTeacherAndCount.groupBy(_._1._1) 

**groupByKey**：作用的KV格式的RDD上，根据Key进行分组。返回（K，Iterable<V>）

val rdd1: RDD[(String, Int)] = lines.map(line => {  val words = line.split("\t")  Tuple2(words(0), words(1).toInt) }) val rdd2: RDD[(String, Iterable[Int])] = rdd1.groupByKey() 

\---------------------------------------------------------------------------------------------------------------------------

**sortBy:**指定的第几个元素进行排序。默认是升序

\# 多元组数据，Array((1, 6, 3), (2, 3, 3), (1, 1, 2), (1, 3, 5), (2, 1, 2)) # 按照第二个元素排序 reduceResult.sortBy(_._2) # 先按照第一个元素升序排序，如果第一个元素相同，再进行第三个元素进行升序排序  val datas2 = sc.parallelize(arr) val sorts2 = datas2.sortBy(e => (e._1,e._2)) 

\# 单列元素 val rdd1 = sc.makeRDD(List(1, 3, 2, 9, 5, 6, 7, 8)) rdd1.sortBy(x =>x,false).foreach(println) 

**sortByKey：**作用在<K,V>格式,根据key进行排序

reduceResult.sortByKey() 

\---------------------------------------------------------------------------------------------------------------------------

**join**：作用在KV格式的RDD上。内连接，根据key进行连接。join后的分区数与父RDD分区数多的那一个相同。

val ageRdd: RDD[(String, Int)] = sc.parallelize(List[(String, Int)](  ("张三", 18),  ("李四", 15),  ("王五", 18),  ("赵六", 19),  ("孙琪", 20) )) val scoreRdd: RDD[(String, Int)] = sc.parallelize(List[(String, Int)](  ("张三", 60),  ("李四", 80),  ("王五", 100),  ("赵六", 120),  ("田七", 20) )) val joinResult: RDD[(String, (Int, Int))] = ageRdd.join(scoreRdd) 

**leftOuterJoin：**左连接，以左表为基表。

val leftOuterJoinResult: RDD[(String, (Int, Option[Int]))] = ageRdd.leftOuterJoin(scoreRdd) 

**rightOuterJoin：**右连接，以右表为基表。

val rightOuterJoin: RDD[(String, (Option[Int], Int))] = ageRdd.rightOuterJoin(scoreRdd) 

**fullOuterJoin：**全连接

val fullOuterJoinResult: RDD[(String, (Option[Int], Option[Int]))] = ageRdd.fullOuterJoin(scoreRdd) 

\---------------------------------------------------------------------------------------------------------------------------

**union**：合并两个数据集。两个数据集的类型要一致。

返回新的RDD的分区数是合并两个RDD分区数的总和。

val rdd1: RDD[Int]  = sc.parallelize(List[Int](1, 2, 3, 4, 5, 6)) val rdd2: RDD[Int]  = sc.parallelize(List[Int](4, 5, 6, 7, 8)) val unionResult: RDD[Int] = rdd1.union(rdd2) 

**intersection**：取两个数据集的交集。

返回新的RDD与父RDD分区多的一致。

val intersectionResult: RDD[Int] = rdd1.intersection(rdd2) 

**subtract**：取两个数据集的交集。

结果RDD的分区数与subtract前面的RDD的分区数一致。

val subtractResult: RDD[Int] = rdd1.subtract(rdd2) 

\---------------------------------------------------------------------------------------------------------------------------

**distinct**：去重。distinct = map + reduceByKey + map

val rdd1 = sc.parallelize(List[Int](1, 1, 2, 3, 4, 5, 5)) val distinctResult: RDD[Int] = rdd1.distinct() 

\---------------------------------------------------------------------------------------------------------------------------

**cogroup**：当调用类型(K,V)和(K,W)的数据上时，返回一个数据集(K,(Iterable<V>,Iterable<W>))。

结果RDD的分区数与父RDD多的一致。

 

\---------------------------------------------------------------------------------------------------------------------------

**repartition**：增加或减少分区。会产生shuffle。（多个分区到一个分区不会产生shuffle）

val rdd1 = sc.parallelize(List[Int](1, 1, 2, 3, 4, 5, 5),3) val repartitionRdd = rdd1.repartition(4) 

**coalesce**：常用来减少分区，第二个参数是分区减少过程中是否产生shuffle。

true为产生shuffle，false不产生shuffle。默认是false。

如果coalesce设置的分区数比原来的RDD的分区数还多的话，第二个参数设置为false不会起作用，如果设置为true就和repartition

的效果一样。即reparation(num) = coalesce(num,true).

val coalesceRdd = rdd1.coalesce(4) // 不生效 val coalesceRdd = rdd1.coalesce(4, true) // 生效 val coalesceRdd = rdd1.coalesce(2) 

\---------------------------------------------------------------------------------------------------------------------------

**zip**：将两个RDD中的元素变成一个KV格式的RDD，两个RDD的每一分区元素个数必须相同。

 

**zipWithIndex**：将RDD中的元素和这一元素在该RDD中索引值（从0开始）组合成KV对。

 

\---------------------------------------------------------------------------------------------------------------------------

**mapValues**：针对KV格式的RDD，该算子对KV格式的RDD中的value进行操作，返回的是KV格式的RDD。

 

**二、Action算子**

Action算子叫做行动算子，该类算子是触发执行。一个Spark application中有几个Action类算子，就有几个job运行。

**count：**返回数据集中的元素数。会在结果计算完成后回收到Driver端。

 

**countByKey**：作用在KV格式的RDD上，根据Key计数相同的Key的数据集元素。

val rdd: RDD[(String, Int)] = sc.makeRDD(Array[(String, Int)](  ("a", 1),  ("b", 2),  ("a", 3),  ("c", 4),  ("d", 5) )) // 结果：Map(d -> 1, a -> 2, b -> 1, c -> 1) val result: collection.Map[String, Long] = rdd.countByKey() 

**countByValue**：根据数据集每个元素相同的内容来计数。返回相同内容的元素对应的条数。

 

\---------------------------------------------------------------------------------------------------------------------------

**take**：返回一个包含数据集前n个元素的集合。

 

**first**：first=take(1),返回数据集中的第一个元素

 

**top**(num)：返回数据集中最大的前num个。

 

**takeOrdered**(num)：将数据集中的数据排序（默认升序），取前num个。

 

\---------------------------------------------------------------------------------------------------------------------------

**foreach**：循环遍历数据集中的每个元素，运行相应的逻辑。

 

**foreachPartition**：遍历的数据时每个Partition的数据。

val rdd = sc.parallelize(List[Int](1, 2, 3, 4, 5), 2) rdd.foreachPartition(iter => {  println("创建连接")  val list = new ListBuffer[Int]  iter.foreach(list.append(_))  println("数据插入 " + list.toString())  println("关闭连接")  println("====================") }) 

\---------------------------------------------------------------------------------------------------------------------------

**collect**：将计算结果回收到Driver端。

 

**collectAsMap**：对K，V格式的数据回收转换成Map<K,V>。

val rdd: RDD[(Int, String)] = sc.parallelize(List[(Int, String)](  (1, "aa"),  (2, "bb"),  (3, "cc"),  (4, "dd") )) rdd.collectAsMap().foreach(println) 

\---------------------------------------------------------------------------------------------------------------------------

**takeSample**(boolean, num, seed)：从数据集中随机获取num个。第一个参数是有无放回，第二个参数是随机获取几个元素，第三个参数如果固定，那么每次获取的数据固定。

 

\---------------------------------------------------------------------------------------------------------------------------

**aggregate ：**是一个聚合函数，接受多个输入，并按照一定的规则运算以后输出一个结果值。先对各个分区内数据处理

val rdd1 = sc.parallelize(List(1, 2, 3, 4, 5, 6, 7, 8, 9), 2) // 结果：13 val i = rdd1.aggregate(0)((x, y) => math.max(x, y), (x, y) => x + y) val rdd2 = sc.parallelize(List("a", "b", "c", "d", "e", "f"),2) // 结果：||abc|def或||def|abc val str = rdd2.aggregate("|")((x, y) => x + y, (x, y) => x + y) 

**三、持久化算子**

将数据持久化

1.**cache**：将数据持久化到内存中。cache是懒执行

如果数据太大，内存根本无法存储，还使用cache做持久化，此时并不会OOM，内存能存多少就存多少，存不下的丢掉。下次其他RDD用到这个数据，一部分从内存中获取，一部分根据RDD的依赖关系开始读取。

 

2.**persist**：可以指定持久化的级别。最常用的是MEMORY_ONLY，MEMORY_AND_DISK。_2表示的是副本数

 

3.**checkpoint**：将数据持久化到磁盘，还可以切断RDD之间的依赖关系。checkpoint目录的数据不归Spark管理，当Spark Application运行结束后，目录数据不会被删除。

 