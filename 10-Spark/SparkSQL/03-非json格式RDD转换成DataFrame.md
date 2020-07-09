**一、反射方式(不建议使用)**

需要自己定义类

1.自定义类要可序列化。

2.自定义类的访问级别是public。

3.RDD转成DataFrame后会根据映射字段按Ascii码排序。

4.将DataFrame转换成RDD时获取字段两种方式。一种是df.getInt(0)下标获取（不推荐使用），另一种是df.getAs("列名")获取（推荐）

case class MyPerson(id: Int, name: String, age: Int, score: Double) 

val session = SparkSession.builder().master("local").appName("demo03").getOrCreate() val peopleRdd : RDD[String] = session.sparkContext.textFile("./data/people.txt") /** 反射方式加载DataFrame */ val personRdd: RDD[MyPerson] = peopleRdd.map(elem => {  val args = elem.split(",")  val id = args(0).toInt  val name = args(1)  val age = args(2).toInt  val score = args(3).toDouble  MyPerson(id, name, age, score) }) import session.implicits._ val df: DataFrame = personRdd.toDF() df.show() 

**二、动态创建Scheme**

val session = SparkSession.builder().master("local").appName("demo03").getOrCreate() val peopleRdd : RDD[String] = session.sparkContext.textFile("./data/people.txt") val rowRdd: RDD[Row] = peopleRdd.map(elem => {  val args = elem.split(",")  val id = args(0).toInt  val name = args(1)  val age = args(2).toInt  val score = args(3).toDouble  Row(id, name, age, score) }) val structType: StructType = StructType(List[StructField](  StructField("id", IntegerType, true),  StructField("name", StringType, true),  StructField("age", IntegerType, true),  StructField("score", DoubleType, true) )) val df2: DataFrame = session.createDataFrame(rowRdd, structType) df2.show() 