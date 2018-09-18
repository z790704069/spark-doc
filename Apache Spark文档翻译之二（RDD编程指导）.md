# 概要
在较高的层面上，每个Spark应用程序都包含一个驱动程序，该程序运行用户的主要功能并在集群上执行各种并行操作。Spark提供的核心抽象是弹性分布式数据集（简称RDD）,RDD是一个元素集合，这些集合分布在跨节点的集群上以保证能够对这些元素进行并行操作。创建RDD有多种方式：通过读取hdfs文件系统中的文件（或者其他任何hadoop支持的文件系统），或者通过驱动程序中已经存在的Scala集合来转换得到。用户还可以让Spark将RDD缓存到内存中，这样并行操作是能够更高效的重复使用。最后，RDD会自动从节点故障中恢复。
在Spark中，第二个重要的抽象是可以在并行操作中使用的共享变量。默认情况下，当Spark并行运行一个函数作为不同节点上的一组任务时，它会将函数中使用的每个变量的副本发送给每个任务。有时候，一个变量在不同任务间共享，或者任务和驱动程序间共享。Spark支持两种类型的共享变量：广播变量以及累加器。广播变量可以缓存在所有节点上。累计器，类似计数器和总和。
本文使用Spark的支持语言来展示各个特性。如果你之前使用过Spark shell（Scala的bin/spark-shell或者python的bin/pyspark），接下来将会很简单。

# 与Spark连接
Scale2.3.1默认使用Scala2.11构建并分发（Spark也可以使用其他版本的Scala）。要使用Scala编写Spark程序，你需要使用兼容的版本（例如2.11.x）。
要编写Spark应用程序，需要在Spark上添加Maven依赖项。
```
groupId = org.apache.spark
artifactId = spark-core_2.11
version = 2.3.1
```
此外，如果想访问HDFS集群，你需要添加指定版本hadoop-client依赖。
```
groupId = org.apache.hadoop
artifactId = hadoop-client
version = <your-hdfs-version>
```
最后，你需要在你的程序中导入一些Spark相关类
```
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf
```

# 初始化Spark

