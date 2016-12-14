###  从零开始学习大数据之六 - Hadoop分布式集群安装

#### 一、环境说明

集群规划：三台虚拟主机：一主俩从。

软件配置如下：

操作系统：OS : CentOs 7 

Hadoop ：Hadoop :2.7.3

| 主从     | 节点名称  | 节点IP         | 账号   | 密码         |
| ------ | ----- | ------------ | ---- | ---------- |
| Master | node1 | 192.168.3.11 | root | bigdata123 |
| Slave  | node2 | 192.168.3.12 | root | bigdata123 |
| Slave  | node3 | 192.168.3.13 | root | bigdata123 |

#### 二、准备工作

2.1.1 伪分布式集群搭建：请参考[从零开始学习大数据之一：伪分布式环境搭建](http://note.youdao.com/noteshare?id=de2d5e8e1c0d20be2df4f0ca049fe00f)

 2.1.2 JDK 1.8安装：请参考 [从零开始学习大数据之二：CentOS 7 最新JDK 8安装](http://note.youdao.com/noteshare?id=b8cac50f3e6a3d904db877b6beb8fa2f)

修改配置文件 `vim /etc/profile`

```shell
export JAVA_HOME=/opt/jdk1.8.0_101/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
使之生效

```shell
$ source /etc/profile
```

2.2 添加host映射关系

分别在三个节点修改host映射关系 `vim /etc/hosts`,添加如下内容

```shell
192.168.3.11 node1
192.168.3.12 node2
192.168.3.13 node3
```

2.3 集群之间无密码登录

2.3.1 设置master无密码登录

主要有三步：：①生成公钥和私钥、②导入公钥到认证文件、③更改权限

```shell
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 700 ~/.ssh && chmod 600 ~/.ssh/*
```

测试，第一次登录可能需要yes确认，之后就可以直接登录了

```shell
[root@node1 ~]# ssh localhost
Last login: Wed Oct 19 09:54:38 2016 from node3
```

对从节点node2，node3，进行无密码登录设置，操作同上

2.3.2 设置Master->Slave 的无密码登录

> cat ~/.ssh/id_rsa.pub | ssh root@node2 'cat - >> ~/.ssh/authorized_keys'

> cat ~/.ssh/id_rsa.pub | ssh root@node3 'cat - >> ~/.ssh/authorized_keys'

测试：

```shell
[root@node1 ~]# ssh node2
Last login: Wed Oct 19 09:50:06 2016 from node1
[root@node2 ~]# exit
logout
Connection to node2 closed.
[root@node1 ~]# ssh node3
Last login: Wed Oct 19 09:17:57 2016 from 192.168.3.19
```

2.3.3 设置Slave 到Master的无密码登录

分别在node2、node3上执行：

> $ cat ~/.ssh/id_rsa.pub | ssh node1 'cat - >> ~/.ssh/authorized_keys'

####三、 Hadoop集群安装配置

Hadoop 将会安装在`/opt` 目录下，创建此目录并下载最新稳定版[Hadoop]() ,

```shell
cd /opt/
wget wget http://211.162.74.231:9011/apache.fayea.com/c3pr90ntc0td/hadoop/common/stable2/hadoop-2.7.3.tar.gz
```
解压 `tar -xzf hadoop-2.7.3.tar.gz`

修改文件夹名字为hadoop `mv hadoop-2.7.3 hadoop`



####四、修改配置文件

配置文件都在 `/opt/hadoop/etc/`目录下，首先进入目录`cd /opt/hadoop/etc/`

4.1 修改 core-site.xml 
```shell
<configuration>  
    <property>  
        <name>hadoop.tmp.dir</name>  
        <value>/opt/hadoop/tmp</value>  
        <description>Abase for other temporary directories.</description>  
    </property>  
    <property>  
        <name>fs.defaultFS</name>  
        <value>hdfs://node1:9000</value>  
    </property>  
    <property>  
        <name>io.file.buffer.size</name>  
        <value>4096</value>  
    </property>  
</configuration>
```

4.2 修改 hdfs-site.xml 
```shell
<configuration>  
    <property>  
        <name>dfs.nameservices</name>  
        <value>hadoop-cluster1</value>  
    </property>  
    <property>  
        <name>dfs.namenode.secondary.http-address</name>  
        <value>node1:50090</value>  
    </property>  
    <property>  
        <name>dfs.namenode.name.dir</name>  
        <value>file:///opt/hadoop/dfs/name</value>  
    </property>  
    <property>  
        <name>dfs.datanode.data.dir</name>  
        <value>file:///opt/hadoop/dfs/data</value>  
    </property>  
    <property>  
        <name>dfs.replication</name>  
        <value>2</value>  
    </property>  
    <property>  
        <name>dfs.webhdfs.enabled</name>  
        <value>true</value>  
    </property>  
</configuration>
```

4.3 修改 mapred-site.xml
```shell
<configuration>  
    <property>  
        <name>mapreduce.framework.name</name>  
        <value>yarn</value>  
    </property>  
    <property>  
        <name>mapreduce.jobtracker.http.address</name>  
        <value>node1:50030</value>  
    </property>  
    <property>  
        <name>mapreduce.jobhistory.address</name>  
        <value>node1:10020</value>  
    </property>  
    <property>  
        <name>mapreduce.jobhistory.webapp.address</name>  
        <value>node1:19888</value>  
    </property>  
</configuration>
```

4.4 修改 yarn-site.xml

```shell
<configuration>  
    <property>  
        <name>yarn.nodemanager.aux-services</name>  
        <value>mapreduce_shuffle</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.address</name>  
        <value>node1:8032</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.scheduler.address</name>  
        <value>node1:8030</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.resource-tracker.address</name>  
        <value>node1:8031</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.admin.address</name>  
        <value>node1:8033</value>  
    </property>  
    <property>  
        <name>yarn.resourcemanager.webapp.address</name>  
        <value>node1:8088</value>  
    </property>  
</configuration>
```
4.4 修改slaves：

```shell
node2
node3
```
4.5 修改JAVA_HOME
分别在文件hadoop-env.sh和yarn-env.sh中添加JAVA_HOME配置
`vim hadoop-env.sh` `I`
>export JAVA_HOME=/opt/jdk1.8.0_101/

`vim yarn-env` `I`
>export JAVA_HOME=/opt/jdk1.8.0_101/

4.6 copy代码和配置到俩台从节点, 由于文件比较多，整个过程大概需要1分钟左右

```shell
scp -r /opt/hadoop node2:/opt/
scp -r /opt/hadoop node3:/opt/
```

####五、格式化文件系统
>cd /opt/hadoop
>bin/hdfs namenode -format

####六、启动 Hadoop 并验证

6.1 启动HDFS

```shell
[root@node1 hadoop]# ./sbin/start-dfs.sh
Starting namenodes on [node1]
node1: starting namenode, logging to /opt/hadoop/logs/hadoop-root-namenode-node1.out
node2: starting datanode, logging to /opt/hadoop/logs/hadoop-root-datanode-node2.out
node3: starting datanode, logging to /opt/hadoop/logs/hadoop-root-datanode-node3.out
```
6.2 启动YARN

```shell
[root@node1 hadoop]# ./sbin/start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /opt/hadoop/logs/yarn-root-resourcemanager-node1.out
node3: starting nodemanager, logging to /opt/hadoop/logs/yarn-root-nodemanager-node3.out
node2: starting nodemanager, logging to /opt/hadoop/logs/yarn-root-nodemanager-node2.out
```

6.3 验证启动进程

6.3.1 通过命令行验证Hadoop启动是否正常

```shell
root@node1 hadoop]# jps
2433 ResourceManager
2690 Jps
2188 NameNode
```
从上面返回的结果可以看出：

HDFS已经成功启动：NameNode
YARN也成功启动：ResourceManager

6.3.2 通过界面验证Hadoop是否启动成功

在浏览器访问HDFS管理界面：

http://192.168.3.11:50070/dfshealth.html#tab-overview

![HDFS Manager](https://raw.githubusercontent.com/huangbaixun/imgs/master/6-hdfs.png)

在浏览器访问YARN管理界面：

http://192.168.3.11:8088/cluster

![HDFS Manager](https://raw.githubusercontent.com/huangbaixun/imgs/master/6-yarn.png)

是否还是熟悉味道　：）

6.3.2 关闭 Hadoop

- 关闭 YARN `./sbin/stop-yarn.sh`
- 关闭HDFS`./sbin/stop-dfs.sh`


#### 参考：

- [Hadoop Cluster Setup](https://hadoop.apache.org/docs/r1.2.1/cluster_setup.html)
- [Hadoop-2.5.1集群安装配置笔记](http://blog.csdn.net/greensurfer/article/details/39450369#)
- [Hadoop+HBase+ZooKeeper分布式集群环境搭建](http://blog.csdn.net/lisonglisonglisong/article/details/46974723)