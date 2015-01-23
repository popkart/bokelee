# Git学习
---
## 前言
大家都知道Linux是Linus创建的，但Linux的壮大却应该归功于全球为完善Linux而贡献代码的coder，但这么多人如何协同开发呢？因为CVS、SVN这类集中式的版本控制系统有速度慢、必须联网的缺点，Linus并未使用它们。最开始大家都把为Linux写的代码发给Linux，由他来整理，但是这个工作随着Linux的渐渐庞大而变的越来越困难。2002年的时候，Linus选择了一家商业公司的版本管理系统来管理代码，但在2005年因为一些原因闹掰，于是Linus花了2周时间用C写了一个分布式的版本管理系统，这就是Git，最先进的分布式版本控制系统。08年GitHub上线。  
分布式版本控制系统，每个人的电脑上都是一个完整的版本库。如果两个人都修改了同一个文件，就把修改的地方互相推送给对方好了。但是事实上，分布式系统也常常有一台充当服务器的机器（如GitHub），注意这个机器只是为了方便大家“交换”修改，没有它大家也能照常干活，只是“交换”不方便了而已。  
Git的优点不止不联网这些，还有强大的分支管理。
## Git Start
安装好Git，配置自己的全局信息，和别人连接的时候自报家门：

	$ git config --global user.name "Your Name"
	$ git config --global user.email "email@example.com"
之后默认机器上所有的Git仓库都是这一个配置，当然也可以对不同的仓库配置不同的用户名和email地址。
### 创建版本库
我的理解是每个项目是一个版本库。  
找一个空目录，然后初始化一个空的版本库：

	$ git init
	Initialized empty Git repository in /Users/lz/programs/gittest/.git/
### 添加和提交文件
这时候我们就能往仓库里添加东西了。内容当然必须放到创建版本库的目录里。我们在目录里新建一个README.md文件，然后执行添加命令：

	git add README.md
OK，添加完毕，然后提交到本地仓库：

	git commit -m "first readme file"
	[master 3e7c925] first readme file
	 1 file changed, 1 insertion(+)
	 create mode 100644 README.md
说明提交完毕。当然我们可以提交多次修改之后，然后commit。
### 查询状态
修改文件后，我们用`git status`可以随时查询git仓库的状态。

```
lzm:gittest lz$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   start.md

no changes added to commit (use "git add" and/or "git commit -a")
lzm:gittest lz$ 
```
用`git diff start.md`来查看start.md的修改内容，显示的格式是Unix通用的diff格式。

```
lzm:gittest lz$ git diff start.md 
diff --git a/start.md b/start.md
index 24284b1..f6b88bb 100644
--- a/start.md
+++ b/start.md
@@ -1 +1,3 @@
-## I'm a new start!
+## I'm a nw start!
+---
+Only because saw U.
```
修改了文件后，依然要执行 `git add`和`git commit`命令才算提交完毕。

### 版本回退
每一个 `git commit`都是一个“快照”，可以恢复到某个commit之后。  
使用`git log`来查看版本变更信息。看到的一大串SHA串是版本号。可以加`--pretty=oneline`参数来让一个commit显示到一行上。  
Git的当前版本用`HEAD`表示，上一个版本是`HEAD^`，再上个是`HEAD^^`。。。往上100个版本是`HEAD~100`。现在我们回退到上个版本：

	lzm:gittest lz$ git reset --hard head^
	HEAD is now at 0333e99 modify
这时候用`git log`查看发现最新的版本已经不在了。如果想回到最新的版本肿么办？如果你还记得最新版本的“commit id”的话，可以用下面的方法过去：

	$ git reset --hard 3628164
	HEAD is now at 3628164 xxx
id不用写全，前几位就可以了。Git内部有个head指针，当你回退时，只是把head指针指向需要回退的版本，然后更新工作区的文件。  
记不住“commit id”怎么办？Git提供了一个命令`git reflog`，可以__查看历史操作记录__:

```
lzm:gittest lz$ git reflog
0333e99 HEAD@{0}: reset: moving to head^
8485f3a HEAD@{1}: commit: nothing
0333e99 HEAD@{2}: commit: modify
3e7c925 HEAD@{3}: commit: first readme file
3871b6f HEAD@{4}: commit (amend): Initial comment
832ff62 HEAD@{5}: commit (initial): Initial comment
```
### Git暂存区工作区版本库
Git有一个**“暂存区”**的概念。先来看几个概念：  
工作区（Working Directory）：你执行git命令的那个地方，不用多解释。
版本库（Repository）：工作区文件夹里的隐藏目录`.git`，这个不算工作区，而是版本库。里面存了很多东西，包括叫做“Stage”或者“Index”的暂存区。还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针`HEAD`。  
![gitTree](img/trees.png)
`git add`把修改内容提交到暂存区，然后`git commit`把修改提交到当前分支。我们在创建版本库时Git为我们创建了一个`master`分支，所以我们commit其实是往`master`分支上提交更改。commit之后暂存区就空了，见下图（图片来自廖雪峰blog）：  
![After commit](img/testcacheljf.jpeg)
### 撤销修改
当修改了一个修改又突然不想修改时，可以：

	use "git checkout -- <file>..." to discard changes in working directory
如果暂存区有东西，则恢复到暂存区的状态，如果没有，则恢复到上一个版本状态。  
如果你已经add到暂存区了，使用`git reset HEAD fileName`可以撤销哦,清空暂存区，使工作区回到add之前的状态，然后再用checkout那个恢复文件到修改前状态。
### 删除
要删除一个文件，先要把它从硬盘上删除，然后使用`git rm filename`提交到缓存区，然后commit到仓库。
### 远程仓库（GitHub）
没创建秘钥对就创建一个：

	lzm:.ssh lz$ ssh-keygen -t rsa -C "lzru@qq.com"
	Generating public/private rsa key pair.
然后登陆GitHub，打开“Account settings”，“SSH Keys”，把生成的公钥`id_rsa.pub`里的内容粘贴上。这样，你就有访问GitHub库的权限了。  
把公钥文件里的内容拷贝到剪切板：

```
$pbcopy < ~/.ssh/id_rsa.pub
# Copies the contents of the id_rsa.pub file to your clipboard
```
然后我们在GitHub上创建一个空的库。在本机上工作区目录执行下面的命令把本机内容推送到远程库里：

	lzm:gittest lz$ git remote add origin git@github.com:popkart/bokelee.git
`origin`是远程库的名字，是github默认的，也可以改成别的。下面把本地分支`master`推送到远程库：

```
lzm:gittest lz$ git push -u origin master
Counting objects: 15, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (15/15), 1.28 KiB | 0 bytes/s, done.
Total 15 (delta 1), reused 0 (delta 0)
To git@github.com:popkart/bokelee.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```