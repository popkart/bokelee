## Parse

Parse content in a segment。

### 脚本调用方法

	Usage: ParseSegment segment [-noFilter] [-noNormalize]
可见默认是Filter和Normalize的。


### 执行流程

 jobname(parse201501011234)\<segments/datedir/content,segment/datedir,ParseSegment.map,reduce,\<SequenceFileInputFormat,ParseOutputFormat>>


### 执行子流程
#### map
1. 判断改网页Meta信息里的抓取状态，不是success就跳过。  
2. 如果设置了skipTruncated，则跳过被truncated的网页。  
3. 用`ParseUtil`解析出`ParseResult`。
4. 对ParseResult出的每一个<Text,Parse>，执行下面的流程：
	1. 设置`Parse.Data.ContentMeta`的`nutch.segment.name`为`segmentname`。
	2. 计算签名，并设置`Parse.Data.ContentMeta`的`nutch.content.digest`为签名的`HexString`。
	3. 用`scfilters.passScoreAfterParsing`计算下分数。

输出 <url, ParseImpl implements Parse>。
#### reduce
collect <url, ParseImpl>，仅collect第一个值。然后在写文件的时候使用了ParseOutPutFormat，把ParseImpl里的值分别写入到`parse_text`,`parse_data`,`crawl_parse`三个目录里。
### 相关数据结构

#### ParseResult implements Iterable<Map.Entry<Text, Parse>> 
A utility class that stores result of a parse. Internally a ParseResult stores <Text, Parse> pairs.  
Parsers may return multiple results, which correspond to `parts or other associated documents` related to the original URL.
There will be usually one parse result that corresponds directly to the original URL, and possibly many (or none) results that correspond to derived URLs (or sub-URLs).  

#### ParseOutputFormat


## UpdateDB
一个segment抓取、解析完成后，更新该segment的内容到crawldb。类：`org.apache.nutch.crawl.CrawlDb`。
### 脚本调用方法

```
Usage: CrawlDb <crawldb> (-dir <segments> | <seg1> <seg2> ...) [-force] [-normalize] [-filter] [-noAdditions]
	crawldb	CrawlDb to update
	-dir segments	parent directory containing all segments to update from
	seg1 seg2 ...	list of segment names to update from
	-force	force update even if CrawlDb appears to be locked (CAUTION advised)
	-normalize	use URLNormalizer on urls in CrawlDb and segment (usually not needed)
	-filter	use URLFilters on urls in CrawlDb and segment
	-noAdditions	only update already existing URLs, don't add any newly discovered URLs

```


### 执行流程

 crawldbjob<CrawlDbFilter.map,CrawlDbReducer.reduce,Input:<segments/201501010101s/crawl_fetch,segments/2015010101s/crawl_parse,crawldb/current>,output:crawldb/random.nextInt>  
 CrawlDb.install(job, crawlDb); //Rename crawldb/current to crawldb/old,crawldb/random.nextInt to crawldb/current
### 执行子流程
#### CrawlDbFilter.map
规范化URL、过滤URL。输出<url, CrawlDatum>。
#### CrawlDbReducer.reduce
Merge new page entries with existing entries.  
从`crawldb/crawl_fetch/crawl_parse`三类目录下，进行merge。  
遍历values，得到`getFetchTime()`最近的fetchDatum和oldDatum，然后这两个CrawlDatum配合各种Fetch_Status得出最后的result（这个有点复杂，因为和CrawlDatum的各种Status相关，`switch(fetchDatum.getStatus())`）。


### 相关数据结构


## Deduplication on CrawlDb

Update db之后，对Crawldb中status为`fetched`的URL进行去重（按照`指纹`而不是URL）。  
Generic deduplicator which `groups fetched URLs` with the same `digest` and `marks` all of them as `duplicate` except `the one with the highest score` (based on the score in the crawldb, which is not necessarily the same as the score indexed). If two (or more) documents have the same score, then the document with the latest timestamp is kept. If the documents have the same timestamp then the one with the shortest URL is kept（如果URL长度又一样呢？哈哈，代码里这个情况没处理，也就是如果长度也一样，就把这2个实例当做不重复来对待了，继续留在crawldb里。我觉得应该就取第一个来的算了，第二个标记为STATUS_DB_DUPLICATE）. The documents marked as duplicate can then `be deleted` with the command `CleaningJob`.
### 脚本调用方法

