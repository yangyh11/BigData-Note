**一、Shark**

Hive on Spark

Hive：解析优化，存储 Spark：执行引擎 

Shark的整体设计架构对Hive的依赖性太强，难以支持其长远发展，比如不能和Spark的其他组件进行很好的集成，无法满足Spark的一栈式解决方案。

**二、SparkSQL**

**1.SparkSQL介绍**

Spark on Hive

Hive：存储 Spark：解析优化，执行引擎 

Hive是Shark的前身，Shark是SparkSQL的前身，SparkSQL产生的根本原因是其完全脱离了Hive的限制。

1.SparkSQL支持查询原生的RDD。RDD是Spark平台的核心概念，是Spark能够高效的处理大数据的各种场景的基础。

2.能够在Scala中写SQL语句。支持简单的SQL语法检查，能够在Scala中写Hive语句访问Hive数据，并将结果返回作为RDD使用。

**2.DataFrame**

DataFrame是一个分布式数据容器。与RDD类似，然而DataFrame更像传统数据的二维表格，除了数据以外，还掌握数据的结构信息，即Schema。同时，和Hive类似，DataFrame也支持嵌套数据类型（struct、array、map）。

**DataFrame就是Row类型的DataSet。**

**3.SparkSQL的数据源**

SparkSQL的数据源可以是JSON类型的字符串，JDBC，Parquent，Hive，Hdfs等。

**4.SparkSQL底层架构**

首先拿到sql后解析一批未被解决的逻辑计划，再经过分析得到分析后的逻辑计划，再经过一批优化规则转换成一批最佳的逻辑计划。再经过SparkPlanner的策略转化成一批物理计划，随后经过消费模型转换成一个个的Spark任务执行。

**5.谓词下推**

![img](E:\Download\YoudaoNote\yangyh11@163.com\413ab6fb83e345b888263ef005179e94\clipboard.png)