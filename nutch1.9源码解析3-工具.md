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

## mergedb

## mergesegs

## readdb

## readseg

## plugin&Nutch的插件机制