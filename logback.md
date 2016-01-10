# Note of Logback Manual
---

## Introduction
Logback is intended as a successor to the popular log4j project.It was designed by Ceki Gülcü, log4j's founder.
更快，封装的更轻便。同样重要的是，它提供了[独特而且非常有用的特性](http://logback.qos.ch/reasonsToSwitch.html)。  
据说现在log4j的新版本性能已经提升，比logback略快。  
## Simple Start
添加依赖：  
slf4j-api.jar(当然要有这个了)  
logback-core.jar  
logback-classic.jar  

其他和log4j一样。

打印测试1：

```
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld1 {

  public static void main(String[] args) {

    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld1");
    logger.debug("Hello world.");

  }
}

```

logback可以打印**Logger status**，如下。
测试2：

```
    // print internal state
    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
    StatusPrinter.print(lc);

```

结果：

```
12:49:22,076 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.groovy]
12:49:22,078 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
12:49:22,093 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.xml]
12:49:22,093 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Setting up default configuration.
```

logback么有找到配置文件，使用了默认配置。注意如果logback出错了也会打印内部状态到控制台。
 
Logback distributions contain complete source code such that you can modify parts of logback library and build your own version of it. 


## Architecture

> It is difficult, if not impossible, for anyone to learn a subject **purely by reading** about it, without applying the information to specific problems and thereby forcing himself to think about what has been read. Furthermore, we all learn best the things that we have discovered ourselves.
>
—DONALD KNUTH, The Art of Computer Programming

### Logback Modules
logback含三个模块：logback-core, logback-classic and logback-access。并且原生实现slf4j的接口（插一句，developer在写代码时候的`log.debug`之类的代码调用的方法签名是由slf4j包来决定的，与logback或log4j无关）。core是其他两个的基础，The third module called access integrates with Servlet containers to provide HTTP-access log functionality.我们主要用classic，它对应了log4j。  

Logback is built upon three main classes: `Logger`, `Appender` and `Layout`.  

* Logger是classic模块的。
* 其他2个是core模块的。

### Name Hierarchy
所有日志系统相对于`System.out.println`的首要优点是，能按照级别分类自由组织日志是否输出。这种分类是logback的`Loggers`的内在属性。所有的Logger都依附于一个`LoggerContext`，由它组织起一个树状层级结构（这个和log4j都类似的）。  
logback的层级结构同样采用java包名组织方法，如`com.apache`是`com.apache.xx`的父级。`root Logger`可通过下面的方法来得到：

	Logger rootLogger = LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);//ROOT_LOGGER_NAME代码里是“ROOT”

logback同样定义了5级日志级别，在`ch.qos.logback.classic.Level`类中定义。  
如果一个Logger没有指定日志级别，它的日志级别从最近的一个指定了级别的祖先节点继承。根节点如果没指定级别则默认为`DEBUG`。摘抄一个例子：  

Logger name  |	Assigned level	| Effective level
-------------|----------------- |---------------
root         |	DEBUG           |	DEBUG
X	         |INFO              |   INFO
X.Y	         |none              |INFO
X.Y.Z        |ERROR             |ERROR

代码示例：

```
import ch.qos.logback.classic.Level;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
....

// get a logger instance named "com.foo". Let us further assume that the
// logger is of type  ch.qos.logback.classic.Logger so that we can
// set its level
ch.qos.logback.classic.Logger logger = 
        (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.foo");
//set its Level to INFO. The setLevel() method requires a logback logger
logger.setLevel(Level. INFO);

Logger barlogger = LoggerFactory.getLogger("com.foo.Bar");

// This request is enabled, because WARN >= INFO
logger.warn("Low fuel level.");

// This request is disabled, because DEBUG < INFO. 
logger.debug("Starting search for nearest gas station.");

// The logger instance barlogger, named "com.foo.Bar", 
// will inherit its level from the logger named 
// "com.foo" Thus, the following request is enabled 
// because INFO >= INFO. 
barlogger.info("Located nearest gas station.");

// This request is disabled, because DEBUG < INFO. 
barlogger.debug("Exiting gas station search");
```

























## References
[1] logback manual, <http://logback.qos.ch/manual/>
