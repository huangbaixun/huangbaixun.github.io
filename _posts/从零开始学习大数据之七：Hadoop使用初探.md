### 从零开始学习大数据之七：HDFS & YARN使用初探

#### 一、写在前面的话

通过之前[从零开始学习大数据之六](http://note.youdao.com/noteshare?id=e10087e26ea8651007b05eb7d25e5817)我们已经完成一个真正Hadoop分布式环境的搭建，目前Hadoop 分布式文件系统：HDFS 和分布式计算：YARN 已经展现在我们面前，那让我们来玩玩这俩个新玩具；

#### 二、Hadoop环境变量配置

`vim ~/.bashrc`

添加如下内容

```shell
#/opt/hadoop 为上一节Hadoop的安装根路径
export HADOOP_HOME=/opt/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_INSTALL=$HADOOP_HOME
```

环境变量生效 `su -`

验证配置是否成功
```shell
[root@node1 ~]# hadoop version
Hadoop 2.7.3
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r baa91f7c6bc9cb92be5982de4719c1c8af91ccff
Compiled by root on 2016-08-18T01:41Z
Compiled with protoc 2.5.0
From source with checksum 2e4ce5f957ea4db193bce3734ff29ff4
This command was run using /opt/hadoop/share/hadoop/common/hadoop-common-2.7.3.jar
```
我们可以看到安装的hadoop版本是2.7.3

#### 三、启动HDFS和YARN
```shell
[root@node1 ~]# $HADOOP_HOME/sbin/start-all.sh
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
16/10/21 09:19:46 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [node1]
node1: starting namenode, logging to /opt/hadoop/logs/hadoop-root-namenode-node1.out
node3: starting datanode, logging to /opt/hadoop/logs/hadoop-root-datanode-node3.out
node2: starting datanode, logging to /opt/hadoop/logs/hadoop-root-datanode-node2.out
16/10/21 09:19:59 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
starting yarn daemons
starting resourcemanager, logging to /opt/hadoop/logs/yarn-root-resourcemanager-node1.out
node2: starting nodemanager, logging to /opt/hadoop/logs/yarn-root-nodemanager-node2.out
node3: starting nodemanager, logging to /opt/hadoop/logs/yarn-root-nodemanager-node3.out
```

尼玛，还有WARN：*WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable*，洁癖发作，立马google，找到[解决方案](http://stackoverflow.com/questions/19943766/hadoop-unable-to-load-native-hadoop-library-for-your-platform-warning)：

添加如下代码`vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh`

```shell
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=/opt/hadoop/lib/native"
```

在执行Hadoop相关的命令，顿时感觉清净多了~~

#### 四、HDFS牛刀小试

4.1 查看HDFS文件

```shell
[root@node1 hadoop]# hdfs dfs -ls /
[root@node1 hadoop]# 
```
4.2 啥都没有，先创建几个文件夹吧，不会？*搞linux的都喜欢man*，执行如下帮助命令

```shell
[root@node1 hadoop]# hdfs dfs -help
Usage: hadoop fs [generic options]
	[-appendToFile <localsrc> ... <dst>]
	[-cat [-ignoreCrc] <src> ...]
	[-checksum <src> ...]
	[-chgrp [-R] GROUP PATH...]
	[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
	[-chown [-R] [OWNER][:[GROUP]] PATH...]
	[-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
	[-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-count [-q] [-h] <path> ...]
	[-cp [-f] [-p | -p[topax]] <src> ... <dst>]
	[-createSnapshot <snapshotDir> [<snapshotName>]]
	[-deleteSnapshot <snapshotDir> <snapshotName>]
	[-df [-h] [<path> ...]]
	[-du [-s] [-h] <path> ...]
	[-expunge]
	[-find <path> ... <expression> ...]
	[-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
	[-getfacl [-R] <path>]
	[-getfattr [-R] {-n name | -d} [-e en] <path>]
	[-getmerge [-nl] <src> <localdst>]
	[-help [cmd ...]]
	[-ls [-d] [-h] [-R] [<path> ...]]
	[-mkdir [-p] <path> ...]
	[-moveFromLocal <localsrc> ... <dst>]
	[-moveToLocal <src> <localdst>]
	[-mv <src> ... <dst>]
	[-put [-f] [-p] [-l] <localsrc> ... <dst>]
	[-renameSnapshot <snapshotDir> <oldName> <newName>]
	[-rm [-f] [-r|-R] [-skipTrash] <src> ...]
	[-rmdir [--ignore-fail-on-non-empty] <dir> ...]
	[-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
	[-setfattr {-n name [-v value] | -x name} <path>]
	[-setrep [-R] [-w] <rep> <path> ...]
	[-stat [format] <path> ...]
	[-tail [-f] <file>]
	[-test -[defsz] <path>]
	[-text [-ignoreCrc] <src> ...]
	[-touchz <path> ...]
	[-truncate [-w] <length> <path> ...]
	[-usage [cmd ...]]
```

好熟悉的感觉，和Linux的文件系统很像吗？:),来先创建一个文件夹：XX到此一游

```shell
[root@node1 hadoop]# hdfs dfs -mkdir  -p /user/huangbaixun
```
检查一下是否创建成功

```shell
[root@node1 hadoop]# hdfs dfs -ls /user/
Found 1 items
drwxr-xr-x   - root supergroup          0 2016-10-21 09:41 /user/huangbaixun
```
4.3 OK，搞定了，放个文件进去  ~/test.txt,文件内容如下：
```
Game of Thrones
American drama series
Rotten Tomatoes
George R.R. Martin's best-selling book series "A Song of Ice and Fire" is brought to the screen as HBO sinks its considerable storytelling teeth into the medieval fantasy epic. It's the depiction of two powerful families -- kings and queens, knights and renegades, liars and honest men -- playing a deadly game for control of the Seven Kingdoms of Westeros, and to sit atop the Iron Throne. Martin is credited as a co-executive producer and one of the writers for the series, which was filmed in Northern Ireland and Malta.
Theme song: Game of Thrones Theme
Awards: Primetime Emmy Award for Outstanding Drama Series, more
Writers: George R. R. Martin, David Benioff, D. B. Weiss, Vanessa Taylor, Bryan Cogman, Jane Espenson, Dave Hill
```
放置该文件到HDFS并验证是否放置成功：

```shell
root@node1 ~]# hdfs dfs -put ~/test.txt /user/huangbaixun
[root@node1 ~]# hdfs dfs -cat /user/huangbaixun/*
Game of Thrones
American drama series
Rotten Tomatoes
George R.R. Martin's best-selling book series "A Song of Ice and Fire" is brought to the screen as HBO sinks its considerable storytelling teeth into the medieval fantasy epic. It's the depiction of two powerful families -- kings and queens, knights and renegades, liars and honest men -- playing a deadly game for control of the Seven Kingdoms of Westeros, and to sit atop the Iron Throne. Martin is credited as a co-executive producer and one of the writers for the series, which was filmed in Northern Ireland and Malta.
Theme song: Game of Thrones Theme
Awards: Primetime Emmy Award for Outstanding Drama Series, more
Writers: George R. R. Martin, David Benioff, D. B. Weiss, Vanessa Taylor, Bryan Cogman, Jane Espenson, Dave Hill
```

就是这么简单，你已经学会使用HDFS了。


#### 五、YARN初探

学习MapReduce的最经典的例子莫过于统计词频，我们使用Hadoop自动的统计词频工具包（wordcount）
需要统计的文本正好是我们刚才放上去的/user/huangbaixun/test.txt 我们希望统计的结果输出到
/output

好吧，让我一起来见证这个一时刻的到来：

```shell
[root@node1 ~]# hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /user/huangbaixun /output
16/10/21 09:55:32 INFO client.RMProxy: Connecting to ResourceManager at node1/192.168.3.11:8032
16/10/21 09:55:33 INFO input.FileInputFormat: Total input paths to process : 1
16/10/21 09:55:33 INFO mapreduce.JobSubmitter: number of splits:1
16/10/21 09:55:33 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1477056624379_0001
16/10/21 09:55:34 INFO impl.YarnClientImpl: Submitted application application_1477056624379_0001
16/10/21 09:55:34 INFO mapreduce.Job: The url to track the job: http://node1:8088/proxy/application_1477056624379_0001/
16/10/21 09:55:34 INFO mapreduce.Job: Running job: job_1477056624379_0001
16/10/21 09:55:43 INFO mapreduce.Job: Job job_1477056624379_0001 running in uber mode : false
16/10/21 09:55:43 INFO mapreduce.Job:  map 0% reduce 0%
16/10/21 09:55:48 INFO mapreduce.Job:  map 100% reduce 0%
16/10/21 09:55:55 INFO mapreduce.Job:  map 100% reduce 100%
16/10/21 09:55:56 INFO mapreduce.Job: Job job_1477056624379_0001 completed successfully
16/10/21 09:55:56 INFO mapreduce.Job: Counters: 49
...
```

大家是不是对统计结果很期待，刚才学习了HDFS，应该知道怎么查看统计的结果了吧？其实我也想看看

```shell
[root@node1 ~]# hdfs dfs -cat /output/*
American	1
Award	1
...
```

#### 六 、结语

至此我们已经初步学会如何使用HDFS和YARN来存储文件和技术，BUT这只是万里长征的第一步:)；

#### 参考资料

- [Hadoop Startup](https://hadoop.apache.org/docs/r1.2.1/cluster_setup.html#Hadoop+Startup)
- [Hadoop - Enviornment Setup](https://www.tutorialspoint.com/hadoop/hadoop_enviornment_setup.htm)

