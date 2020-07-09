--master

master的地址，提交任务到哪里执行，如：spark://host:port,yarn,local 

--deploy-mode client | cluster

在本地启动driver或在cluster上启动，默认是client 

--class

应用程序的主类，仅针对Java或Scala应用 

--jars

用逗号分隔的本地jar包，设置后，这些jar将包含在driver和executor的classpath下 

--driver-class-path

传给driver额外的类路径 

--conf

指定spark配置属性的值 

--name

应用程序的名称 

--driver-cores

Driver使用的内核数，默认为1。在 yarn 或者 standalone 下使用 

--driver-memory

Driver的内存大小，默认为1G 

--executor-cores

每个executor使用的内核数，默认为1。在yarn或者standalone下使用 

--executor-memory

executor的内存大小，默认为1G 

--total-executor-cores

所有的executor的总核数。仅仅在 mesos 或者 standalone 下使用 

--num-executors 

启动executor的数量，默认为2。在 yarn下使用 