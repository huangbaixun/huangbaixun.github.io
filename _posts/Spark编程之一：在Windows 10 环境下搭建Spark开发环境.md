#### Spark编程之一：在Windows 10 环境下搭建Spark开发环境

#### 环境搭建

搭建Window 10系统的版本及硬件信息

```
------------------
System Information
------------------
Operating System: Windows 10 企业版 64-bit (10.0, Build 14393) 
Language: Chinese (Simplified) (Regional Setting: Chinese (Simplified))
Processor: Intel(R) Core(TM) i7-6770HQ CPU @ 2.60GHz (8 CPUs), ~2.6GHz
Memory: 16384MB RAM
Available OS Memory: 16274MB RAM
```

1 安装JDK 1.8 [地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

2 安装Scala 2.11 [地址](http://www.scala-lang.org/download/2.11.8.html)
  *注*：Spark 2.0 官网文档介绍Spark 2.0 兼容Scala 2.11版本
  
3 下载 Spark 2.0.1 libs [地址](http://spark.apache.org/downloads.html) ,并解压
  ![Spark下载版本](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.1.png)
  
4 安装Intellij IDEA Community，[下载地址](https://www.jetbrains.com/idea/download/download-thanks.html?platform=windows&code=IIC) ,*注* 社区版是面向个人学习的免费版本；

5 在 Intellij IDEA Community安装 Scala插件

5.1 正常情况可以参照如下步骤安装 Scala插件：`run IDEA`->`plugins` ->`search scala` -> `install`。

![自动安装](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.2.png)
![自动安装](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.3.png)

> 但是由于各种原因，安装基本都会返回超时的错误:time out，参考下面的手动安装方法。

5.2 手动安装Scala插件，先下载插件[地址](http://plugins.jetbrains.com/files/1347/27110/scala-intellij-bin-2016.2.1.zip)，下载后手动安装：下载 ->`run IDEA` ->`Install plugin form disk` ->`select package` ->`OK` ->`restart IDEA`

 ![手动安装](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.4.png)

#### 创建Scala项目

1 `Run IDEA` -> `create new project` -> `scala`

 ![创建项目](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.5.png)

2 指定项目名,路径和JDK

 ![创建项目](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.6.png)

3 导入JAR选择菜单中的 `File` ->`Project Structure` ->`libraries` ->+`java` 选择Spark libs 的所有jar 包（图省事）：

4 创建一个object `选者src` ->`new scala class` 

![new code](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.9.png)

添加如下code 
```scala
// scalastyle:off println
package org.apache.spark.examples

import scala.math.random

import org.apache.spark.sql.SparkSession

/** Computes an approximation to pi */
object SparkPi {
  def main(args: Array[String]) {

    val spark = SparkSession
      .builder
      .appName("Spark Pi")
      .getOrCreate()
    val slices = if (args.length > 0) args(0).toInt else 2
    val n = math.min(100000L * slices, Int.MaxValue).toInt // avoid overflow
    val count = spark.sparkContext.parallelize(1 until n, slices).map { i =>
      val x = random * 2 - 1
      val y = random * 2 - 1
      if (x*x + y*y < 1) 1 else 0
    }.reduce(_ + _)
    println("Pi is roughly " + 4.0 * count / (n - 1))
    spark.stop()
  }
}
// scalastyle:on println
```

5 添加Hadoop_home 变量，否则会报如下错误

`java.io.IOException: Could not locate executable null\bin\winutils.exe in the Hadoop binaries.`

5.1 下载 hadoop bin [地址](https://github.com/srccodes/hadoop-common-2.2.0-bin/archive/master.zip)

5.2 将解压的bin放入到`D:\BigData\hadoop`目录下面

5.3 设置HADOOP_HOME环境:`run`->`Edit Configurations` ->`Envrionments Variables` 添加环境变量：
> HADOOP_HOME=D:\BigData\hadoop

5.4 设置vm options:`run`->`Edit Configurations` ->`vm options` 修改为
 `-Dspark.master=local`
![地址](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.1.8.png)
否则会报如下错误：
`Exception in thread "main" org.apache.spark.SparkException: A master URL must be set in your configuration`


####  运行程序 `run SparkPi`

结果如下：

>Pi is roughly 3.1400157000785005

计算PI的原理

上面的代码通过密集计算来求PI 的近似值，在(0,0) 到(1,1)的方块里面投掷飞镖，落在1/4圆的概率为1/4 * Pi R* R  = （落入圆中的次数）/投掷的总次数，也即： pi = 4 * (count)/(n-1)

![pi](https://raw.githubusercontent.com/huangbaixun/imgs/master/unit-circle.png)

*注*：R=1

#### 参考资料

1 [[使用IntelliJ IDEA配置Spark应用开发环境及源码阅读环境](http://blog.tomgou.xyz/shi-yong-intellij-ideapei-zhi-sparkying-yong-kai-fa-huan-jing-ji-yuan-ma-yue-du-huan-jing.html)


2 [Intellij idea配置scala开发环境](http://blog.csdn.net/dream_an/article/details/51935354)

3 [Run the Spark Pi example](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.0/bk_spark-quickstart/content/run_spark_pi.html)