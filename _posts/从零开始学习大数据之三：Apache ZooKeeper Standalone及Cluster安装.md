### 决战大数据之三-Apache ZooKeeper Standalone及Cluster安装及测试

[TOC]

#### Apache ZooKeeper 单机模式安装

#####创建hadoop用户&赋予sudo权限,安全第一:)
> 默认情况行下 CentOS 的group wheel 用的用户拥有 sudo权限

```shell
# useradd hadoop
# passwd hadoop
Changing password for user hadoop.
New password: bigdata123
Retype new password: bigdata123
passwd: all authentication tokens updated successfully.
usermod -aG wheel username
su - hadoop
```
#####安装vim，方便后期更改配置
```
sudo yum install vim
```

####配置 `JAVA_HOME` `vim ~/.bashrc`添加如下代码,保存退出`:wq`，如何安装JDK？请参考[CentOS 7 JDK 8安装](http://note.youdao.com/noteshare?id=b8cac50f3e6a3d904db877b6beb8fa2f)

```shell
JAVA_HOME=/opt/jdk1.8.0_101
PATH=$PATH:$JAVA_HOME/bin
```
测试 
```
$ source ~/.bashrc
# echo $JAVA_HOME
```

#####下载Apache ZooKeeper-3.4.9[^zk] 安装并测试

```shell
$ cd /opt
$ sudo wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
$ sudo tar xzf zookeeper-3.4.9.tar.gz

```
#####下载Apache ZooKeeper 安装并测试
创建配置文件conf/zoo.cfg
```shell
$ cd zookeeper-3.4.9/
$ sudo mkdir data
$ sudo vim conf/zoo.cfg
#添加如下配置到zoo.cfg
tickTime = 2000
dataDir = /opt/zookeeper-3.4.9/data #数据文件路径
clientPort = 2181 #端口
initLimit = 5
syncLimit = 2
```
保存退出，并回到Console，启动Zookeeper

```shell
$ sudo bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.4.9/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
##### 恭喜你，you make it,让我们在启动命令行测试下
```shell
$ sudo bin/zkCli.sh
Connecting to localhost:2181
...
```
就是这么简单我们连接成功

##### 用zkCli创建一个znode `simon`,值为`data 1`
```shell
[zk: localhost:2181(CONNECTED) 8] create /simon 'data 1'
```
##### 用zkCli查看刚才创建的路径
```shell
[zk: localhost:2181(CONNECTED) 9] ls /    
[simon, zookeeper]
```
##### 用zkCli查看刚才创建的路径的值，请注意此时`ctime`和 `mtime`的值是一样的
```shell
[zk: localhost:2181(CONNECTED) 10] get /simon
data 1
cZxid = 0x4
ctime = Sat Sep 10 03:11:27 EDT 2016
mZxid = 0x4
mtime = Sat Sep 10 03:11:27 EDT 2016
pZxid = 0x4
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```
##### 修改 /simon的值,是不是和linux的文件系统很像:),此时ctime和mtime的值不一样的了
```
set /simon 'data 2'
cZxid = 0x4
ctime = Sat Sep 10 03:11:27 EDT 2016
mZxid = 0x5
mtime = Sat Sep 10 03:16:22 EDT 2016
pZxid = 0x4
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```
##### romove刚才创建的znode /simon

```shell
[zk: localhost:2181(CONNECTED) 13] rmr /simon
[zk: localhost:2181(CONNECTED) 14] ls /
[zookeeper]
```
So eeeeeasy~right ,那我开始玩点高级的，搭建集群模式
首先停止单机服务
```shell
[hadoop@note1 zookeeper-3.4.9]$ sudo bin/zkServer.sh stop
[sudo] password for hadoop: 
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.4.9/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```
#####ZooKeeper Cluster配置：
三台CentOS 7的虚拟机运行在VitualBox上面，规划如下

| name  | ip           | myid |
| ----- | ------------ | ---- |
| node1 | 192.168.3.11 | 0    |
| node2 | 192.168.3.12 | 1    |
| node3 | 192.168.3.13 | 2    |

修改note1的域名配置 `sudo vim /etc/hosts`
添加如下内容,并保存退出
> node1 192.168.3.11
> node2 192.168.3.12
> node3 192.168.3.13

修改ZooKeeper的配置

备份之前的单机配置
> cd /opt/zookeeper-3.4.9
> cp conf/zoo.cfg conf/zoo_standalone.cfg

创建log目录
>sudo mkdir /opt/zookeeper-3.4.9/log
>创建myid `sudo vim /opt/zookeeper-3.4.9/myid` insert 1, `:wq`

添加集群配置配置 `sudo vim conf/zoo.cfg`,替换之前的配置为如下内容：
```shell
tickTime=2000
#Replace the value of dataDir with the directory where you would like ZooKeeper to save its data
dataDir=/opt/zookeeper-3.4.9/data
#Replace the value of dataLogDir with the directory where you would like ZooKeeper to log
dataLogDir=/opt/zookeeper-3.4.9/log
clientPort=2181
initLimit=10
syncLimit=5
server.1=192.168.3.11:2888:3888
server.2=192.168.3.12:2888:3888
server.3=192.168.3.13:2888:3888
```


关机 `sudo shutdown -h 0`

##### 快速搭建分布集群的另外的俩个节点
现在是见证奇迹的时刻，使用VitualBox复制功能clone另外俩台虚拟机node2, node3

复制node2

![cloneCentOS](https://github.com/huangbaixun/imgs/blob/master/cloneCentOS.png?raw=true)

复制成功后启动node2

修改hostname:
>hostnamectl set-hostname node2 --static
>修改静态IP
>vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
>修改内容如下，保存退出
```shell
#指定的静态ip,node3 改为 IPADDR=192.168.3.13
IPADDR=192.168.3.12
#MAC地址 node3 改为 83179c2f-b512-4c1a-8358-2ff4e823d956
UUID=83179c2f-b512-4c1a-8358-2ff4e823d955
```
重启机器 `reboot now` 现在可以使用新的IP登陆：`192.168.3.12`

创建note3，参考note2

#启动分布式集群node1，node2，node3

修改 node2,和node3的zookeeper的myid

```shell
#for node2 echo "3"
echo "2" > /opt/zookeeper-3.4.9/data/myid
```
启动各个节点ZkSerer(root账号)：
```shell
# cd /opt/zookeeper-3.4.9/ &&bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.4.9/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
可以查看各个启动日志：

