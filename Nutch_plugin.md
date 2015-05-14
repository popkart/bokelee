# Nutch plugin
---
相关链接：[Nutch插件中心](http://wiki.apache.org/nutch/PluginCentral) （插件介绍，别人写的插件example） 

Nutch通过插件系统来进行定制扩展，支持  

* 插件parsing
* 多协议（http、ftp、等）
* 灵活存储（1.x底层存储到hdfs，2.x利用Apache Gora把存储抽象出来了，可以存储到Gora支持的后端，如hbase，Cassandra等非关系型数据库、solr、avro、hadoop等（Gora已经废弃SQL backend了））
* 索引，以支持Solr，Elastic Search，Solr Cloud等搜索引擎。 

### plugin source directory
1. plugin.xml :tells Nutch about the plugin.
2. build.xml  :tells how to build the plugin.
3. ivy.xml	:describes any dependencies required by the plugin.
4. src code of plugin.  
见[nutch plugin例子，urlmeta](http://wiki.apache.org/nutch/WritingPluginExample)

### Nutch extension-point & extension
Nutch在爬虫的各个阶段提供了extension-points.  
extension-point：接口  
extension：plugin实现  
Nutch提供了9个插件接口，在`src/plugin/nutch-extensionpoints/plugin.xml`里列出来了。
见[AboutPlugin](http://wiki.apache.org/nutch/AboutPlugins)。我们可以在这些接口上提供自己的扩展。
### Write a Nutch Plugin
见[Write Plugin Example](http://wiki.apache.org/nutch/WritingPluginExample)

