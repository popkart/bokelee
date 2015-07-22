# hadoop 2.6.0-cdh5.4.4安装
---
## 环境配置
### 用户
设有26-30共5台主机。每个主机上建hadoop用户。配置`/etc/hosts`设置每个主机的ip和主机别名映射为`nxx`方便后续操作。要不要改主机名（/etc/sysconfig/network里面改是永久配置，重启生效；用hostname命令改重启前生效）？每个主机改好hosts之后，就可以用别名访问了。先不改主机名。

	注意：
	还是要改主机名的，不然hadoop会弄出
	“host = host-10-1-235-30/202.106.199.37”之类的东西来，其中
	“host-10...”是主机名，后面好像是网通的一个DNS地址。。
	也是醉了。他把主机地址解析到这地方怎么能连的上呢？修改主机名后[#hostname n30]
	正常了：“n30/10.1.235.30”
### JDK环境
安装jdk1.7并配置好jdk路径（root用户装，配进`/etc/profile`以后大家都可以用）。我用rpm安装包安装的，安装后`profile`里添加：

	export JAVA_HOME=/usr/java/jdk1.7.0_55
	export PATH=$JAVA_HOME/bin:$PATH

### ssh无密码互访
1. 在**26-30**上分别执行 `ssh-keygen -t rsa` 生成各自的秘钥。 
2. 用 `ssh-copy-id n26` 把各自公钥拷贝到26上的**authorized-keys**里。
3. 在26上分别执行`rsync  -vzrtopgu   --progress .ssh/authorized_keys hadoop@n29:/home/hadoop/.ssh/authorized_keys` 把**authorized_keys**（它里面现在有26-30的公钥）拷贝到27-30上。
4. 在26上分别执行 `rsync  -vzrtopgu   --progress known_hosts hadoop@n29:/home/hadoop/.ssh/known_hosts` 把known_hosts拷贝到27-30，至此ssh全部打通。  

这里出现了一个问题：known_hosts里的所有主机第二个字段的串串都一样！按道理这个应该是主机的公钥，为什么这个文件里的这个都一样？是虚拟主机的原因吧？连接几个主机的RSA key fingerprint都是一样的...[参考这里-stackoverflow](http://security.stackexchange.com/questions/20706/what-is-the-difference-between-authorized-key-and-known-host-file-for-ssh)   `Much like how the authorized_keys file is used to authenticate users the known_hosts file is used to authenticate serversWhenever SSH is configured on a new server it always generates a public and private key for the server, just like you did for your user. Every time you connect to an SSH server it presents its private key in order to prove its identity. If you do not have its public key, then your computer will ask for it and add it into the known_hosts file. If you have the key, and it matches, then you connect straight away.`。但是依然没说明白为什么known_hosts不同主机第二个字段的串串相同的问题）。
### 防火墙
还是要关掉的。root用户下：
```
service iptables stop
& 说明
如果不关闭防火墙，让datanode通过namenode机的访问，请配置slave01,slave02等相关机器的iptables表，各台机器都要能互相访问
vi /etc/sysconfig/iptables
添加：
-I INPUT -s 192.168.2.18 -j ACCEPT
-I INPUT -s 192.168.2.38 -j ACCEPT
-I INPUT -s 192.168.2.87 -j ACCEPT
```

### hadoop安装位置配置
下载hadoop安装包，用root用户解压到`/opt`下，然后`chown -R hadoop:hadoop /opt/hadoop-2.6.0-cdh5.4.4/` 所有者改为hadoop用户。  创建`/opt/hadoop_dfs`目录并更改所有者为hadoop，作为hdfs目录。  
在其他机器上创建这两个目录并更改所有者为hadoop，为之后同步hadoop文件作准备（不然只能用root传了，要输n次密码还要改权限）：

```
#root用户下
cd /opt
mkdir hadoop-2.6.0-cdh5.4.4
mkdir hadoop_dfs
chown -R hadoop:hadoop hadoop*
```

配置`/home/hadoop/.bash_profile`文件，添加hadoop的路径：

```
export HADOOP_HOME=/opt/hadoop-2.6.0-cdh5.4.4
export PATH=$HADOOP_HOME/bin:$PATH

```
把配置文件用如下命令发送到其他每个机器（机器都是裸机，可覆盖）：
`rsync -vzrtopgu --progress ~/.bash_profile n30:/home/hadoop/.bash_profile `

### hadoop配置文件
决定`n30`为NN。

`etc/hadoop/core-site.xml`：

```
<configuration>
 <property>
                <name>hadoop.tmp.dir</name>
                <value>/tmp/hadooptmp/</value>
                <description>Abase for other temporary directories.</description>
        </property>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://n30:9000</value>
        </property>
        <property>
                <name>io.file.buffer.size</name>
                <value>4096</value>
        </property>
</configuration>

```

`hdfs-core.xml`

```
<configuration>

        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:///opt/hadoop_dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:///opt/hadoop_dfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>

    <property>
        <name>dfs.nameservices</name>
        <value>hadoop-cluster1</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>n29:50090</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>

```
`mapred-site.xml`

```
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
                <final>true</final>
        </property>

    <property>
        <name>mapreduce.jobtracker.http.address</name>
        <value>n30:50030</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>n30:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>n30:19888</value>
    </property>
        <property>
                <name>mapred.job.tracker</name>
                <value>http://n30:9001</value>
        </property>
</configuration>

```

`yarn-site.xml`

```
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>n30</value>
        </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>n30:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>n30:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>n30:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>n30:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>n30:8088</value>
    </property>

</configuration>

```
配置`slaves`文件，选26-29作DN。

```
n26
n27
n28
n29
```
配置`*env.sh`  
在`hadoop-env.sh`  和 `yarn-env.sh`中`export JAVA_HOME=/usr/java/jdk1.7.0_55`。这个变量是必须引入的，不然脚本里用到了但找不到。如果在系统里已经配置了`$JAVA_HOME`，也需要配置！

### 传送hadoop安装目录到其他机器
OK，在hadoop用户下从26上传送配置好的hadoop安装目录到27-30各机器上：

`rsync -avzrtopgu --progress /opt/hadoop-2.6.0-cdh5.4.4/* n30:/opt/hadoop-2.6.0-cdh5.4.4`  

**注意：**上面的命令传送的时候需要加`-a`参数，不然`软链接`不会被传送。hadoop有软链接`/opt/hadoop-2.6.0-cdh5.4.4/share/hadoop/mapreduce`指向同目录下的`mapreduce2`以此来选lib。我没传过去，是后来手工添加的：
`ln -s mapreduce2 mapreduce`。
查找目录下所有软链接并ls出来：
`find . -type l | xargs ls -l`

## 启动集群

### 格式化datanode
namenode上：
`hdfs -format namenode`

### 启动hdfs
`start-dfs.sh`
### 启动yarn
`start-yarn.sh`
### 测试
跑share下的example中的wordcount。
