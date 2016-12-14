### 从零开始学习大数据之五：使用ZooKeeper实现集群选举的一个简单实现(译)

#### 我们看下使用ZooKeeper的集合是如何选举一个主节点？ 假设一个集群有N个节点，下面就是一个主节点选举的流程（也可以称为简单的实现）


* 所有的节点连接ZooKeeper后都会创建一个连续，临时的（*当这个节点和zk的心跳停止后，该节点会被自动删除：译者注*）znode 并且在一个相同的路径：/app/leader_election/guid_.
* ZooKeeper的集合将会附加10位的连续数字字符串在各个节点创建的znode的路径上面，比如：/app/leader_election/guid_0000000001, /app/leader_election/guid_0000000002, 以此类推
* 创建的znode最小值的节点将会是主节点（比如：/app/leader_election/guid_0000000001），其他节点都会是从节点：/app/leader_election/guid_0000000002~/app/leader_election/guid_000000000N
* 各个从节点监控一个线性紧挨着比他小的znode，比如：创建znode /app/leader_election/guid_0000000008的节点将会监控（watch）znode /app/leader_election/guid_0000000007，相同的创建znode /app/leader_election/guid_0000000007的节点将会监控znode /app/leader_election/guid_0000000006。
* 如下主节点宕机了（节点N）， 那么对应的znode  /app/leader_election/guid_000000000N 将会被删除（比如：/app/leader_election/guid_0000000001）
* 紧挨着这个节点的节点（比如：/app/leader_election/guid_0000000002）通过监控znode知道这个主节点已经被移除了
* 那么这个节点会检查是否有比他创建znode更小的节点？如果没有，他将会承担主节点的任务，否则，它将会接受znode比他小的节点为主节点
* 相同的， 其他从节点将会选举znode最小的节点为主节点

如果我们自己从0开始实行选举，那是非常的复杂，但是你会发现使用zookeeper提供的服务来实现，却又相当的简单。

原文 [Zookeeper - Overview](https://www.tutorialspoint.com/zookeeper/zookeeper_leader_election.htm)

