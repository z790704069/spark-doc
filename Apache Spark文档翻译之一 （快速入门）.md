![图片][5]
[英文版原文][4]

本教程为使用spark的快速入门介绍。首先我们会通过Spark的交互式shell(Python或者Scala)来介绍API，然后展示如何使用java、scala以及python来编写spark程序。

跟随这个指导，首先从Spark官网下载spark软件包。因为我们未必正在使用HDFS，你可以下载针对任何版本hadoop的spark软件包。

注意，在Spark 2.0之前，spark主要的编程接口是弹性分布式数据集（Resilient Distributed Dataset:RDD）。Spark 2.0之后，RDDS被Dataset取代，Dataset跟RDD非常相似，但是具有更好的性能。RDD的接口在2.0以后依然被支持，你可以在这里[RDD编程指导][1]获得更完整的参考。然而，我们更加推荐你使用Dataset,因为相对RDD，它有更好的性能。点击[SQL编程指导][2]获取更多关于Dataset的信息。

# 使用Spark Shell进行交互式分析
## 基础
Spark的shell提供了一种简单方式来学习API，并且是一个交互式分析数据的强大工具。使用Scala和python语言都能轻松使用。在Spark目录下运行如下命令来启动它：
```
./bin/spark-shell
```
Spark的主要抽象是一个关于items的分布式集合，我们称之Dataset。Dataset可以创建自Hadppo inputFormats（例如HDFS的文件），或者通过其他Dataset来转换得到。现在，让我们使用Spark目录中的README文件内容来制作一个新的Dataset:
```
scala> val textFile = spark.read.textFile("README.md")
textFile: org.apache.spark.sql.Dataset[String] = [value: string]
```
你可以通过调用一些动作（actions）直接从Dataset中获取值，或者转换当前Dataset得到一个新的Dataset。对于更多细节，请查看[API文档][3]。
```
scala> textFile.count() // Number of items in this Dataset
res0: Long = 126 // May be different from yours as README.md will change over time, similar to other outputs

scala> textFile.first() // First item in this Dataset
res1: String = # Apache Spark
```
现在，让我们通过转换（transform）Dataset来得到一个新的Dataset。我们调用filter来返回一个新的Dataset：
```
scala> val linesWithSpark = textFile.filter(line => line.contains("Spark"))
linesWithSpark: org.apache.spark.sql.Dataset[String] = [value: string]
```
当然，我们可以把动作和转换放在一起一次性操作：
```
scala> textFile.filter(line => line.contains("Spark")).count() // How many lines contain "Spark"?
res3: Long = 15
```
## 更多在Dataset上的操作
Dataset的动作和转换可以被用于更复杂的计算。接下来让我们看看如何找到文件中单词数最多的一行：
```
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)
res4: Long = 15
```

首先将一行转换为整数值，创建一个Dataset。在Dataset上调用reduce来查找最大的单词数。map和reduce的参数值是Scala函数的字面值，并且能使用任何语言的特性或者Scala/Java库。例如，我们可以很容易地调用在别处声明的函数。下面，我们使用Math.max()函数来让代码更容易理解：
```
scala> import java.lang.Math
import java.lang.Math

scala> textFile.map(line => line.split(" ").size).reduce((a, b) => Math.max(a, b))
res5: Int = 15
```

一个常见的是数据流模式是在hadoop中流行的mapreduce。Spark可以很容易地实现mapreducede：
```
scala> val wordCounts = textFile.flatMap(line => line.split(" ")).groupByKey(identity).count()
wordCounts: org.apache.spark.sql.Dataset[(String, Long)] = [value: string, count(1): bigint]
```
这里，我们调用flatmap讲行的Dataset转换成单词的Dataset,然后结合groupByKey和count来计算文件中每个单词的个数，结果形式为（String,Long）对的Dataset。我们可以在shell中调用collect收集单词个数
```
scala> wordCounts.collect()
res6: Array[(String, Int)] = Array((means,1), (under,2), (this,3), (Because,1), (Python,2), (agree,1), (cluster.,1), ...)
```
## 缓存



  [1]: http://spark.apache.org/docs/latest/rdd-programming-guide.html
  [2]: http://spark.apache.org/docs/latest/sql-programming-guide.html
  [3]: http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset
  [4]: http://spark.apache.org/docs/latest/quick-start.html
  [5]: http://kooola.com/upload/2018/06/7qanlnrum8i7jrg86u2vita3b2.jpg