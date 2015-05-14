# Linux用户管理
---
* linux认识的是ID。账号和ID的对应关系在`/etc/passwd`里。登陆者会有2个ID：uid和gid（group ID）。

		#/etc/passwd里：
		hadoop:x:500:500::/home/hadoop:/bin/bash
		#注意第一个500是uid，第二个500是gid。文件等都会记录owner uid等。如果把uid改了的话。。
* ssh login:
	1. 核对用户名是否在/etc/passwd里
	2. 去/etc/shadow核对密码
* /etc/passwd结构
	
		#第一行是：
		root:x:0:0:root:/root:/bin/bash
		以前的Unix密码就放第二个字段，后来因为安全问题移到/etc/shadow里了，所以这里是个x。
> uid＝0的是管理员账号。当然可以设置多个管理员，但是一般只有root一个。  
> uid在1-499一般是给系统用的，一些服务啥的使用到的。  
> uid在500以上是用户用的，一般我们的uid都是500开始的～  
> gid是与/etc/group相关的  
> 第5个字段是账号信息说明  
> 第6个字段是用户的home目录  
> 第7个字段是登录预设shell。设置成/sbin/nologin就是禁止用户shell登录。
* /etc/shadow结构

		root:$6$coxavcYw4/LCtRW8$2Lb3qHqxNOt1axaHwHXFfp7tLIns8epr2i.ZnWcRYirCR7ps3oMTUk1kVIX/nQM9R8scbo3sA4ixsykC3mDOd/:16502:0:99999:7:::
> 字段1:用户名  
> 2:加密的密码  
> 3:最近改动秘密的日期（相对1970年的天数）  
> 4:密码不可被改变的天数，从第三个字段那天开始，在这个天数内密码不能修改。。为0表示随时可以改
> 5:密码强制过期天数（必须在这些天内修改）
> 后面的也是一些天数啥的设定
* /etc/group结构

		root:x:0:
		bin:x:1:bin,daemon
> 字段1:组名  
> 字段2:密码，同/etc/passwd的那个  
> 字段3:gid  
> 字段4:群组支持的账号名称，把账号放这里就加入这个群组了，注意用逗号分割，不能有空格。注意到root那个组里为什么后面是空的？root账号不应该放在字段4吗？且听分解：  
> 
> ***初始群组和当前有效群组***  
> ```
> passwd文件里指定的用户群组是用户的初始群组，用户默认加入这个群组，
> 所以group文件里该群组最后没有这个用户名。
> 用户一登录系统就具有初始群组的权限。
> 但是我们用加入多个群组的用户新建立一个档案它的owner group是谁呢？是用户当前的有效群组。
> ```
> * groups 命令 ：显示当前用户加入的群组，显示的第一个群组是用户的当前**有效群组**。
> * newgrp 命令 ：切换当前用户的有效群组。 用法：`newgrp 群组名`。当然必须是用户已经加入的群组才能切。
> * usermod 命令：加入一个用户到群组中（需root权限）
* /etc/gshadow 群组管理员等，略。
* useradd 添加用户

		root@www ~]# useradd [-u UID] [-g 初始群组] [-G 次要群组] [-mM]\		> [-c 说明栏] [-d home绝对路径] [-s shell] 使用者账号名		参数（有的参数会修改上述passwd、group等文件相应字段的内容哦）:		-u :指定UID 
		-g :initial group 名称~		-G :该用户还要加入的另外的群组。		-M :强制!不建立用户home目录！(系统账号默认值)		-m :强制!要建立用户home目录!(一般账号默认值)		-c :这个就是 /etc/passwd 第5个字段的说明文字~ 		-d :指定home目录,务必使用绝对路径!		-r :建立一个系统账号,这个账号 UID 会有限制(参考 /etc/login.defs) 		-s :后面接一个 shell ,若没有指定则为 /bin/bash ~		-e :后面接一个日期,格式为『YYYY-MM-DD』此项目可写入 shadow 第8字段,		账号的实效日期;		-f :指定密码是否会失效。0 为立刻失效,-1 为永不失效（shadow第7字段）
用`useradd -D`可以查看useradd的默认设置。保存在` /etc/default/useradd `文件里，可以修改之。