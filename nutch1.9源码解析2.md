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
	1.  

### 相关数据结构