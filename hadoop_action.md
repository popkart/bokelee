#Hadoop Action
###MR 任务优化
#### 计算性能优化&I/O优化
1. 任务调度，Hadoop尽量分配InputSplit所在的机器执行任务，减少I/O消耗。
2. 数据预处理：适合大文件，大量小文件应先合并。Map时间最好1分钟左右，设置InputSplit输入大小（设置块大小）设置Map任务数量来调节。使用CombineInputFormat或者自己实现输入格式类来自动合并文件。
3. 设置Map/Reduce任务数量。Reduce应该为Reduce任务槽的0.95（一个任务失败后立即有其他槽来执行）或1.75（确保执行速度快的机器可以得到更多任务）倍。
4. Combine过程。本地处理下，减少网络传输。
5. 压缩。但是最终结果如果压缩，对下一个读取需要耗费点事。常见对Map输出压缩，设置mapred.compress.map.output=true.mapred.map.output.compression.codec设置压缩格式。
6. 自定义数据类型，自定义comparator来直接对二进制数据流比较。

####Hadoop Streaming
Streaming提供了一个API，允许用任何脚本语言写`Map和Reduce函数`，只要语言支持UNIX标准输入流作为输入，标准输出流作为输出。这里只是把这个java的处理过程调为其他语言的过程而已。
还有C++等其他的语言，使用的是Pipe方式。

####Shuffle
包括Map和Reduce两端的内容，Map端是Partition、Sort（的啥？）、Spill（分割的啥），然后按照划分（同一个Partition，先本地合并）Merge写入磁盘；Reduce端对各个Map送来的属于同一个划分进行合并，然后Sort，然后给Reduce处理。

#####Shuffle-Map端
collect函数。Map函数的输出内存缓冲区是一个环形结构，当填到一定阈值时，需要spill到磁盘。但是可以一边spill一边往缓冲区写。
#####Shuffle-Reduce端
复制各个Map文件到Reduce，同时进行Merge（保持顺序，排序）。如果使用MR默认Shuffle，那么这个Reduce前面的必然是排好序的。当然如果只求集合并集，是不需要排序的。通过合理设置`io.sort.*`属性来减少写操作次数（Map Spill的次数，具体就是增大io.sort.mb，缓冲区大小）
#####推测式执行
默认开启。如果发现一个运行缓慢的task，则会同时再起个相同任务，哪个先完成就ok。缺陷：如果任务确实运行慢，则多开一份还是慢。根据实际情况关掉。
#####JVM重用
Map、Reduce任务都是在不同的JVM上执行。如果运行时间很短太浪费了，可以在一个Map之类运行完后，不销毁虚拟机，然后其他Map在其上运行。可以设置每个JVM resue次数。
#####任务执行环境
Map任务可以知道自己处理的文件名称、自己在急群中的ID等。