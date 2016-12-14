### 从零开始学习大数据之十：Apache Spark 2.0.1 安装

#### 一、环境准备

请参考如下链接，完成Hadoop安装

- [Hadoop 2.7.3安装](http://note.youdao.com/noteshare?id=e10087e26ea8651007b05eb7d25e5817)

#### 二、下载Spark Hadoop包安装

下载Spark包 到文件夹/opt
```shell
wget http://d3kbcqa49mib13.cloudfront.net/spark-2.0.1-bin-hadoop2.7.tgz
```

解压

```shell
tar -xvf spark-2.0.1-bin-hadoop2.7.tgz
```
#### 三、修改配置

`vim ~/.bashrc`
添加如下内容
```shell
export SPARK_HOME=/opt/spark-2.0.1-bin-hadoop2.7
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$HIVE_HOME/bin:$SPARK_HOME/bin #添加$HIVE_HOME/bin 到PATH中
```
刷新环境变量:`su -`

添加Spark配置信息

```shell
cp spark-env.sh.template spark-env.sh
```

修改 spark-env.sh，添加如下内容
```shell
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
```

修改  /etc/profile,添加如下

```shell
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native/:$LD_LIBRARY_PATH
```

生效新的环境变量
```shell
source /etc/profile
```

#### 四、运行Spark样例程序和交互式命令

Spark自带几个示例程序。在examples/src/main目录有Scala,Java,Python，和R的例子，在Spark的安装根目录执行`bin/run-example <class> [params]` ，比如：

```shell
#计算Pi的值
./bin/run-example SparkPi 10
```

可以看到计算的结果

>Pi is roughly 3.1432951432951435

spark-shell是spark提供的一个交互式命令行工具，也是学习spark最好的选择

```shell
./bin/spark-shell --master local[2]
```

--master  参数用于指定 分布式集群 master的 URL，或者本地使用一个线程，local[N]使用N个本地线程运行。可以使用本地模式用来测试，如果想知道所有的其他参数可以使用 `spark-shell --help` 来获取。 Spark也提供Python API，使用`bin/pyspark`就可以在一个python 解析器环境里面运行Spark交互式命令。

```shell
./bin/pyspark --master local[2]
```

执行一个Pyhon的例子

```shell
./bin/spark-submit examples/src/main/python/pi.py 10
```
运行的结果： `Pi is roughly 3.141060`


#### 五、运行Spark on YARN集群

启动YARN集群
```shell
start-all.sh
```
提交任务到YARN：

```shell
./bin/spark-submit --class org.apache.spark.examples.SparkPi  --master yarn    --deploy-mode cluster  --driver-memory 4g   --executor-memory 2g     --executor-cores 1  examples/jars/spark-examples_2.11-2.0.1.jar 10

application_1477578019796_0004 (state: RUNNING)
16/10/27 10:45:02 INFO yarn.Client: Application report for application_1477578019796_0004 (state: RUNNING)
16/10/27 10:45:03 INFO yarn.Client: Application report for application_1477578019796_0004
shell
```

#### 六、参考文章

1.[Spark Overview](http://spark.apache.org/docs/latest/)
2.[Run Spark On YARN](http://spark.apache.org/docs/latest/running-on-yarn.html)
3.[why-does-bin-spark-shell-give-warn-nativecodeloader-unable-to-load-native-had](http://stackoverflow.com/questions/23572724/why-does-bin-spark-shell-give-warn-nativecodeloader-unable-to-load-native-had)
