#Shell单双引号的区别
---
我们在输入shell命令的时候有时候按回车键会不小心按到`'`键，这时候再按回车会发现命令没有执行，变成`>`开头的下一行输入了，这是怎么回事？看了下面的内容你就明白了。
  
shell的command line包括`literal（文本）`和`meta（保留字符，如空格、回车、>、&、|$等）`两种。meta中常见的两种：

* IFS:\<space\>、\<tab\>、\<enter\>。
* CR:\<enter\>产生。

如果要把保留字符功能关闭，变成literal，需要quoting处理。常见quoting处理有三种：

* hard quote：单引号。关闭单引号对中所有meta保留字符。
* soft quote：双引号。关闭双引号中某些meta，其他（如$）不关闭。
* escape：反斜线。紧接`\`之后的一个meta字符被关闭。

所以才有这些：

	$cd 'program files'
	$cd program\ files
### 举例
awk命令：

	$ awk '{print $0}' 1.txt
如果大括号外不加单引号，则大括号会被shell解释为meta，但是awk需要一个字符串参数`{print $0}`，因此会报错。所以不能写成`awk {print $0}`，但可以写成`awk \{print\ \$0\}`😄。
