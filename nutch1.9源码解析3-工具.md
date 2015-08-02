# Nutch1.9源码解析：工具类
---
Nutch1.9提供了一些操作Nutch的Crawldb、linkdb库等的工具命令，比如合并segment，readdb统计等。下面我们对这些工具源码作一概览，之后我们也可以根据我们的需求做出自己的工具类。  
下面是nutch命令里不在crawl流程内的其他命令，接下来对部分命令进行解析。

```
 echo "Usage: nutch COMMAND"
  echo "where COMMAND is one of:"
  echo "  readdb            read / dump crawl db"
  echo "  mergedb           merge crawldb-s, with optional filtering"
  echo "  readlinkdb        read / dump link db"
  echo "  freegen           generate new segments to fetch from text files"
  echo "  readseg           read / dump segment data"
  echo "  mergesegs         merge several segments, with optional filtering and slicing"
  echo "  mergelinkdb       merge linkdb-s, with optional filtering"
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
```

## freegen
org.apache.nutch.tools.FreeGenerator  
随心所欲生成爬取列表（segment/crawl_generate），主要是脱离了crawldb。输入是文本文件，格式很简单，只是一行一条URL。输出是一个segment的crawl_generate。
### 脚本调用方法

```

Usage: FreeGenerator <inputDir> <segmentsDir> [-filter] [-normalize]
	inputDir	input directory containing one or more input files.
		Each text file contains a list of URLs, one URL per line
	segmentsDir	output directory, where new segment will be created
	-filter	run current URLFilters on input URLs
	-normalize	run current URLNormalizers on input URLs

```

### 执行流程
和Generate类似。

```
InputPath:种子文件所在目录
OutPutPath:segment/crawl_generate
Mapper:FG.map //1.不同
Recucer:FG.reduce //2.不同
Patitioner:URLPartitioner.getPartition()//同generate
MapOutPut<Text, Generator.SelectorEntry>
OutPut<Text, CrawlDatum>
Input/Output Format:<TextInputformat, SequenceFileOutputFormat>
OutputKeyComparator:Generator.HashComparator//同generate，Hash乱序

```
总体流程没有Generate麻烦，只一次性生成到segment目录即可。


### 执行子流程
#### FG.map
对UR进行规范化、过滤、设置score，fetchInterval，输出\<url, SelectorEntry\>。  
#### FG.reduce
每个URL只取一个crawldatum（最后一个）。最后输出。同一个key用一个HashMap去重。我觉得只取一个value输出就去重了啊，reduce的输入key既然是URL ，这个HashMap最后也只能有一个item。

## mergedb
org.apache.nutch.crawl.CrawlDbMerger.  

>
This tool `merges several CrawlDb-s into one`, optionally filtering URLs through the current URLFilters, to skip prohibited pages.
>
It's possible to use this tool `just for filtering` - in that case only one CrawlDb should be specified in arguments.
>
If more than one CrawlDb contains information about the same URL, only the most recent version is retained, as determined by the value of org.apache.nutch.crawl.CrawlDatum.getFetchTime(). However, all metadata information from all versions is accumulated, with newer values taking precedence over older values.



### 脚本调用方法

```
Usage: CrawlDbMerger <output_crawldb> <crawldb1> [<crawldb2> <crawldb3> ...] [-normalize] [-filter]
	output_crawldb	output CrawlDb
	crawldb1 ...	input CrawlDb-s (single input CrawlDb is ok)
	-normalize	use URLNormalizer on urls in the crawldb(s) (usually not needed)
	-filter	use URLFilters on urls in the crawldb(s)


```

### 执行流程

>
crawldbMergeJob  
Input:crawldbs/current
Output:`new Path("crawldb-merge-" + Integer.toString(new Random().nextInt(Integer.MAX_VALUE)));`
Input/OutputFormat:SequenceFileInputFormat/MapFileOutputFormat //Sequence和Map有何区别？Map是2个Sequence文件？
\<CrawlDbFilter.map,Merge.reduce\>
output<Text, CrawlDatum>


### 执行子流程
#### Merge.reduce
1. 建立一个resCrawlDatum。
2. 把该URL的第一个CrawlDatum赋给resCrawlDatum。
3. 其他的CrawlDatum val：
	* val.lastFetchTime > res.lastFetchTime，val.getMeta()全set到res的meta里（val新，val的meta覆盖res的）
	* else， res的Meta全set（覆盖）到val的Meta里，然后把val的Meta set为res的Meta
4. 最后输出\<URL, res\>。


