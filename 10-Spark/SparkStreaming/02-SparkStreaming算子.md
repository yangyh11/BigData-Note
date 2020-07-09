**一、transformation类算子**

**1.flatMap、map、reduceByKey...**

基本上RDD的transformation算子DStream都有，是不是全都有，有待考证...

**2.updateStateByKey**

根据key进行分组，然后对每组的value进行处理。

注意：需要设置checkpoint目录用于保存状态，自从SparkStreaming启动以来将所有的key的状态进行统计。

代码

def main(args: Array[String]): Unit = {  val conf = new SparkConf()  conf.setMaster("local[2]")  conf.setAppName("UpdateStateByKey demo")   val ssc = new StreamingContext(conf, Durations.seconds(5))  ssc.sparkContext.setLogLevel("error")  ssc.checkpoint("./data/ck/sparkSteaming")   val lines: ReceiverInputDStream[String] = ssc.socketTextStream("node4", 9999)   val pairWords: DStream[(String, Int)] = lines.flatMap(_.split(" ")).map((_, 1))   val result: DStream[(String, Int)] = pairWords.updateStateByKey((seq: Seq[Int], option: Option[Int]) => {    var preValue = option.getOrElse(0)    seq.foreach(elem => preValue += elem)    Option(preValue)  })  result.print()   ssc.start()  ssc.awaitTermination() 

**3.reduceByKeyAndWindow窗口操作**

示意图

![img](E:\Download\YoudaoNote\yangyh11@163.com\19c7dbc7ff664abcbe30357ee1fd77bb\clipboard.png)

1.假设每隔5s一个batch，上图中窗口长度为15s，窗口滑动间隔10s。

2.窗口长度和窗口滑动间隔必须是batchInternal的整数倍。不是整数倍会检测报错。

代码：

def main(args: Array[String]): Unit = {  val conf = new SparkConf()  conf.setMaster("local[2]")  conf.setAppName("ReduceByKeyAndWindow demo")   val ssc = new StreamingContext(conf, Durations.seconds(5))  ssc.sparkContext.setLogLevel("error")   val lines: ReceiverInputDStream[String] = ssc.socketTextStream("node4", 9999)   val pairWords: DStream[(String, Int)] = lines.flatMap(_.split(" ")).map((_, 1))   // windowDuration，slideDuration。每隔10s按照指定的逻辑处理最近15s（3个批次）数据  val result: DStream[(String, Int)] = pairWords.reduceByKeyAndWindow((v1: Int, v2: Int) => {    v1 + v2  }, Durations.seconds(15), Durations.seconds(10))   result.print()   ssc.start()  ssc.awaitTermination()  ssc.stop() } 

窗口操作的优化

![img](E:\Download\YoudaoNote\yangyh11@163.com\2253671150bb4cbaa325b8ec16635702\clipboard.png)

优化前：将最近窗口的所有批次进行参与计算（t + (t + 1) + (t + 2) + (t + 3) + (t + 4)）。

优化后：将操作结果存起来（checkpoint），下次计算只需要减去不在最近窗口的批次（t-1），加上新加入窗口的批次（t+4）。

代码：

ssc.checkpoint("./data/streamingCheckpoint") val result: DStream[(String, Int)] = pairWords.reduceByKeyAndWindow(  (v1:Int, v2:Int)=>{v1+v2},  (v1:Int, v2:Int)=>{v1-v2},  Durations.seconds(15),  Durations.seconds(10)) result.print() 

**二、output operation算子**

**foreachRDD**

1.可以获取Dstream中的RDD，对RDD使用Transformation类算子进行转换，但是最后一定要在foreachRDD算子内使用RDD的action类算子触发执行，否则RDD的transformation类算子不会执行。

2.foreachRDD内，RDD的算子里的代码实在Executor端执行，算子外的代码实在Driver端执行的，利用这个特点可以动态的改变广播变量。

代码：

def main(args: Array[String]): Unit = {   val conf = new SparkConf()  conf.setMaster("local[2]")  conf.setAppName("ForeachRDD demo")   val ssc: StreamingContext = new StreamingContext(conf, Durations.seconds(5))  ssc.sparkContext.setLogLevel("error")   val lines: ReceiverInputDStream[String] = ssc.socketTextStream("node4", 9999)   val words: DStream[String] = lines.flatMap(_.split(" "))   val pairWord: DStream[(String, Int)] = words.map((_, 1))   val result: DStream[(String, Int)] = pairWord.reduceByKey(_ + _)   result.foreachRDD(rdd => {    println("************  在Driver端执行 ******************")     val mapRdd: RDD[String] = rdd.map(tp => {      println("----------  在Executor端执行 ------------------")      tp._1 + "#" + tp._2    })    // Executor端执行的代码必须得使用action算子才能触发    mapRdd.foreach(println)  })   ssc.start()  // 等待Spark程序被终止  ssc.awaitTermination()  ssc.stop() } 