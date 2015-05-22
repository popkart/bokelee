# Kafka
---
Linkedin开源的分布式消息系统。

* 速度快：3个节点组成的集群，1Gb网卡，6核CPU，6个7200RPM SATA，32G内存。发送有效载荷可达70MB/s(有producer和consumer)，70w/s记录，每个payload100字节。单独读或者写可达100-200w+的级别。不同的partition数、不同的producer和consumer数、不同的replication数、是否异步也会影响这个结果，但是总体速度是很大的，吞吐量随着payload的增大而增大。
* 持久化到硬盘（总是这样，而且是O(1)，因为它顺序写，而且绕过了系统级的cache模式），可存储TB级的数据而不影响效率。有备份容错机制（replication）。
* producer采用push，consumer来pull。kafka并不记录consumer读到哪里了。由consumer来记录一个偏移位置（并不是消息id）。所以consumer可以自由地从一个偏移位置开始取记录。因此consumer读完后`并不删除数据`，删除数据是由kafka配置的数据存放时间来决定的，kafka会保留最近一段时间的数据。

下面是几个涉及到的概念：

* topic ：消息的主题，producers可以选择发送消息到一个topic。
* partition ：一个topic可以设置多个partition，producers也可以选择把消息放入一个topic的partitions的任一个（可通过分区选择算法指定）。因此一个partition里消息是有顺序的，但不一定是id连续的。
* replication ：消息备份数量，可以理解为每个partition被复制的个数。每个partition有一个leader server，然后备份的那些server都是followers，leader负责partition的读写请求，followers只是复制leader partition的数据。leader挂了后，一个follower就被升级为leader。一个replication为N的集群允许N-1个节点失败。
* broker ：集群里的一个kafka服务实例（服务器单节点上也可运行多个kafka实例）。
* producer和consumer ：不解释。
* group ：consumers从属group，一个group里的consumers之间是queuing模式（此group订阅了一个topic，该topic里的一条消息只能被group里的一个consumer消费），而group之间是publish-subscribe模式（2个group都订阅了一个topic，这2个group都能消费topic里的一条消息）。

## 杂记
* 压缩  
kafka支持以集合方式发送数据，也支持对消息集合的压缩。Producer可以用Gzip或Snappy格式压缩之，然后在Consumer端解压缩。 具体细节参考：[ https://cwiki.apache.org/confluence/display/KAFKA/Compression](https://cwiki.apache.org/confluence/display/KAFKA/Compression) 

* 可靠性  
producer通过broker的确认保证发送可靠性，consumer由于可以自由重取数据，自己保证处理的可靠性。

### Cousumer API
消费端需要API，生产端没那么复杂。

* High-level Consumer API.
提供了更高的抽象，使用方便简单。
* SimpleConsumer API.
	* 可一个消息读取多次。在一个处理过程中只消费partition中的部分消息。自己添加事务管理机制。
	* 同时也有弊端：必须自己控制offset值。必须找出指定topic的partition的leader broker，且要处理broker的变动。