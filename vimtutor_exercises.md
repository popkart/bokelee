# vimtutor exercises
---
### 常见操作
```
:w xx.txt #Save as xx.txt

:w 2,10 xx.txt # Partial saving. Save content between line 2 and 10 to xx.txt

:!pwd #execute pwd command. So, :! can execute linux command

:r xx.txt # insert the content of xx.txt into  current cursor position.

:r !ls # insert command 'ls' result into current cursor position.
```
### 跳转类命令

`ctrl+g` : Show current cursor position info.  
先按`行数`，再按大写`G`（go）跳转到指定行，直接`G`则跳到文件尾部,`gg`跳转到首行。  
数字`0`跳到行首。  
`ctrl+o`与`ctrl+i`，o跳转到上一个位置，i是新位置。不知道位置是啥意思。
小写欧`o`在当前光标行`后`插入空行并进入insert模式。  
大写欧`O`在当前光标行`前`插入空行并进入insert模式。  
小写`a`在当前字符后进入insert模式。  
大写`A`在行末进入insert模式。  
大写`I`在行首进入insert模式。  
大写`R`进入replace模式，可替换多个连续字符（不是插入，相当于windows中的insert键关闭）。  
小写`u`是撤销，大写`U`是撤销本行所有操作。而`ctrl+r`是恢复（和小写u撤销相反）.  
输入dd后删除一行，这一行会被置入缓冲区。这时候按`p`可以把缓冲区中的内容粘贴到`光标所在行的下一行`。如果缓冲区不是一行（如一个单词），则按`p`会把缓冲区内容置入光标后。  
`c`应该代表“clean”，注意与`d`删除的区别。`cc`清除当前行为空行并进入insert状态。`ce`删除当前字符开始到当前单词结束内容并进入insert状态，`cw`好像和它一样，`c2w`你懂的。  

### 查找、替换、设置类命令
* 查找  
查找一个单词word可以用`/word`（不需要分号）,查找下一个按`n`，查找上一个按`shift+n`。如果要逆向查找字符串，用`?`替换`/`。
* 替换  
`s/old/new<enter>`:替换当前行第一个匹配.  
`s/old/new/g`:`/g`代表替换当前行所有出现。
`#,#s/old/new<enter>`:两行之间的全替换。
`%s/old/new/g`:全文替换。
`%s/old/new/gc`：`c`代表询问每个替换。
* 设置命令
设置查找忽略大小写：`:set ic`(ic, Ignore Case)。相反：`set noic`. 或者**单次命令**忽略大小写`/searchword\c`。 
设置高亮（hlsearch）、增量查找（incsearch，敲查找字符的时候就开始匹配）：`set hls is`
### 帮助、版本、vimrc脚本、命令自动完成
F1、`:help`都可以打开帮助界面。`:help set`查看set的帮助页面。  
帮助文档看来也是遵循一定标记语言格式，如下面的内容：

```
|41.1|  Introduction
|41.2|  Variables
```
可以用命令`help 41.1`跳转到Introduction，这是41.1的开头文本：


```
=======================================
*41.1*  Introduction                            *vim-script-intro* *script*

```

`:version`查看vim的版本信息，里面有vim的一些信息，如资源文件位置、支持信息等。

```
  system vimrc file: "$VIM/vimrc"
     user vimrc file: "$HOME/.vimrc"
      user exrc file: "$HOME/.exrc"
  fall-back for $VIM: "/usr/share/vim"
 ```
 资源文件里可以包含任何在vi命令模式":"后面出现的东东，通常是保存一些设置信息，如`set incsearch`.  
 导入vimrc示例文件`$VIMRUNTIME/vimrc_example.vim`。More information see `help vinrc-intro`.  
 首先保证运行在非兼容模式下：`set nocp`，然后输入命令可以用`tab`键补全，按`ctrl＋d`键提示所有可能命令。
### 括号匹配
把光标置于括号位置。按`%`键光标跳到匹配括号处。再按返回。 
### 复制粘贴
`y`，yank，代表复制。可以：按`v`进入可视化模式，移动光标选择内容，按`y`进行复制。可按`p`粘贴。  
`yw`复制一个单词，举一反三。 
### 可视化编辑 v
