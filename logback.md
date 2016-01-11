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

### Appenders & Layout
#### Appender
Logback allows logging requests to print to multiple destinations. An output destination is called an appender.
Currently, appenders exist for the console, files, remote socket servers, to MySQL, PostgreSQL, Oracle and other databases, JMS, and remote UNIX Syslog daemons.More than one appender can be attached to a logger.  
在理解上，应该把Logger的级别和Appender分开来计算。级别决定这个Logger Request是否要输出，Appender决定输出到哪些地方。  
The `addAppender` method adds an appender to a given logger. Each **enabled logging request**(比如`log.info("xxx")`) for a given logger(该类名全限定名的Logger) will be forwarded to all the appenders in that logger as well as the appenders higher in the hierarchy。也就是说，一个Logger的请求，会发给它以及它的祖先的所有Appender，前提是这个请求是达到设定级别的（由该Logger的级别定义决定）。还有一个限制，`Addtivity`：  

>**Appender Additivity**
>
The output of a log statement of logger L will go to all the appenders in L and its ancestors. This is the meaning of the term "appender additivity".
>
However, if an ancestor of logger L, say P, has the additivity flag set to false, then L's output will be directed to all the appenders in L and its ancestors up to and including P but **not the appenders in any of the ancestors of P**.(可加性，某一节点设置该属性为false，则它及它的后代节点均不会继承它的祖先节点的Appender)
>
Loggers have their additivity flag set to true by default.

#### Layout
The layout is responsible for **formatting the logging request** according to the user's wishes, whereas an appender takes care of sending the formatted output to its destination.   
The `PatternLayout`, part of the standard logback distribution, lets the user specify the output format according to conversion patterns similar to the C language printf function.

For example, the PatternLayout with the conversion pattern "`%-4relative [%thread] %-5level %logger{32} - %msg%n`" will output something akin to:

	176  [main] DEBUG manual.architecture.HelloWorld2 - Hello world.
The first field is the number of milliseconds elapsed since the start of the program. The second field is the thread making the log request. The third field is the level of the log request. The fourth field is the name of the logger associated with the log request. The text after the '-' is the message of the request.

### Other Tips
#### Parameterized Logging

>
1. `logger.debug("The new entry is "+entry+".");`
2. `logger.debug("The new entry is {}.", entry);`

第2种写法在debug输出级别被禁用（也就是不输出）时会比第1种写法快30%。因为参数化的方法是在验证是否输出之后，再format输出字符串，而且format的方法也被优化过。
#### A peek under the hood
下面以在Logger上调用`info()`方法为例展示logback输出日志的步骤。  


1. ***Get the filter chain decision***  
按照限制条件过滤。  
If it exists, the TurboFilter chain is invoked. Turbo filters can set a context-wide threshold, or **filter out certain events** based on information such as **Marker, Level, Logger, message**, or the Throwable that are associated with each logging request. If the reply of the filter chain is ：
	* FilterReply.DENY, then the logging request is dropped.   
	* FilterReply.NEUTRAL, then we continue with the next step, i.e. step 2. 
	* FilterReply.ACCEPT, we skip the next and directly jump to step 3.

2. ***Apply the basic selection rule***  
比较request和该Logger的有效级别，并过滤。  
At this step, logback **compares the effective level of the logger** with the **level of the request**. If the logging request is disabled according to this test, then logback will drop the request without further processing. Otherwise, it proceeds to the next step.

3. ***Create a LoggingEvent object***  
创建一个包含request各种信息的`LoggingEvent`。  
If the request survived the previous filters, logback will create `a ch.qos.logback.classic.LoggingEvent` object containing all the relevant parameters of the request, such as the logger of the request, the request level, the message itself, the exception that might have been passed along with the request, the current time, the current thread, various data about the class that issued the logging request and the MDC. Note that some of these fields are initialized lazily, that is only when they are actually needed. The MDC is used to decorate the logging request with additional contextual information. MDC is discussed in a subsequent chapter.

4. ***Invoking appenders***  
调用所有关联Appender的`doAppend()`方法，每个Appender也可以有单独的`custom filters`。  
After the creation of a LoggingEvent object, logback will invoke the doAppend() methods of all the applicable appenders, that is, the appenders inherited from the logger context.  
All appenders shipped with the logback distribution extend the AppenderBase abstract class that implements the doAppend method in a synchronized block ensuring thread-safety. The doAppend() method of AppenderBase also invokes custom filters attached to the appender, if any such filters exist. Custom filters, which can be dynamically attached to any appender, are presented in a separate chapter.

5. ***Formatting the output***  
由Appender来格式化输出字符串（有时候Appender会把这个工作交给Layout去做，但有的Appender比如SocketAppender没有Layout）。  
It is responsibility of the invoked appender to format the logging event. However, some (but not all) appenders delegate the task of formatting the logging event to a layout. A layout formats the LoggingEvent instance and returns the result as a String. Note that some appenders, such as the SocketAppender, do not transform the logging event into a string but serialize it instead. Consequently, they do not have nor require a layout.

6. ***Sending out the LoggingEvent***  
由Appender把结果发送出去。  
After the logging event is fully formatted it is sent to its destination by each appender.
Here is a sequence UML diagram to show how everything works. 

 ![underTheHoodSequenceDiagram](img/underTheHoodSequence2.gif)


#### Performance
1. ***Logging performance when logging is turned off entirely***  
You can turn off logging entirely by setting the level of the root logger to `Level.OFF`, the highest possible level. When logging is turned off entirely, the cost of a log request consists of **a method invocation plus an integer comparison**. 注意这是采用`Parameterized Logging`之后的结果，如果使用“+”的方式连接输出字符串，则不管关不关日志开关，连接字符串操作等计算都会进行。

2. ***The performance of deciding whether to log or not to log when logging is turned on***  
每个Logger的输出级别都在创建时已经计算出，因此可准实时做出判断而不必遍历其祖先节点的级别。  
3. ***Actual logging (formatting and writing to the output device)***  
Layout的formatter和Appender一样都被高度优化过，实际的logging时间消耗典型值为9-12microseconds（输出到本地日志文件）。

虽然有很多特性，logback始终把性能放在第一位，其次是可靠性。







## References
[1] logback manual, <http://logback.qos.ch/manual/>
