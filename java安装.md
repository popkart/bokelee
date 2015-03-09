#Linux下java的安装

将`jdkxxx.tar.gz`放在`/usr/local/java`目录，解压缩。  
因此java的安装路径是：  

	/usr/local/java/javaxxx版本号
环境设置在：  

	/etc/profile.d/javaenv.sh
内容为：  

```
export JAVA_HOME=/usr/local/java/jdk1.7.0_75
export JRE_HOME=/usr/local/java/jdk1.7.0_75/jre
export PATH=$PATH:/usr/local/java/jdk1.7.0_75/bin
export CLASSPATH=./:/usr/local/java/jdk1.7.0_75/lib:/usr/local/java/jdk1.7.0_75/jre/lib
```

另一种设置环境变量的方法：
把环境变量搞成符号链接那种形式，可以方便多个版本的java切换。

	alternatives -install /usr/bin/java java /usr/local/java/javaxxx版本号 1000
这种情况下，就不用在profile.d目录下放置环境设置脚本了。
具体见`alternatives`这个玩意。
