#### Spark编程之二：Spark 2.0.1 集群安装，配置及运行

-----------------------------------------

*本文基于Spark 2.0.1，介绍Spark基于Hadoop 2.7的安装配置和简单的使用
硬件环境：三个节点，node1,node2,node3 ，其中node1是Master，另外俩个节点是Salve, 具体可以参考[从零开始学习大数据之一：伪分布式环境搭建](http://note.youdao.com/noteshare?id=de2d5e8e1c0d20be2df4f0ca049fe00f)*

##### 环境要求
1 下载并安装Scala-z.11.8到目录/opt/
`wget http://downloads.lightbend.com/scala/2.11.8/scala-2.11.8.tgz`

解压安装
```shell
tar -xzf scala-2.11.8.tgz
```

配置环境变量 `vi ~/.bashrc`
添加如下
```shell
export SCALA_HOME=/opt/scala-2.11.8
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$HIVE_HOME/bin:$HBASE_HOME/bin:$SPARK_HOME/bin:$SCALA_HOME/bin
```
生效环境变量

`su -`

2 下载Spark2.0.1 安装包并安装,参考[从零开始学习大数据之十：Apache Spark 2.0.1 安装](http://note.youdao.com/noteshare?id=0a31c1db29ac7a2b6cdfc4ba3daa50a4)


##### 配置Spark

1 添加SPARK运行时的环境变量`vi $SPARK_HOME/conf/spark-env.sh`

添加如下内容：

```shell
export JAVA_HOME=/opt/jdk1.8.0_101
export HADOOP_HOME=/opt/hadoop
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
export SCALA_HOME=/opt/scala-2.11.8
export SPARK_HOME=/opt/spark-2.0.1-bin-hadoop2.7
export SPARK_MASTER_IP=192.168.3.11 #node1 ip
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8099

export SPARK_WORKER_CORES=3     #每个Worker使用的CPU核数
export SPARK_WORKER_INSTANCES=1   #每个Slave中启动几个Worker实例
export SPARK_WORKER_MEMORY=3G    #每个Worker使用多大的内存
export SPARK_WORKER_WEBUI_PORT=8081 #Worker的WebUI端口号
export SPARK_EXECUTOR_CORES=1       #每个Executor使用使用的核数
export SPARK_EXECUTOR_MEMORY=1G     #每个Executor使用的内存

export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$HADOOP_HOME/lib/native
```
2 添加Slaves配置
```shell
cp slaves.template slaves
vi slaves
```
添加：
```shell
node2
node3
```
3 同步代码到Salves节点

```shell
scp -r /opt/spark-2.0.1-bin-hadoop2.7 node2:/opt/ #同步spark代码到node2
scp -r /opt/spark-2.0.1-bin-hadoop2.7 node3:/opt/ #同步spark代码到node3
```

4 启动HDFS & YARN 

```shell
start-all.sh
```

5 启动Spark Master

```shell
[root@node1 conf]# cd $SPARK_HOME
[root@node1 spark-2.0.1-bin-hadoop2.7]# sbin/start-master.sh 
starting org.apache.spark.deploy.master.Master, logging to /opt/spark-2.0.1-bin-hadoop2.7/logs/spark-root-org.apache.spark.deploy.master.Master-1-node1.out
```
在浏览器输入 http://192.168.3.11:8099/ 即可看到Spark的WebUI界面

![web UI](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.2.1.png)

6 启动Spark Slaves

```shell
[root@node1 spark-2.0.1-bin-hadoop2.7]# sbin/start-slaves.sh 
node3: starting org.apache.spark.deploy.worker.Worker, logging to /opt/spark-2.0.1-bin-hadoop2.7/logs/spark-root-org.apache.spark.deploy.worker.Worker-1-node3.out
node2: starting org.apache.spark.deploy.worker.Worker, logging to /opt/spark-2.0.1-bin-hadoop2.7/logs/spark-root-org.apache.spark.deploy.worker.Worker-1-node2.out
```

1 数据准备，随便放置一个txt文件到hdfs如下目录，后续例子将会统计此文件的word count：

```shell
[root@node1 spark-2.0.1-bin-hadoop2.7]# hdfs dfs -ls /user/huangbaixun
Found 1 items
-rw-r--r--   2 root root        789 2016-10-21 09:47 /user/huangbaixun/test.txt
```
2 启动Spark-Shell并连接刚才搭建的Spark集群

```shell
cd $SPARK_HOME
bin/spark-shell --master spark://node1:7077
```

3 刷新页面，可以看到我们启动的spark-shell

![web UI](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.2.2.png)

4 运行WordCount示例

```scala
var srcFile = sc.textFile("/user/huangbaixun/test.txt")
var a = srcFile.flatMap(line=>line.split("\\t")).map(word=>(word,1)).reduceByKey(_+_)
a.map(word=>(word._2,word._1)).sortByKey(false).map(word=>(word._2,word._1)).take(50).foreach(println)
```

![web UI](https://raw.githubusercontent.com/huangbaixun/imgs/master/2.2.3.png)

##### 参考资料

[Spark1.3.1安装配置运行](http://lxw1234.com/archives/2015/06/281.htm)