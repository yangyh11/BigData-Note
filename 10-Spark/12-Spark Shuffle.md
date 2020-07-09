**一、Shuffle的概念**

**二、HashShuffleManager**

**1.普通机制**

![img](E:\Download\YoudaoNote\yangyh11@163.com\65e3e42506c340718db3f0b14f0eff10\clipboard.png)

**执行流程**

1.每一个map task将不同结果写到不同的buffer中，每个buffer的大小未32k。buffer起到数据缓存的作用。

2.每个Buffer文件最后对应一个磁盘小文件。

3.reduce task来拉取对应的磁盘小文件。

**总结：**

1.map task的计算结果会根据分区器（默认是hashPartitioner）来决定写入到哪一个磁盘小文件中去。reduce task会去map端拉取相应的磁盘小文件。

2.产生的磁盘小文件的个数

M * R

M：map task的个数

R：reduce task的个数

**存在的问题：**

产生的磁盘小文件过多导致以下问题：

1.在Shuffle write过程中会产生很多写磁盘小文件的对象。

2.在Shuffle Read过程中会产生很多读取磁盘小文件的对象。

3.在JVM堆内存中对象过多会造成频繁的gc，gc还无法解决运行所需的内存的话，就会OOM。

4.在数据传输过程中会有频繁的网络通信，频繁的网络通信出现通信故障的可能性大大增加。一旦网络通信出现故障会导致shuffle file connot find，由于这个错误导致task失败，TaskScheduler不负责重试，有DAGScheduler负责重试Stage。

**2.合并机制**

![img](E:\Download\YoudaoNote\yangyh11@163.com\1b154c0cc14b480d9402d02376198e95\clipboard.png)

**总结**

产生磁盘小文件的个数：C * R

C：core的个数

R：reduce的个数

**三、SortShuffleManager**