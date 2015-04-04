# ZooKeeper: A Distributed Coordination Service for Distributed Applications
---
## 目标
* 提供一个类似普通文件系统的、异构的分布式共享namespsce，分布式应用协调服务。提供简单原始的功能，只要多于一半的server可用服务就可用。leader提供写数据和同步（不向client提供读数据），失效后会在follower中重新选举。
* 数据保存在内存中，因此能高throughout，低延迟。
* 提供简单的API。

```
多节点组成一个团。
读多写少，r:w=10:1。
类似普通文件系统，但是目录节点也能存数据。
事务有序，有序号。
```
`znode`是namespace中的`data node`。每个znode保存了一个状态结构，包括version numbers for date changes，ACL changes， timestamps, 数据更新的时候版本号会增加。  
暂时节点：这些节点在创建节点的session active时才在。否则被删除。
watch：支持client观察某个znode，当znode change时，client会得到通知。
## 特性：
* 顺序持久化：来自clients的更新是有序的。
* 原子性：更新只有成功和失败2种状态
* 单系统映像：client不论连接哪个节点，看到的内容都是一样的。
* 可靠性：当其接受了一个更新后，这个更新的内容会持续到下次别的更新覆盖它之前。
* 及时性：客户端视图会在很短的时间内及时更新到最新。
## API操作
* create:在namespace上某个地方创建一个z节点。
* delete：删除一个z节点。
* exists：判断一个z节点是否存在。
* get data：从一个z节点上读数据。
* set data：
* get children：检索znode的所有子节点。
* sync：

## 安装配置
* 解压
* 配置`conf/zoo.cfg`
* 启动
* 有`zkCli.sh`可以登录上去做一些操作。  
配置里面最后配置server的地方`server.1=host:2888:3888`，第一个端口是zk节点间互相通信的port，第二个是选举leader用的。均使用TCP协议。可以在单机上部署多个zk多个端口测试。
## 应用场景
* ___数据发布订阅___ &nbsp;&nbsp;将配置放在某个znode上，clients在节点注册一个watcher，每次配置更新后应用会得到通知。
* ___分布式锁___ &nbsp;&nbsp;一个用户创建一个节点znode作为锁。另一个用户来检测这个节点，如果存在则说明别的用户已锁住。和多线程的critical section临界区类似。
* ___集群管理___ &nbsp;&nbsp;各个机器均创建一个节点，写入自己的状态，监控机器watch这些znode，进行协调。
* 
## 其他内容
Our protocol assumes that we can construct point-to-point FIFO channels between the servers.   
选举2种算法：
* LeaderElection
* FastLeaderElection （AuthFastLeaderElection，用UDP，在Fast基础上增加了IP spoofing欺骗）。
