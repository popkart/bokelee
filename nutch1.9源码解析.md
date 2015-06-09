# Nutch1.9源码解析
---
## 概述
这是本人学习Nutch源码的一点心得和笔记，希望能在看源码的过程中，总结到爬虫各个关键问题的处理方法。在学习的过程中参考了<http://blog.csdn.net/spacewalkman/>的”Nutch源码分析“系列文章。

Nutch是Doug Cutting发起的爬虫项目（刚开始是搜索引擎，后来发展为专注爬虫），基于HDFS和Hadoop框架，事实上HDFS和Hadoop就是因为这个项目而被写出来的。 

Nutch1.x版本的后端存储是HDFS，2.x版本把存储层抽象了出来，通过Gora实现了多后端存储，可以把结果存储在nosqldb/关系数据库等不同类型的存储系统上。2.x版本还把URL过滤、索引去重等公共模块抽取出来了，可以直接用在自己的爬虫上。抽取上，只是抽取出NutchDocument，然后索引、搜索的功能交给第三方做，更专注爬取功能。还有RESTFUL接口等功能<sup>[NutchRoadMap](http://wiki.apache.org/nutch/Nutch2Roadmap)</sup> 。

我们先从1.x看起。
## Nutch工程引入
nutch采用ant编译，使用ivy管理项目依赖。安装ant之后，在nutch根目录下执行`ant eclipse`生成eclipse所需要的`.classpath和.project`文件；或者执行`ant deploy`会生成maven项目，会在nutch的根目录下生成pom文件。  
注意：nutch的plugin项目并没有在生成的pom里配置，因此用maven的话不太爽，我用的是生成.project文件那种方式，然后导入到eclipse里<sup>[[reference]](http://blog.csdn.net/spacewalkman/article/details/41700593)</sup>。
## 流程和中间目录
### 流程图
Nutch的爬取流程由一个个MR程序构成，而驱动这个流程的是`./bin`目录下的脚本`nutch`和`crawl`，nutch脚本里定义了爬取流程的每一步需要调用的MR程序，而crawl脚本负责调用nutch，把这些步骤串起来。 建议先读一下这2个脚本的代码，会对Nutch的工作流程有一个大概的认识。
![nutch基本流程图](img/nutchcircle.png)  
注意，标数字的代表nutch的处理过程（流程图里应该用非正方形的平行四边形表示），其他是存储的表示（比如CrawlDb指的是存储URL相关的文件）。“1、2、3、4”是Nutch爬取的circle，5、6等是为搜索引擎准备的索引和链接关系分析模块。  
注意Inject这个过程在这个图里没有画出来，它是Nutch第一次启动时候，向crawldb库里注入初始种子链接用的。  

nutch脚本内的命令列表：

```
 echo "Usage: nutch COMMAND"
  echo "where COMMAND is one of:"
  echo "  readdb            read / dump crawl db"
  echo "  mergedb           merge crawldb-s, with optional filtering"
  echo "  readlinkdb        read / dump link db"
  echo "  inject            inject new urls into the database"
  echo "  generate          generate new segments to fetch from crawl db"
  echo "  freegen           generate new segments to fetch from text files"
  echo "  fetch             fetch a segment's pages"
  echo "  parse             parse a segment's pages"
  echo "  readseg           read / dump segment data"
  echo "  mergesegs         merge several segments, with optional filtering and slicing"
  echo "  updatedb          update crawl db from segments after fetching"
  echo "  invertlinks       create a linkdb from parsed segments"
  echo "  mergelinkdb       merge linkdb-s, with optional filtering"
  echo "  index             run the plugin-based indexer on parsed segments and linkdb"
  echo "  dedup             deduplicate entries in the crawldb and give them a special status"
  echo "  solrindex         run the solr indexer on parsed segments and linkdb - DEPRECATED use the index command instead"
  echo "  solrdedup         remove duplicates from solr - DEPRECATED use the dedup command instead"
  echo "  solrclean         remove HTTP 301 and 404 documents from solr - DEPRECATED use the clean command instead"
  echo "  clean             remove HTTP 301 and 404 documents and duplicates from indexing backends configured via plugins"
  echo "  parsechecker      check the parser for a given url"
  echo "  indexchecker      check the indexing filters for a given url"
  echo "  domainstats       calculate domain statistics from crawldb"
  echo "  webgraph          generate a web graph from existing segments"
  echo "  linkrank          run a link analysis program on the generated web graph"
  echo "  scoreupdater      updates the crawldb with linkrank scores"
  echo "  nodedumper        dumps the web graph's node scores"
  echo "  plugin            load a plugin and run one of its classes main()"
  echo "  junit             runs the given JUnit test"
  echo " or"
  echo "  CLASSNAME         run the class named CLASSNAME"
  echo "Most commands print help when invoked w/o parameters."
```

脚本里的命令和对应调用的类：

```
# figure out which class to run
if [ "$COMMAND" = "crawl" ] ; then
  echo "Command $COMMAND is deprecated, please use bin/crawl instead"
  exit -1
elif [ "$COMMAND" = "inject" ] ; then
  CLASS=org.apache.nutch.crawl.Injector
elif [ "$COMMAND" = "generate" ] ; then
  CLASS=org.apache.nutch.crawl.Generator
elif [ "$COMMAND" = "freegen" ] ; then
  CLASS=org.apache.nutch.tools.FreeGenerator
elif [ "$COMMAND" = "fetch" ] ; then
  CLASS=org.apache.nutch.fetcher.Fetcher
elif [ "$COMMAND" = "parse" ] ; then
  CLASS=org.apache.nutch.parse.ParseSegment
elif [ "$COMMAND" = "readdb" ] ; then
  CLASS=org.apache.nutch.crawl.CrawlDbReader
elif [ "$COMMAND" = "mergedb" ] ; then
  CLASS=org.apache.nutch.crawl.CrawlDbMerger
elif [ "$COMMAND" = "readlinkdb" ] ; then
  CLASS=org.apache.nutch.crawl.LinkDbReader
elif [ "$COMMAND" = "readseg" ] ; then
  CLASS=org.apache.nutch.segment.SegmentReader
elif [ "$COMMAND" = "mergesegs" ] ; then
  CLASS=org.apache.nutch.segment.SegmentMerger
elif [ "$COMMAND" = "updatedb" ] ; then
  CLASS=org.apache.nutch.crawl.CrawlDb
elif [ "$COMMAND" = "invertlinks" ] ; then
  CLASS=org.apache.nutch.crawl.LinkDb
elif [ "$COMMAND" = "mergelinkdb" ] ; then
  CLASS=org.apache.nutch.crawl.LinkDbMerger
elif [ "$COMMAND" = "solrindex" ] ; then
  CLASS="org.apache.nutch.indexer.IndexingJob -D solr.server.url=$1"
  shift
elif [ "$COMMAND" = "index" ] ; then
  CLASS=org.apache.nutch.indexer.IndexingJob
elif [ "$COMMAND" = "solrdedup" ] ; then
  echo "Command $COMMAND is deprecated, please use dedup instead"
  exit -1
elif [ "$COMMAND" = "dedup" ] ; then
  CLASS=org.apache.nutch.crawl.DeduplicationJob
elif [ "$COMMAND" = "solrclean" ] ; then
  CLASS="org.apache.nutch.indexer.CleaningJob -D solr.server.url=$2 $1"
  shift; shift
elif [ "$COMMAND" = "clean" ] ; then
  CLASS=org.apache.nutch.indexer.CleaningJob
elif [ "$COMMAND" = "parsechecker" ] ; then
  CLASS=org.apache.nutch.parse.ParserChecker
elif [ "$COMMAND" = "indexchecker" ] ; then
  CLASS=org.apache.nutch.indexer.IndexingFiltersChecker
elif [ "$COMMAND" = "domainstats" ] ; then 
  CLASS=org.apache.nutch.util.domain.DomainStatistics
elif [ "$COMMAND" = "webgraph" ] ; then
  CLASS=org.apache.nutch.scoring.webgraph.WebGraph
elif [ "$COMMAND" = "linkrank" ] ; then
  CLASS=org.apache.nutch.scoring.webgraph.LinkRank
elif [ "$COMMAND" = "scoreupdater" ] ; then
  CLASS=org.apache.nutch.scoring.webgraph.ScoreUpdater
elif [ "$COMMAND" = "nodedumper" ] ; then
  CLASS=org.apache.nutch.scoring.webgraph.NodeDumper
elif [ "$COMMAND" = "plugin" ] ; then
  CLASS=org.apache.nutch.plugin.PluginRepository
elif [ "$COMMAND" = "junit" ] ; then
  CLASSPATH="$CLASSPATH:$NUTCH_HOME/test/classes/"
  CLASS=org.junit.runner.JUnitCore
else
  CLASS=$COMMAND
fi

```



### 爬取存放目录简介
Nutch各个模块之间的数据交互是通过HDFS来进行的，所以每个模块执行完后，会把结果存在HDFS的某个目录内。

* url_dir: 自己定义一个目录，里面放URL种子的文件。一行一条URL记录，记录后面可选跟若干以`\t`分割的`metakey=vale`对，nutch自己定义了一些。
* nutchWorkdir： 自己定义一个目录。nutch的爬取内容等中间结果全在这里。它里面有几个目录，一个是crawldb，放爬取URL的信息，一轮爬取待爬的URL也是从这里产生的。segments，放中间结果的文件夹，里面按照时间命名目录。nutch的一轮爬取在generate的时候，自动产生一个时间目录。然后一轮的fetch、parse、update都是针对这个最新生成目录里的内容来进行的，这个目录里也按照一轮爬取的不同阶段，生成了按照阶段命名的目录。第三个是linkdb，这个是在爬取循环外的，也就是索引和solr存储那一块用的。当然以上的目录都是可配的，默认就这么着吧。

## Inject
### nutch脚本调用方法
`Usage: Injector <crawldb> <url_dir>`。crawldb是URL库，url_dir是seedURLdir。
### 执行流程
#### 总体流程-crawldb不存在时
1. 创建临时目录temp_dir
2. sortJob<url_dir,temp_dir,InjectMapper,InjectReducer在这里只有inject的URL，其实跳过了合并的逻辑>, job直接输出`MapFile`格式文件。
3. CrawlDb.install(sortJob,crawldb_dir)//将crawldb的current目录rename为old，将job输出目录rename为crawldb的current。

####总体流程-crawldb存在时

1. 创建临时目录temp_dir
2. sortJob<url_dir,temp_dir,InjectMapper> `无Reduce，Reduce在mergeJob做`，只有一个Map，生产SequenceFile格式文件。
3. mergeJob<temp_dir,crawldb_temp_dir,CrawlDbFilter(map)URL规范化和过滤,CrawlDbReducer(reduce)合并new page entries with exists entries,`InjectReducer(reduce)`然后再和inject的合并，注意和crawldb不存在的区别> 输入目录为temp_dir还有**current？**输出是MapFile格式文件。
4. CrawlDb.install(mergeJob,crawldb_dir)//将crawldb的current目录rename为old，将job输出目录rename为crawldb的current。


### 相关数据结构
#### CrawlDatum
此类贯穿爬取的整个流程，存储一个URL的爬取状态、评分、htmlMD5摘要签名、抓取时间等。  
定义了一个segment里面的对应目录：

```
  public static final String GENERATE_DIR_NAME = "crawl_generate";
  public static final String FETCH_DIR_NAME = "crawl_fetch";
  public static final String PARSE_DIR_NAME = "crawl_parse";
```
定义了爬取各个过程的状态(用一个字节表示)，更多见`CrawlDatum.java`，亦可参考[NutchWiki: CrawlDatumStates](http://wiki.apache.org/nutch/CrawlDatumStates)：

```
 /** Page was not fetched yet. */
  public static final byte STATUS_DB_UNFETCHED      = 0x01;
  /** Page was successfully fetched. */
  public static final byte STATUS_DB_FETCHED        = 0x02;
  ...
```
实现了`WritableComparable<CrawlDatum>`接口。  
实现了一个继承自`WritableComparator`的嵌套类`Comparator`，可以在不反序列化的情况下直接对字节流进行URL的score比较。

## Generator
### 脚本调用方法
`Usage: Generator <crawldb><segments_dir> [-force] [-topN N] [-numFetchers numFetchers] [-adddaysnumDays] [-noFilter] [-noNorm][-maxNumSegments num]`
从crawldb产生待爬取URL到segments目录。
### 执行流程

1. 在`map.temp.dir`里创建一个临时目录temp_dir。
2. 按照score排序，并在临时目录里生成多个fetchlist。generateJob<crawldb/current, temp_dir,sequenceFile->sequenceFile,Mapper:Selector,Partitioner:Selector,Reducer:Selector,output:<FloatWritable,SelectorEntry,DecreasingFloatComparator>,OutputFormat:GeneratorOutputFormat>。
3. 从临时目录生成segments，原则是临时目录里有几个以`fetchlist-`开头的文件夹，就产生几个job，生成几个segment，每个segment里产生一个子目录`crawl_generate`。partitionSegmentJob<temp_dir/fetchlist-N,segments/当前时间命名的文件夹/crawl_generate,sequenceFile->sequenceFile,Mapper:SelectorInverseMapper,Partitioner:URLPartitioner,Reducer:PartitionReducer,Output:<Text,CrawlDatum,HashComparator>>

### 相关子流程

#### Selector.mapper
1. 调用filters过滤该URL。通过则继续。
2. 检查是否在爬取日程上，比如有的URL可能设置爬取间隔很大（上次爬取时间+爬取间隔 > 当前时间），故不爬取。
3. 进行评分，方法是设置初始分值1分，调用一系列scoreFilters（责任链），不断更新分数，最后得出一个分值。这些打分器有的原分值直接返回(generatorSortValue)，有的将原分值*crawlDatum里的score(OPICScoringFilter)，还有按链接的depth打分的等。。
4. 设置CrawlDatum的genTime，设置该SelectorEntry，设置score为key，SelectorEntry为value输出。

#### Selector.Partitoner
虽然重写了getPartition方法，但是其实是调用了URLPartitioner的getPartition方法，只使用了URL作为分区依据。可以根据Host、Domain、IP（在这里调用`InetAddress.getByName(url.getHost())`解析出IP）3种方式（`partition.url.mode`进行配置）来计算hashcode，进而分区。这样相同的Host（或IP等）就分到一个分区下。

#### Selector.Reducer
我们这里讨论一个Reduce。  
因为该Reduce处理之前所有URL已经按照score倒序排好了，所以我们取limit（topN/Partition数，Partition数也即Reduce数）个URL即可。类的私有变量count记录了一个Reduce已经取了多少个URL。以byHost为例。  
如果`generate.max.count`不为-1（默认-1），逻辑有点麻烦，意味着需要判断host/domain下的URL个数是否达到限制，这个详见源码。为-1不必考虑这个问题。
然后依次取够segmentNum（如果其不为1）遍limit，设置SelectorEntry的segmentNum为1，2，3。。。（如果设置了`generate.max.num.segments`，则一个Reduce可产生多个segment，每个segment都能有limit个URL，没设置则只能产生1个segment）。  
举个例子，如果segmentNum的数量为2，一共有4个Reduce，TopN=10000，则最后我们在hdfs上segments文件夹下产生了2个时间命名的文件夹，每个文件夹里的crawl_generate文件夹里都应有4个part-0000x文件,每个文件包含item数量为：10000/4=2500。其实这个segmentNum设置成1就行，因为下面Fetching我们就能看到，一个Fetching job是针对segments文件夹下的一个文件夹进行的。一次产生多个还是要手动提交他们到Fetching的job里的。


#### SelectorInverseMapper.Mapper
因为上一步已经选出URL的list，所以这一步仅仅是取<URL，selectorEntity>输出。

#### URLPartitioner
上一步partitioner已经介绍。

#### PartitionReducer.Reducer
这一步取<URL，CrawlDatum>输出。其实这个Reduce一个key（URL）的values集合只应有一个SelectorEntry元素。这样说来`这个Reduce是多余的？`是否只要一个Map就可以了？  
如果只有Map是否就没有Partition了，这个Partition似乎`也是多余的？`因为即使partition了他们也是在一个输出目录。而上一步的`GeneratorOutputFormat`重写了输出文件名方法`generateFileNameForKeyValue`，会让有不同segNum的URL被放入不同的fetchlist-segNum目录。  
最后这个排序`似乎也是多余的`，排序的比较器是HashComparator，排序依据是URL的hash值。。没啥用啊这个排序。还不如一个Map直接<URL，CrawlDatum>输出算了。其实这个hash也算个随机，防止URL按照字符序排列，尽量打散这些URL。  
上一个job分区使同一个host或domain等的URL处于一个reducer里，避免对同一个host或domain下的URL并行爬取，这个job对同一个分区内的URL按照hash进行了随机打散，避免同一个host下的URL排在一起。


#### GeneratorOutputFormat
这个类只重写了generateFileNameForKeyValue方法，定义了输出的文件名格式为：“fetchlist-segmentNum/name”，name据推断是part-000N这类玩意。自定义输出格式除了文件名之外还有其他的一些设置。如常见的SequenceFileOutputFormat直接继承自FileOutPutFormat，而我们这个GeneratorOutputFormat为了实现重命名的功能继承关系为：`GeneratorOutputFormat->MultipleSequenceFileOutputFormat->MultipleOutputFormat(这货有命名的功能，因此可以多个目录输出)->FileOutputFormat`
### 相关数据结构
#### SelectorEntry
是segNum、URL、CrawlDatum的简单包装。segNum是segment的编号，取值1，2，3...
### 其他问题
#### 每个reducer生成的fetchlist中究竟有多少个url？受哪些参数控制？（见参考的博客内容）--> 一轮Generate我们生成了多少个URL？
每个reducer能生成的fetchlist个数为maxNumSegments个，该参数由命令行传入(或者配置generate.max.num.segments)。而每个fetchlist里面含有多少个url主要由参数topN和reducer的个数N控制，为limit=topN/N个，但是也受到generate.max.count和generate.count.mode参数控制，如果满足:  
1）host/domain的种类足够多，并且2）crawldb中url个数足够多的话，每个fetchlist中URL的个数肯定能达到上限limit，generate.max.count和generate.count.mode参数只能控制里面url的分布而已。但是如果不满足这2个条件中任何一个的话，可能会出现fetchlist“装不满”的情况。  
由上面的分析可以看出，如果URL足够多，则：

		最后生成的segments共包含的URL个数=reduce个数*maxNumSegments*limit
		                             =reduce个数*maxNumSegments*topN/reduce个数
		                             =maxNumSegments*topN
这个Generate的URL数量其实是挺大的。但是在一轮抓取里这些选出来的URL是否都能被抓取一次呢？且看下节Fetcher分解。

## Fetcher
先来看一下源码里的介绍：  
具体见源码注释，下面仅列摘要 ^_^  
基于队列的Fetcher，一个producer（QueueFeeder）用来读取fetchlists，然后把要抓取的FetchItem送给队列们，一群抓取线程从Queues里取item来抓取。队列的个数和host的个数相等。任何时候所有队列里的item的个数会小于一个上限：抓取线程数的N(fetcher.queue.depth.multiplier,default 50)倍。  
随着抓取的进行，队列里的item会减少，同时producer会进行补充。有时抓取线程也会往队列里添加item，比如遇到重定向时。  
Fetcher不使用插件机制，而是自己实现了基于host的队列阻塞处理，每个byhost的队列都可以有自己的最大并发请求数、2次URL请求间的抓取间隔等个性设置。  
如果Queues里还有items，但是这些items都不ready，那么抓取threads会spin-wait直到有item ready，或者等超时了，就abort，认为task is hung。

### 脚本调用方法

	Usage: Fetcher <segment> [-threads n]
可以看出最多指定segment文件夹、抓取线程数2个参数。注意这里segment指的是segments文件夹里以时间命名的文件夹。每一个Fetcher流程只能处理一个时间文件夹。如果  
我们查看crawl脚本里，
	
	"$bin/nutch" fetch $commonOptions -D fetcher.timelimit.mins=$timeLimitFetch "$CRAWL_PATH"/segments/$SEGMENT -noParsing -threads $numThreads
里面的参数`-noParsing`并没有什么卵用，源码里根本没用到这个，可能是历史遗留。而放入配置变量内的`fetcher.timelimit.mins`却会起作用，限制了一轮Fetching的最长时间，在这个时间点到了之后，Queue里即不再补充新URL。因此一轮抓取并不一定能够把Segments里的URL都抓完。
### 执行流程

1. 抓取开始，check是否配置`http.agent.name`，未配置直接退出。设置hadoop推测执行关闭，防止它觉得job太慢又启动一个同样的task。设置timeLimit，最多运行多久后退出。设置`fetcher.follow.outlinks.depth`，设置`fetcher.follow.outlinks.num.links（default 4）`，设置`fetcher.follow.outlinks.depth.diviso(default 2)`，这3个还不知道干啥的。
2. 抓取job fetch-segment<segments/yyyyMMddhhmmss/crawl_generate,segments/yyyyMMddhhmmss/crawl_fetch,SequeuceFile->SequenceFile,**MapRunner:Fetcher**, Input:InputFormat,output:FetcherOutputFormat<Text,NutchWritable>>.

### 执行子流程

#### Fetcher.run
这块是Fetcher启动的主流程。  
注意Fetcher的Map是默认的，也就是IdentityMapper，啥也不做。它实现了MapRunnable接口，启动主流程都在这个run方法里。这里面有几个数据结构，会在下面详细介绍。它的流程如下：  
1. 创建FetchItemQueues，它包含所有的Queue。
2. 创建QueueFeeder,负责读取URL到Queues，并启动。
3. 创建所有的爬取线程。
4. 设置流量控制相关参数（每秒页面数，记录每秒下载bytes数等）。
5. 进入循环（主要是做一些状态日志打印，如那个1秒一行的状态信息，还有每秒下载pags数、流量等的统计加和。还有一些策略控制）。循环退出条件是抓取线程数为0。或者System.currentTimeMillis() - lastRequestStart.get()) > timeout（mapred.task.timeout/divisor）直接return。
#### InputFormat
为防止Generate生成的part-0000x能顺序读取而不被打散，重写了getSplits方法。这里可以看一下父类FileInputFormat里getSplits方法怎么处理输入文件的。

#### FetcherOutputFormat
输出这里做了一个判断，针对fetch、content、parse三部分内容存入不同的目录。

#### 
### 相关数据结构

#### QueueFeeder
继承自Thread类，我们看它的run方法。里面是个while读取RecordReader，如果队列们不满（FetcherItemQueues.getSize() < threadcount*fetcher.queue.depth.multiplier），就循环往队列里放直到放够为止。如果timeLimit超了，就把RecordReader读完，扔掉，不往队列里放。

#### FetchItemQueues
这是一个Queues的包装类，里面用Map<String, FetchItemQueue>维持队列们，但是维持了最多threadcount*fetcher.queue.depth.multiplier个抓取URL。提供了一些遍历抓取线程和QueueFeeder的方法。包装了对队列的一些操作。
我们还是来看看包装的主要方法吧：
  
	//添加一个url到Queues，可byHost/byDomain等添加到不同的Queue中
 	public void addFetchItem(Text url, CrawlDatum datum);

	//完成一个FetchItem的抓取？仅仅是设置了FetchItem所属队列的nextFetchTime为当前时间（如果asap的话，不然是当前时间+crawlDelay）。
    public void finishFetchItem(FetchItem it) ;

	//从Queues中取一个FetchItem
    public synchronized FetchItem getFetchItem();
    
    // check是否时间过了，如果过了就清空所有队列并返回清空的FetchItem个数。只在feeder停止后执行。
    public synchronized int checkTimelimit();
    // 清空所有Queues并返回清空的FetchItem个数
    public synchronized int emptyQueues();

#### FetchItemQueue
这代表一个Queue，处理一堆拥有相同hostId（byHost/Domain/IP等）的FetchItems，同时还记录当前爬虫request的状态，using nextFetchTime（这是一个重要的变量，当前时间只有大于它时，才能从队列中取元素。通过它可以控制线程抓取的间隔），inProgress（正在被抓取的item个数），crawlDalay/minCrawlDelay等变量。  
当一个Queue的线程数为1时，使用crawlDelay（fetcher.server.delay），大于1时，使用minCrawlDelay（fetcher.server.min.delay）。  
注意：每个Queue都有crawlDelay这个变量，但是在创建Queue的时候使用的是FetchItemQueues的crawlDelay，所以每个Queue的crawlDelay都一样。如果想针对不同Host/domain/ip设置不同的crawDelay，需要改造Queues的`getFetchItemQueue`方法，在其中的createQueue加入判断设置不同crawlDelay。
下面看需要注意的方法：  
	
	//正在爬取的item个数
    public int getInProgressSize();
    
	//一个item爬取结束了，通知Queue。Queue减少一个正在爬取的item个数，然后设置队列的nextFetchTime，asap的y/n设置nextFetchTime为当前时间/当前时间+crawlDelay
    public void finishFetchItem(FetchItem it, boolean asap) ;
 
	//一个item爬取开始，通知队列把inProgress计数加1.
    public void addInProgressFetchItem(FetchItem it) ;
    
    //从队列里取一个item。如果条件不满足，返回null。什么是条件不满足？
    //1. 该Queue正在抓取的线程数大于等于设置的一个Queue的最抓取线程数
    //2. 当前时间还没到nextFetchTime
    //3. Queue.size() == 0
    //满足则：队列第一个元素出队并返回，inProgress++。
    public FetchItem getFetchItem();
   
   	//这个就是设置nextFetchTime的方法。nextFetchTime = endTime（当前item爬完的时间） + delay
    private void setEndTime(long endTime);
    
#### FetchItem
代表一个要被抓取的item。成员变量包括：  
URL，CrawlDatum，queueID（每个Queue都有一个String类型的id，作为FetchItemQueues的Queue Map的key，如果byHost这个id就是protocol://HostName，举个栗子：http://music.baidu.com），outlinkDepth（呃，这个东西。。）。  
它有2个构造函数，同时也有2个静态的create方法。

#### FetcheThread
最后这个是Fetcher的真正主线。。这才是抓取的线程。它里面的成员变量真是多啊，有URL过滤的，URL分数过滤的，规范化，重定向相关，parse outlink相关的一堆参数。  
当然这个主线也是一个死循环，循环如下：

1. getFetchItem，如果得不到，则 [判断Feeder挂了&&Queues大小为0 ->`退出`，否则spin-waiting（也就是sleep 500 然后continue）]。
2. 设置**Fetcher**的lastRequestStart为当前时间。
3. 进入一个while循环，退出条件是`redirect次数 > maxRedirect次数`，退出后这个URL的抓取就完事了。下面是这个循环内的内容。Robots协议相关内容，不符合，跳过；Robots中设置的crawlDelay过大，跳过。。。 然后就是一个抓取，抓取完成就output，附带parse一下里面的内容，这里根据抓取结果做了很多状态判断，输出，redirect，输出错误等等。

比较重要的是output里面的逻辑。