```
Usage: DeduplicationJob <crawldb>

```


### 执行流程
DeduplicationJob<DBFilter.map, DedupReducer.reduce,input:crawldb.current,output:mapred.tmp.dir/dedup-job-randomInt>，其中`setMapOutputKeyClass(BytesWritable.class)`。  
MergeJob。替换了UpdateDB的crawldbjob的ReducerClass，`mergeJob.setReducerClass(StatusUpdateReducer.class)`。另外，Inputpath包含上一步的临时目录输出。

### 执行子流程
#### DbFilter.map

1. 判断URL是否是`STATUS_DB_FETCHED`和`STATUS_DB_NOTMODIFIED`两种状态中的一种。
2. 取得其signature，作为map输出的`key`。
3. 把CrawlDatum的MetaData里加上`_URLTEMPKEY_`属性存入当前URL（仅仅是在内存中，这个属性）。

#### DbFilter.reduce

按照类说明里的规则进行标记、去重，这里粘出来前两步示例：

```
 CrawlDatum newDoc = values.next();
                // compare based on score，existingDoc为同一个指纹的第一个URL文档
                if (existingDoc.getScore() < newDoc.getScore()) {
                    writeOutAsDuplicate(existingDoc, output, reporter);//status变为STATUS_DB_DUPLICATE
                    existingDoc = new CrawlDatum();
                    existingDoc.set(newDoc);
                    continue;
                } else if (existingDoc.getScore() > newDoc.getScore()) {
                    // mark new one as duplicate
                    writeOutAsDuplicate(newDoc, output, reporter);
                    continue;
                }
                // same score? delete the one which is oldest
                if (existingDoc.getFetchTime() > newDoc.getFetchTime()) {
                    // mark new one as duplicate
                    writeOutAsDuplicate(newDoc, output, reporter);
                    continue;
                } else if (existingDoc.getFetchTime() < newDoc.getFetchTime()) {
                    // mark existing one as duplicate
                    writeOutAsDuplicate(existingDoc, output, reporter);
                    existingDoc = new CrawlDatum();
                    existingDoc.set(newDoc);
                    continue;
                }
```
最后剩下的那个确定要留下的URL未做任何处理，直接被抛弃了（当然它还在crawldb里）。标记为`STATUS_DB_DUPLICATE`的URL删除`_URLTEMPKEY_`属性，进行输出，放入临时目录。

#### StatusUpdateReducer.reduce
MergeJob前面的Map和UPdateDB一样，后面的reduce不同。Reducer类注释稍有不同：

	Combine multiple new entries for a url.
这个很简单，就是对同一个URL的不同values，如果发现里面有`STATUS_DB_DUPLICATE`标记的实例，就输出它。如果没有，就输出最后一个value。（其实一个URL只会有2种情况，一种是只有一个来自crawldb的value，另一种是多了一个来自上一步临时目录的`STATUS_DB_DUPLICATE`的value）。

### 相关数据结构

## InvertLinks
org.apache.nutch.crawl.LinkDb.

### 脚本调用方法

```


```


### 执行流程

 
 CrawlDb.install(job, crawlDb); //Rename crawldb/current to crawldb/old,crawldb/random.nextInt to crawldb/current
### 执行子流程
#### CrawlDbFilter.map
规范化URL、过滤URL。输出<url, CrawlDatum>。
#### CrawlDbReducer.reduce



### 相关数据结构

## SolrIndex
org.apache.nutch.indexer.IndexingJob.

### 脚本调用方法

```
Usage: Indexer <crawldb> [-linkdb <linkdb>] [-params k1=v1&k2=v2...] (<segment> ... | -dir <segments>) [-noCommit] [-deleteGone] [-filter] [-normalize]

```


### 执行流程

 
 CrawlDb.install(job, crawlDb); //Rename crawldb/current to crawldb/old,crawldb/random.nextInt to crawldb/current
### 执行子流程
#### CrawlDbFilter.map
规范化URL、过滤URL。输出<url, CrawlDatum>。
#### CrawlDbReducer.reduce



### 相关数据结构
