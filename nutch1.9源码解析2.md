## Parse

如果Queues里还有items，但是这些items都不ready，那么抓取threads会spin-wait直到有item ready，或者等超时了，就abort，认为task is hung。

### 脚本调用方法

	Usage: ParseSegment segment [-noFilter] [-noNormalize]
可见默认是Filter和Normalize的。


### 执行流程

 parse201501011234\<segments/datedir/content,segment/datedir,ParseSegment.map,reduce,\<SequenceFileInputFormat,ParseOutputFormat>>


### 执行子流程
#### map
判断改网页Meta信息里的抓取状态，不是success就跳过。  
如果设置了skipTruncated，则跳过被truncated的网页。  

### 相关数据结构