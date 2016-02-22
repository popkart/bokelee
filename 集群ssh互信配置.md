# 配置集群ssh互信
---

假设集群中各服务器分配的用户为hadoop。需要先建立集群各节点间hadoop用户ssh无密码互信。  
现在以node26 - 30共5台机器上的hadoop用户间建立无密码互信为例：  

1. 在26-30每台机器（26也需要这一步）上生成ssh密钥，并将密钥添加到node26的/home/hadoop/.ssh/authorized_keys文件中，完成从26-30机器登录node26免密码。  
下面以在node27上操作为例，其他机器操作相同。先登录到27的hadoop用户下，执行下面的命令，在必要的时候需要输入对应用户的密码：  
  ```
[hadoop@node27 ~]$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identifhadooption has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
a5:74:e8:43:bb:4c:f3:d6:5d:76:f7:04:3a:6e:19:92 hadoop@node27
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         .       |
|        + o   .  |
|       + = . . . |
|        S E +   *|
|       o = + = ++|
|        o o = . .|
|         . .     |
|                 |
+-----------------+
[hadoop@node27 ~]$ ssh-copy-id hadoop@node26
The authenticity of host 'node26 (node26)' can't be established.
ECDSA key fingerprint is c3:c7:e8:df:72:f8:f8:2e:8e:81:24:f4:30:11:2b:51.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@node26's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'hadoop@node26'"
and check to make sure that only the key(s) you wanted were added.
  ```
  通过上面的操作，已经可以在26-30这5台机器上免密码登录26的hadoop用户。测试如下：  
  ```
[hadoop@node28 ~]$ ssh hadoop@node26
Last login: Thu Jan 18 16:19:00 2016 from n26
[hadoop@node26 ~]$
  ```
3. 将26机器上的/home/hadoop/.ssh/authorized_keys文件拷贝到27-30机器相同目录下。  
下面以在node27上操作为例，其他机器操作相同，在必要的时候需要输入对应用户的密码：  
  ```
[hadoop@node26 ~]$ scp /home/hadoop/.ssh/authorized_keys hadoop@node27:/home/hadoop/.ssh/
hadoop@node27's password: 
authorized_keys                      100% 1215     1.2KB/s   00:00    
[hadoop@node26 ~]$
  ```

至此26-30的5台机器间hadoop用户ssh免密码互信配置完毕。