```shell
# more zookeeper.out 
2016-09-10 04:39:03,077 [myid:] - INFO  [main:QuorumPeerConfig@124] - Reading configuration from: /opt/zookeeper-3.4.9/bin/../conf/zoo.cfg
2016-09-10 04:39:03,105 [myid:] - INFO  [main:QuorumPeer$QuorumServer@149] - Resolved hostname: 192.168.3.13 to address: /192.168.3.13
2016-09-10 04:39:03,106 [myid:] - INFO  [main:QuorumPeer$QuorumServer@149] - Resolved hostname: 192.168.3.12 to address: /192.168.3.12
2016-09-10 04:39:03,106 [myid:] - INFO  [main:QuorumPeer$QuorumServer@149] - Resolved hostname: 192.168.3.11 to address: /192.168.3.11
2016-09-10 04:39:03,106 [myid:] - INFO  [main:QuorumPeerConfig@352] - Defaulting to majority quorums
2016-09-10 04:39:03,108 [myid:3] - INFO  [main:DatadirCleanupManager@78] - autopurge.snapRetainCount set to 3
2016-09-10 04:39:03,108 [myid:3] - INFO  [main:DatadirCleanupManager@79] - autopurge.purgeInterval set to 0
2016-09-10 04:39:03,108 [myid:3] - INFO  [main:DatadirCleanupManager@101] - Purge task is not scheduled.
2016-09-10 04:39:03,119 [myid:3] - INFO  [main:QuorumPeerMain@127] - Starting quorum peer
2016-09-10 04:39:03,136 [myid:3] - INFO  [main:NIOServerCnxnFactory@89] - binding to port 0.0.0.0/0.0.0.0:2181
2016-09-10 04:39:03,141 [myid:3] - INFO  [main:QuorumPeer@1019] - tickTime set to 2000
2016-09-10 04:39:03,141 [myid:3] - INFO  [main:QuorumPeer@1039] - minSessionTimeout set to -1
2016-09-10 04:39:03,141 [myid:3] - INFO  [main:QuorumPeer@1050] - maxSessionTimeout set to -1
2016-09-10 04:39:03,141 [myid:3] - INFO  [main:QuorumPeer@1065] - initLimit set to 10
```
查看各个节点的zk的状态,可以看到node2目前是leader节点
```shell
[root@node2 zookeeper-3.4.9]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: leader
```
自此我们已经完成Zookeeper Cluster的搭建，To be contiue

Have a good weekend~

有任何问题可以联系我： email:huangbaixun(at)outlook.com

[^zk]: 当前最新的稳定版
参考：http://www.tutorialspoint.com/zookeeper/zookeeper_cli.htm
参考：
http://blog.csdn.net/lisonglisonglisong/article/details/46974723


