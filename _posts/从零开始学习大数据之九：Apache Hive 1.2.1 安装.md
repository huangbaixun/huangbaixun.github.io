### 从零开始学习大数据之九：Apache Hive 1.2.1 安装

#### 一、环境准备

请参考如下链接，完成Hadoop安装

- [Hadoop 2.7.3安装](http://note.youdao.com/noteshare?id=e10087e26ea8651007b05eb7d25e5817)

#### 二、下载安装包

下载Hive最新稳定版本
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hive/stable/apache-hive-1.2.1-bin.tar.gz
```

解压并重命名文件夹为hive

```shell
tar -xzvf apache-hive-1.2.1-bin.tar.gz
mv apache-hive-1.2.1-bin hive
```
#### 三、修改配置

`vim ~/.bashrc`
添加如下内容
```shell
export HIVE_HOME=/opt/hive
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$HIVE_HOME/bin #添加$HIVE_HOME/bin 到PATH中
```
刷新环境变量:`su -`

#### 四、启动Hive

启动Hadoop
```shell
[root@node1 ~]# start-all.sh 
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
Starting namenodes on [node1]
node1: starting namenode, logging to /opt/hadoop/logs/hadoop-root-namenode-node1.out
node2: starting datanode, logging to /opt/hadoop/logs/hadoop-root-datanode-node2.out
node3: starting datanode, logging to /opt/hadoop/logs/hadoop-root-datanode-node3.out
starting yarn daemons
starting resourcemanager, logging to /opt/hadoop/logs/yarn-root-resourcemanager-node1.out
node2: starting nodemanager, logging to /opt/hadoop/logs/yarn-root-nodemanager-node2.out
node3: starting nodemanager, logging to /opt/hadoop/logs/yarn-root-nodemanager-node3.out
```

启动Hive

确认: HADOOP_HOME 已经配置到你的环境变量中，或者执行`export HADOOP_HOME=/opt/hadoop`

另外，我需要使用HDFS的命令创建如下俩个目录，并且赋权限,/user/hive/warehouse也即是hive的根目录：hive.metastore.warehouse.dir

```shell
[root@node1 ~]# $HADOOP_HOME/bin/hadoop fs -mkdir /tmp 
[root@node1 ~]# $HADOOP_HOME/bin/hadoop fs -mkdir  -p /user/hive/warehouse 
[root@node1 ~]# $HADOOP_HOME/bin/hadoop fs -chmod g+w /tmp
[root@node1 ~]# $HADOOP_HOME/bin/hadoop fs -chmod g+w /user/hive/warehouse
```

#### 五、运行 HiveServer2 和 Beeline

从 Hive 2.1版本开始 , 我们需要运行  chematool 来初始化hive. 我用使用derby来存储元数据

```shell
[root@node1 ~]#  $HIVE_HOME/bin/schematool -dbType derby -initSchema
Metastore connection URL:	 jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :	 org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:	 APP
Starting metastore schema initialization to 1.2.0
Initialization script hive-schema-1.2.0.derby.sql
Initialization script completed
schemaTool completed
```

HiveServer2（在Hive 0.11中引入）有自己的命令行工具：Beeline。 HiveCLI现在已经废弃因为它缺少HiveServer2 Beeline的多用户，安全和其他功能。 从shell运行HiveServer2和Beeline：

```shell
$ $HIVE_HOME/bin/hiveserver2&
```
创建root账号HDFS的工作目录和权限(不执行此步骤连接Hive会报错)
```shell
$ hadoop fs -mkdir /user/root
$ hadoop fs -chown root:root /user/root
$ hdfs dfs -chmod -R 777 /tmp/
$ hdfs dfs -chmod -R 777  /user/hive/warehouse
```

Beeline通过JDBC来连接HiveServer2， JDBC的URL由HiveServer2部署的机器和端口组成，目前端口为：10000，通过如下命令主要完成如下事项：

- 启动HiveServer2
- 通过Beeline连接到启动HiveServer2
- 创建一个表dual
- 向表中插入一条数据
- 查询表dual中的数据
- 退出beeline:`!q`

```shell
[root@node1 ~]# $HIVE_HOME/bin/beeline -u jdbc:hive2://localhost:10000
Connecting to jdbc:hive2://localhost:10000
Connected to: Apache Hive (version 1.2.1)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 1.2.1 by Apache Hive
0: jdbc:hive2://localhost:10000> show databases;
+----------------+--+
| database_name  |
+----------------+--+
| default        |
+----------------+--+
1 row selected (1.335 seconds)
0: jdbc:hive2://localhost:10000> use default
0: jdbc:hive2://localhost:10000> ;
No rows affected (0.116 seconds)
0: jdbc:hive2://localhost:10000> create table dual( tmp string);
No rows affected (0.134 seconds)
0: jdbc:hive2://localhost:10000> insert into dual values ('1');
INFO  : Number of reduce tasks is set to 0 since there's no reduce operator
INFO  : number of splits:1
INFO  : Submitting tokens for job: job_1477403909882_0001
INFO  : The url to track the job: http://node1:8088/proxy/application_1477403909882_0001/
INFO  : Starting Job = job_1477403909882_0001, Tracking URL = http://node1:8088/proxy/application_1477403909882_0001/
INFO  : Kill Command = /opt/hadoop/bin/hadoop job  -kill job_1477403909882_0001
INFO  : Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 0
INFO  : 2016-10-25 10:59:40,644 Stage-1 map = 0%,  reduce = 0%
INFO  : 2016-10-25 10:59:49,074 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.12 sec
INFO  : MapReduce Total cumulative CPU time: 1 seconds 120 msec
INFO  : Ended Job = job_1477403909882_0001
INFO  : Stage-4 is selected by condition resolver.
INFO  : Stage-3 is filtered out by condition resolver.
INFO  : Stage-5 is filtered out by condition resolver.
INFO  : Moving data to: hdfs://node1:9000/user/hive/warehouse/dual/.hive-staging_hive_2016-10-25_10-59-28_217_8800961220422921799-2/-ext-10000 from hdfs://node1:9000/user/hive/warehouse/dual/.hive-staging_hive_2016-10-25_10-59-28_217_8800961220422921799-2/-ext-10002
INFO  : Loading data to table default.dual from hdfs://node1:9000/user/hive/warehouse/dual/.hive-staging_hive_2016-10-25_10-59-28_217_8800961220422921799-2/-ext-10000
INFO  : Table default.dual stats: [numFiles=1, numRows=1, totalSize=2, rawDataSize=1]
No rows affected (22.357 seconds)
0: jdbc:hive2://localhost:10000> select * from dual;
+-----------+--+
| dual.tmp  |
+-----------+--+
| 1         |
+-----------+--+
1 row selected (0.199 seconds)
0: jdbc:hive2://localhost:10000> !q
```

#### 六、参考文章

1. [Hadoop Hive Get Started](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-InstallingHivefromaStableRelease)