## mergesegs
org.apache.nutch.segment.SegmentMerger.  
这个合并有点意思，合并各个segment里的对应目录。  
英文是代码原注释，中文是提要。  
功能：  
合并segments，若不同segment里有相同URL，保留最近的一个。同时也可以用来过滤一个segment里的URL。同时也可以用来把合并结果分割成固定大小的segments。
>
This tool takes several segments and **merges their data together**. Only the latest versions of data is retained.
>
Optionally, you can apply current URLFilters to remove prohibited URL-s.
>
Also, it's possible to **slice the resulting segment** into chunks of **fixed size**.

需要注意的点：  

1. segments里包含各阶段的数据，程序只会合并每个segment都共有的部分。如一个segment是unfetched阶段（只有crawl_generate目录），其他的segment都是已经抓去过、解析完的（还有crawl_fetch,content目录等），这时也只会合并出一个只有crawl_generate目录的结果。
2. 不推荐只合并crawl_generate，因为最后产生的结果不像generate会产生打散的URL。
3. 去重，只根据URL、segement的名字（时间）选取最新的内容，因此segment的命名要严格按照时间来。不会合并不同URL、相同content的网页。
4. 关于索引，因为建立索引（如solr索引）时会写入一个字段segement，因此合并后有新名字的segment将导致索引和segment这点无法对应，需要重新index。



>
Important Notes
Which parts are merged?
It doesn't make sense to merge data from segments, which are at different stages of processing (e.g. one unfetched segment, one fetched but not parsed, and one fetched and parsed). Therefore, prior to merging, the tool will determine the lowest common set of input data, and only this data will be merged. This may have some unintended consequences: e.g. if majority of input segments are fetched and parsed, but one of them is unfetched, the tool will fall back to just merging fetchlists, and it will skip all other data from all segments.
>
Merging fetchlists
Merging segments, which contain just fetchlists (i.e. prior to fetching) is not recommended, because this tool (unlike the org.apache.nutch.crawl.Generator doesn't ensure that fetchlist parts for each map task are disjoint.
>
Duplicate content
Merging segments removes older content whenever possible (see below). However, this is NOT the same as de-duplication, which in addition removes identical content found at different URL-s. In other words, running DeleteDuplicates is still necessary.
For some types of data (especially ParseText) it's not possible to determine which version is really older. Therefore the tool always uses segment names as timestamps, for all types of input data. Segment names are compared in forward lexicographic order (0-9a-zA-Z), and data from segments with "higher" names will prevail. It follows then that it is extremely important that segments be named in an increasing lexicographic order as their creation time increases.
>
Merging and indexes
Merged segment gets a different name. Since Indexer embeds segment names in indexes, any indexes originally created for the input segments will NOT work with the merged segment. Newly created merged segment(s) need to be indexed afresh. This tool doesn't use existing indexes in any way, so if you plan to merge segments you don't have to index them prior to merging.

### 脚本调用方法

```
SegmentMerger output_dir (-dir segments | seg1 seg2 ...) [-filter] [-slice NNNN]
	output_dir	name of the parent dir for output segment slice(s)
	-dir segments	parent dir containing several segments
	seg1 seg2 ...	list of segment dirs
	-filter		filter out URL-s prohibited by current URLFilters
	-normalize		normalize URL via current URLNormalizers
	-slice NNNN	create many output segments, each containing NNNN URLs

```
`-dir`可以指定包含一堆segment的目录，一个一个的segment添加直接将segement目录作为参数即可。  
`-slice NNNN`指定了结果输出分块大小，一个结果segment包含NNNN个URL。不指定是输出为一个segment。

### 执行流程
在开始job之前，进行各个segment的包含文件夹判定，也就是`crawl_generate`、`crawl_fetch`...等，确定每个segment都有的文件夹。对于每个segment，加入共有的文件夹到`Inputpath`。
>
segmentMergeJob  
Input:所有segment共有文件夹
Output:`new Path("crawldb-merge-" + Integer.toString(new Random().nextInt(Integer.MAX_VALUE)));`
Input/OutputFormat: ObjectInputFormat/SegmentOutputFormat
\<SegmentMerger.map,SegmentMerger.reduce\>
output<Text, MetaWrapper>


### 执行子流程
#### SegmentMerger.map
过滤，输出<URL, MetaWrapper> ，输入也是它，MetaWrapper是ObjectFileInputFormat里生成的，对NutchWritable的包装。



## readdb
### 脚本调用方法


### 执行流程



### 执行子流程

### 相关数据结构
## readseg
### 脚本调用方法


### 执行流程



### 执行子流程

### 相关数据结构
## plugin&Nutch的插件机制