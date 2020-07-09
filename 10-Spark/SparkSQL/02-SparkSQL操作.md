**一、DataFrame数据源**

**1.数据源：读取json格式的文件**

val session = SparkSession.builder().master("local").appName("spark sql").getOrCreate() val df: DataFrame = session.read.json("./data/json") df.printSchema() // 默认打印前20条 df.show() df.show(10) 

**2.读取Json格式的字符串加载DataFrame**

val session = SparkSession.builder().master("local").appName("demo02").getOrCreate() val list  = List[String](  "{\"name\":\"zhangsan\",\"age\":\"18\"}",  "{\"name\":\"lisi\",\"age\":\"19\"}",  "{\"name\":\"wangwu\",\"age\":\"20\"}",  "{\"name\":\"maliu\",\"age\":\"21\"}" ) val rdd1: RDD[String] = session.sparkContext.parallelize(list) val df: DataFrame = session.read.json(rdd1) df.show() import session.implicits._ val jsonDS: Dataset[String] = list.toDS() val df2: DataFrame = session.read.json(jsonDS) df2.show() 

**二、DataFrame API使用**

/** select name,age from table where age >= 19 **/ val df2: Dataset[Row] = df.select("name", "age").where(df.col("age").>=(19)) df2.show() /** select * from table where age >= 19 **/ val df3: Dataset[Row] = df.filter("age >= 19") df3.show() /** select name,age+10 as addage from table */ val df4: DataFrame = df.select(df.col("name"), df.col("age").plus(10).alias("addage")) df4.show() /** select name,age from table order by name ,age desc */ import session.implicits._ val df5: Dataset[Row] = df.sort($"name".asc, $"age".desc) df5.show() 

**三、注册一个视图，直接使用sql语句**

df.createTempView("myTable") val df6: DataFrame = session.sql("select name,age from myTable where age = 18") df6.show() 