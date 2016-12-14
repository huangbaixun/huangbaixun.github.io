### 从零开始学习大数据之七：HBase 1.2.3安装及初探

#### 一、环境准备

请参考如下链接，完成Zookeeper和Hadoop安装

- [Zookeeper-3.4.9的安装](http://note.youdao.com/noteshare?id=59fdada492c836913e48874f16d34bce) ,注：本次安装HBase，对之前的安装笔记做了修正，修改了zoo.cfg和/etc/hosts 俩个配置文件，请注意。
- [Hadoop 2.7.3安装](http://note.youdao.com/noteshare?id=e10087e26ea8651007b05eb7d25e5817)

#### 二、下载安装

下载HBase最新稳定版本：hbase-1.2.3到/opt目录
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/stable/hbase-1.2.3-bin.tar.gz
```

解压并重命名文件夹为hbase

```shell
tar -xzvf hbase-1.2.3-bin.tar.gz
mv hbase-1.2.3 hbase
```
#### 三、修改配置

`vim /opt/hbase/conf/hbase-env.sh`
添加如下内容
```shell
export JAVA_HOME=/opt/jdk1.8.0_101/
export HBASE_CLASSPATH=/opt/hadoop/etc/hadoop/
export HBASE_MANAGES_ZK=false
```


`vim /opt/hbase/conf/hbase-site.xml`
添加如下内容
```shell
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://node1:9000/hbase</value>
    </property>
    <property>
        <name>hbase.master</name>
        <value>node1</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>node1,node2,node3</value>
    </property>
    <property>
        <name>zookeeper.session.timeout</name>
        <value>60000000</value>
    </property>
    <property>
        <name>dfs.support.append</name>
        <value>true</value>
    </property>
</configuration>
```
`vim /opt/hbase/conf/regionservers`
添加如下内容
```shell
node2
node3
```

同步安装包到另外俩个节点
```shell
scp -r /opt/hbase node2:/opt/
scp -r /opt/hbase node3:/opt/
```

#### 三、启动集群

在三个节点分别启动Zookeeper

```shell
/opt/zookeeper-3.4.9/&bin/zkServer.sh start
```

启动Hadoop（只需要在主节点node1操作）

```shell
$HADOOP_HOME/sbin/start-all.sh
```

启动HBase（只需要在主节点node1操作）

```shell
hbase/bin/start-hbase.sh
```

#### 四、验证并试玩

4.1 验证进程

主节点执行`jps`，查看相应的进程是否都启动

```shell
[root@node1 opt]# jps
2114 QuorumPeerMain  #Zookeeper进程
2520 ResourceManager #Hadoop进程
3322 Jps
2924 HMaster         #HBase Master进程
2271 NameNode        #Hadoop Master进程
```
从节点执行`jps`
```shell
[root@node2 opt]# jps
2437 HRegionServer  #HBase Slave进程
2072 QuorumPeerMain #Zookeeper进程
2251 NodeManager    #Hadoop slave进程
2140 DataNode       #Hadoop slave进程
2606 Jps
```
4.2 通过HBase shell把玩HBase

4.2.1 进入Hbase shell
```shell
[root@node1 opt]# hbase/bin/hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hbase/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.3, rbd63744624a26dc3350137b564fe746df7a721a4, Mon Aug 29 15:13:42 PDT 2016

hbase(main):001:0> list
TABLE                                                                                                                                                                        
0 row(s) in 0.2890 seconds

=> []
```

4.2.2 创建一个HBase表:huangbaixun_test,列族为cf
```shell
hbase(main):003:0> create 'huangbaixun_test','cf'
0 row(s) in 2.3270 seconds

=> Hbase::Table - huangbaixun_test
```
4.2.3 验证创建的表是否成功

```shell
hbase(main):004:0> list
TABLE                                                                                           
huangbaixun_test                                                                                
1 row(s) in 0.0140 seconds

=> ["huangbaixun_test"]
```

可以看到创建的表了

4.2.4  向表中插入数据

```shell
hbase(main):004:0> put 'huangbaixun_test', 'row1', 'cf:a', 'value1'
hbase(main):004:0> put 'huangbaixun_test', 'row2', 'cf:b', 'value2'
hbase(main):004:0> put 'huangbaixun_test', 'row3', 'cf:c', 'value3'
```

4.2.5 扫描表来查看刚才插入的数据

```shell
hbase(main):004:0> scan 'huangbaixun_test'
ROW                       COLUMN+CELL                                                           
 row1                     column=cf:a, timestamp=1477233090998, value=value1                    
 row2                     column=cf:b, timestamp=1477233160633, value=value2                    
 row3                     column=cf:c, timestamp=1477233177444, value=value3                    
3 row(s) in 0.0530 seconds
```
4.2.6 获取其中一条数据

```shell
hbase(main):001:0> get 'huangbaixun_test','row1'
COLUMN                    CELL                                                                  
 cf:a                     timestamp=1477233090998, value=value1 
```
4.2.7 删除表并退出hbase shell

```shell
hbase(main):004:0> disable 'huangbaixun_test'
0 row(s) in 2.3830 seconds
hbase(main):007:0> drop 'huangbaixun_test'
0 row(s) in 1.2660 seconds

hbase(main):008:0> exit
```

#### 五、结语

从零开始学习大数据将会专注大数据环境的搭建和大数据组件的基本使用；
后期的深入大数据系列将会覆盖组件配置，编程，集性能调优；

#### 六、参考文章

1. [Hadoop+HBase+ZooKeeper分布式集群环境搭建](http://blog.csdn.net/lisonglisonglisong/article/details/46974723)
2. [HBase Quick Start](http://hbase.apache.org/0.94/book/quickstart.html)
