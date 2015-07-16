# hadoop 2.6.0 cdh5.4.4版本编译
---
因为cloudera的hadoop包里不包含编译好的`native-hadoop library`，故需要自己编译源码才能获得。  
因为要编译`native`的library，故在运行机器同样的机型上编译，编译系统为CentOS6.5x64系统。

Apache官方编译参考：  
<http://svn.apache.org/repos/asf/hadoop/common/trunk/BUILDING.txt>    
网上一篇文章：  
<http://blog.csdn.net/w13770269691/article/details/16883663/>

---
## 官方Requirements:

* Unix System
* JDK 1.6+
* Maven 3.0 or later
* <del>Findbugs 1.3.9 (if running findbugs) </del>没要这个
* ProtocolBuffer 2.5.0
* CMake 2.6 or newer (if compiling native code)
* Zlib devel (if compiling native code)
* openssl devel ( if compiling native hadoop-pipes )
* Internet connection for first build (to fetch all Maven and Hadoop dependencies)

-------

## 源码下载
cloudera的CDH5下载目录在：<http://archive.cloudera.com/cdh5/cdh/5/>  
我下载的bin包(300M左右)，里面的src目录既是源码。另外可单独下载src源码包（40M）。

## 环境
1. hadoop用`maven`管理工程，故需要配置maven环境变量。
2. `probuf2.5.0`需要用到，下载`probuf2.5.0`源码（github上也有，tag里），先编译它。编译它又要用到其他的依赖，先安装:  
	yum install gcc  
	yum intall gcc-c++  
	yum install make 
然后开始编译安装： 
	tar -xvf protobuf-2.5.0.tar.bz2  
	cd protobuf-2.5.0  
	./configure --prefix=/opt/protoc/  #安装到/opt/protoc,这根据需要安装到一个目录就行  
	make && make install 
之后要配置`/opt/protoc/bin`到环境变量。
再确认下面的也安装了：
	yum install cmake  
	yum install openssl-devel  
	yum install ncurses-devel 
## 开始编译
hadoop源码根目录执行：

	mvn package -Pdist,native -DskipTests -Dtar  
开始编译。
编译选项（官方编译参考文档）：
Maven build goals:

 * Clean                     : mvn clean
 * Compile                   : mvn compile [-Pnative]
 * Run tests                 : mvn test [-Pnative]
 * Create JAR                : mvn package
 * Run findbugs              : mvn compile findbugs:findbugs
 * Run checkstyle            : mvn compile checkstyle:checkstyle
 * Install JAR in M2 cache   : mvn install
 * Deploy JAR to Maven repo  : mvn deploy
 * Run clover                : mvn test -Pclover [-DcloverLicenseLocation=${user.name}/.clover.license]
 * Run Rat                   : mvn apache-rat:check
 * Build javadocs            : mvn javadoc:javadoc
 * Build distribution        : mvn package [-Pdist][-Pdocs][-Psrc][-Pnative][-Dtar]
 * Change Hadoop version     : mvn versions:set -DnewVersion=NEWVERSION

 Build options:

  * Use -Pnative to compile/bundle native code
  * Use -Pdocs to generate & bundle the documentation in the distribution (using -Pdist)
  * Use -Psrc to create a project source TAR.GZ
  * Use -Dtar to create a TAR with the distribution (using -Pdist)

第一次编译需要下很多maven相关、hadoop相关包，持续了两个小时。。最后在`hadoop-dist`目录下生成编译后jar包等。


## 单独编译hadoop工程
官方说明：  
If you are building a submodule directory, all the hadoop dependencies this
submodule has will be resolved as all other 3rd party dependencies. This is,
from the Maven cache or from a Maven repository (if not available in the cache
or the SNAPSHOT 'timed out').
An alternative is to run 'mvn install -DskipTests' from Hadoop source top
level once; and then work from the submodule. Keep in mind that SNAPSHOTs
time out after a while, using the Maven '-nsu' will stop Maven from trying
to update SNAPSHOTs from external repos.

总体意思是，先全部编译一遍hadoop并install到本地库去，然后就可以单独编译一个工程了。因为这样这个工程依赖的那些工程就可以作为第三方jar包来引入了。