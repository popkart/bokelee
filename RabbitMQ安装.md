# RabbitMQ3.2.0版本安装
---
##第一部分 erlang源码安装
操作系统为CentOS6.5。  
对于erlang源码安装需要的依赖包，我采用下载rpm包的方式进行安装（因为我假设不联网，联网可以采用yum的方式，先装epel的repo库`epel-release-6-8.noarch.rpm`，然后直接yum就能装依赖包,命令`yum install  build-essential make gcc gcc-c++ glibc-devel kernel-devel m4 ncurses-devel  openssl openssl-devel autoconf unixODBC unixODBC-devel`。我下面其实只装了ncurses，其他ssl那些没装，对应erlang功能也未开启）。  
下面的内容包含试错过程。  
下载erlang的源码包`otp_src_R16B03-1.tar.gz`，解压，源码根目录下`./configure`。  
报错： `configure: error: No curses library functions found`。网上查了下说要装`ncurses-dev`。  
`rpm -qa|grep curses` 列一下已经安装的curses，发现`ncurses-base`、`ncurses-libs`都装了，是5.7.3版本。  
网上只下到`ncurses-dev5.7.4`版本，依赖5.7.4版本的`ncurses-base`、`ncurses-libs`库。因此又下了5.7.4版本的`ncurses-base`、`ncurses-libs`。  
[三个ncurses包（base、libs、dev）下载](http://rpmfind.net/linux/rpm2html/search.php?query=ncurses-devel&submit=Search+...&system=centos&arch=)  
安装rpm需要切换到root权限，切换到root用户。
用`rpm -e ncurses-base-5.7-3.20090208.el6.x86_64` 卸载5.7.3版本的2个东东，结果说他们的某个.so被某个西西依赖，不能卸载。
用
 `rpm -ivh ncurses-base-5.7-4.20090207.el6.x86_64.rpm` 安装5.7.4-base成功，可以和5.3共存。  
看来只能update了。  
`rpm -U  ncurses-libs-5.7-4.20090207.el6.x86_64.rpm` 成功把5.7.3-libs升级到5.7.4。
这时候可以用
`rpm -ivh ncurses-devel-5.7-4.20090207.el6.x86_64.rpm` 安装dev了。  
此时因为5.7.3版本的base已经不被libs依赖了，可以用`rpm -e`删掉。

> 总结：更新、删除老的包要从最顶层的包删起，因为底层的包被顶层的包依赖，不删掉顶层没法动底层的包。

接下来我`export LANG=C`(未验证不设置能否安装成功，网上说要设置下)。然后`./configure`配置成功。用`make && make install`安装到系统默认路径（需要root用户权限）。这时候切换回普通用户，敲`erl`命令进入erl的cli，说明安装成功。

## 第二部分 RabbitMQ安装和配置
我们在普通用户下安装它。安装步骤，解压就是了。。。  
先解压`rabbitmq-server-generic-unix-3.2.0.tar.gz`。假设解压后根目录为`RABBITMQ_HOME`。
然后到`RABBITMQ_HOME/etc/rabbitmq`文件夹下新建配置文件 `rabbitmq-env.conf`（必须叫这个名字），我这里简单配置如下，更详细配置参见官网文档(username/hostname对应机器的用户名和主机名，主机名一定要写当前节点的主机名)：
<pre>
RABBITMQ_NODENAME=username@hostname
RABBITMQ_MNESIA_BASE=$RABBITMQ_HOME/data/mnesia
RABBITMQ_MNESIA_DIR=$RABBITMQ_MNESIA_BASE/$RABBITMQ_NODENAME
RABBITMQ_LOG_BASE=$RABBITMQ_HOME/logs

#Erlang 日志文件
#RABBITMQ_LOGS=$RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME.log
#Erlang 认证日志文件
#RABBITMQ_SASL_LOGS=$RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME-sasl.log
#RabbitMQ PID
#RABBITMQ_PID_FILE=$RABBITMQ_MNESIA_DIR.pid
</pre>

注意我把数据文件和日志文件放在了安装目录里。这里可以配置到其他地方。

下面是几个常用命令，他们在安装目录的sbin目录下。如果想在任何目录下使用这个命令请将sbin目录加入shell的path。  
mq启动之后默认是没有装插件的。启动之后，根据下面的命令激活控制台插件，这时候就可以登录web控制台了，里面可以监控队列情况，并做一些操作。  
还可以安装cli管理插件，也就是登录到web控制台，看页面的最下面，有两个链接，一个是`api`（api接口说明），一个是`cli`（命令行接口）。点开cli链接，页面里有个rabbitmqadmin文件，里面是Python写的命令行脚本。把这个脚本拷贝到系统里的一个文件里并赋予可执行权限，就能从命令行调用管理了（不知道为什么不把这个脚本直接放到安装目录的sbin目录下，还要手工下载。。）。


```
启动
rabbitmq-server
后台启动
rabbitmq-server -detached
状态
rabbitmqctl status
监听端口验证
netstat -an | grep 5672
停止
rabbitmqctl stop

添加用户：用户名/密码
rabbitmqctl  add_user userfoo bar
设置管理者
rabbitmqctl  set_user_tags userfoo administrator
给这个用户vshost权限
rabbitmqctl  set_permissions -p / userfoo ".*" ".*" ".*"

激活Web控制台等插件（执行后，重启mq生效）
rabbitmq-plugins enable rabbitmq_management

安装cli管理插件(不用命令行操作这个不用装)
脚本路径：http://${IP地址}:15672/cli/rabbitmqadmin 将脚本另存为$RABBITMQ_HOME/sbin/rabbitmqadmin 文件，修改文件权限为可执行：
chmod u+x $RABBITMQ_HOME/sbin/*

```

最后注意一下**数据目录的权限**问题。假设我们用foo账号安装了mq，然后用root用户来启动，这时候mq会在data文件夹里创建一些目录等，而他们的权限是只有root可以写的。这样导致在停掉mq，用foo用户启动mq时，由于foo用户对root用户产生的目录等无写权限导致一些问题，如无法持久化队列等。因此最好只用foo用户来启动mq。    
3.2.0版本的MQ安装完后有一个默认的guest用户（还是Administrator权限）。  
3.3.3版本的MQ安全起见，默认禁止guest用户登录，可以手动添加用户。  
更多内容请参见RabbitMQ的官网相关文档。





