# Markdown 语法学习摘要
---
* [前言](#qianyan)
* [Markdown对HTML的支持]
* [字符自动转换]
* [区块元素](#blockElem)
   * [段落和换行]
   * [标题]
   * [区块引用Blockquotes]
   * [列表]
   * [分割线]
* [span Elements](#spanElem)
   * [链接]
   * [强调]
   * [代码（code）](#code)
   * [代码区块]
   * [图片]
* [其他的一些内容]
   * [自动链接]
   * [反斜杠]
* [MarkDown扩展](#extension)
   * [锚]
   * [strikethrough删除线（有的不支持）]
   * [角标]
   * [Table]

## [前言](id:qianyan)
Markup Language（标记语言）有很多种，如HTML、reStructuredText（简称ReST，Python文档使用这个，有解释工具DocUtiles）等。MarkDown也是一种标记语言，写md文档和直接写Html源码的感觉类似，但是它简化了Html的标记语法，使之更容易书写和理解。通过md解释工具（与解释Html源码的浏览器对应）可以将md书写的内容显示出来。  
下面是最基本的md语法，最后根据Mou的doc增加了几个扩展标记。发现Mou的md reference简单明了，很快能学到基本标记。

## Markdown对HTML的支持
* HTML block类标签（如div、table等）。只需要把html标签的代码直接嵌入即可。注意两点：
    1. 代码段前、后需要空一行。
    2. 代码段最外层的标签前面不能有缩进。
    3. 在block里的md语法将不会被解析。
* HTML inline标签（如 \<span>、\<cite>、\<del>等）。可以和md标签混用，比如如果喜欢Html的\<a>标记可以直接写，和用md的链接表示方法效果一样。代码实例：

		It's <span>**important**</span>! 
	显示效果：	
	> It's <span>**important**</span>! //这是显示效果

## 字符自动转换
HTML文件中有2个字符需要特殊处理才能显示出来，<(\&lt;)和&(\&amp;)。在md里这些都不用担心，可以直接写“<”。同时，你在md里直接写实体形式“\&lt;”也会被显示为字符的原型“<”的，因为Html的那一套md也能解析（上面说了），如可以用\&copy;来表示&copy;，Html能做的md也支持。

md还有“code”的标记，里面的“<”之类的内容它都会自动转义，最后显示的和你写的是一样的字符，不用你来用实体的形式表示，所以你只管写就是了。

还有一点，md的“\”后面的一个字符原样显示（算手动转义一个字符），你懂的。

## [区块元素](id:blockElem)
### 段落和换行
可以用两个回车（空一行）来表示换段（\<p>元素），也可以用两个空格＋一个回车（正常换行，第一行结尾加2个空格）来表示换行（\<br>元素）,当然直接写Html的“\<br/>”也可以（不过看着恶心）。
只有一个回车的话，并未换行，只是多了一个空格。  
注意到：md的段落都**没有行首缩进**。
### 标题
md支持两种形式，类 [Setext](http://docutils.sourceforge.net/mirror/setext.html) 和类 [atx](http://www.aaronsw.com/2002/atx/) 形式。我觉得atx的比较舒服，用“N个＃＋空格＋标题”表示N级标题（最多6级）。

	# 这是 H1

	## 这是 H2

	###### 这是 H6

### 区块引用Blockquotes
md使用类似email引用的“>”方式。代码：

	> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
	> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
	> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
	> 
	> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
	> id sem consectetuer libero luctus adipiscing.

显示效果：
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.

可以偷懒只在第一行写“>”：

	> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
	consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
	Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

	> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
	id sem consectetuer libero luctus adipiscing.
引用可以嵌套，可以用md语法标记，代码：

```
> ##我是标题
> This is the first level of quoting.
> 
> > This is **nested** blockquote.
> 
> Back to the first level.
> 
> * list1
> * list2 
```

效果：
> ##我是标题
> This is the first level of quoting.
> 
> > This is **nested** blockquote.
> 
> Back to the first level.
> 
> * list1
> * list2

### 列表
* 无序列表：“*-+”任一字符开头＋空格＋列表内容。
* 有序列表：数字＋“.”+空格＋列表内容(列表第一项数字最好从1开始，后面的数字只要是数字就行，是几都一样，它只会＋1、＋1的往下排)

列表可以包含段落，但是每个段落开头要缩进4个空格或者1个tab。

列表可以包含引用，引用也要缩进。

列表如果包含代码，代码要缩进2次（在列表外的代码就缩进一次）。

### 分割线
“*-_”之一三个以上在一行上，且无可见字符就是分割线，如：

```
***
---
___ 
```
## [span Elements](id:spanElem)
### 链接
两种方式：

1. *一般方式*：

	代码如下：
		
		[锚文本](http://www.baidu.com "title")
	效果：
	我是[锚文本](http://www.baidu.com "title")
2. *参考文献引用方式*:

	代码如下：
	
		文章中引用这样写：[锚文本][id]
		下面是引用的参考，一般放在最后。
		它的优点为，文章的正文比较简洁，只有一个引用ID，而且参考可以重用。  
		格式为：
		[id]＋：＋空格＋URL＋空格＋“双引号/括号”括住的title。
		注意，参考的左边不能有4个及以上空格或者一个及以上tab，最好顶着左边写。
		[id]: http://www.baidu.com "title2"
	效果：
	这个[我是通过引用来的][idx]

[idx]:http://www.baidu.com "alter2"

### 强调
*和_包围住文本，一个转化为"<em></em>"，两个转化为“<strong></strong>”,三个是斜体＋黑体。

代码：

	这是em要*强调*的内容。
	这是strong要**强调**的内容。
	这是em&strong要***强调***的内容。	
效果：

> 这是em要*强调*的内容。  
> 这是strong要**强调**的内容。  
> 这是em&strong要***强调***的内容。

### [代码（code）](id:code)
标记一小段行内代码，用反引号\`code内容\`标记出来（产生\<code>标记）。第一个反引号后和第二个反引号前均可以有一段*空白*，这样能把“\`”也包进去，如\`\` \` \`\`结果为：`` ` ``，可用多个\`来包裹里面的一个\`。
### 代码区块
HTML中用“\<pre>\<code>\</code>\</pre>”来包裹代码，md用4个空格或一个tab缩进来表示(md转Html后会也会转换成这4个标记)。

还可以用**空行**＋连续3个以上"`"的闭合对来表示(**扩展功能**)，如代码：
	
	````
	<html>
	我被包裹了我是代码
	</html>
	````
效果：

````
<html>
我被包裹了我是代码
</html>
````
### 图片
图片和链接的很相似，也有一般方式和参考引用方式两种。图片比链接前面多个感叹号。  
代码：

	![alt text](http://su.bdimg.com/static/xtpl/img/music/music_logo_b5071b52.png "可选 title")
效果：
> ![alt text](http://su.bdimg.com/static/xtpl/img/music/music_logo_b5071b52.png "可选 title")

参考式代码：

	![Alt text][id]
	[id]: url/to/image  "Optional title attribute"
***md现在还不支持图片宽高设定，如有需要只能使用HTML的\<img\>元素***。

## 其他的一些内容
### 自动链接
	<http://example.com/> or <lz@qq.com>
Markdown 会转为：

> <http://example.com/> or <lz@qq.com>

源代码是酱紫的：

	<a href="http://example.com/">http://example.com/</a> or <a href="mailto:lz@qq.com">lz@qq.com</a>
### 反斜杠
你懂的

## [MarkDown扩展](id:extension)
### 锚
比如可以把每一个标题后面加一个Anchor，然后在目录里引用，点击跳转。代码：

	## [This is an example](id:anchor1)
	用下面的代码引用，其实就是链接那种语法，URL变成了锚：
	* [第二章](#anchor1)
目录示例：

```
*   [概述](#overview)
    *   [宗旨](#philosophy)
    *   [兼容 HTML](#html)
    *   [特殊字符自动转换](#autoescape)
*   [区块元素](#block)
    *   [段落和换行](#p)
    *   [标题](#header)
    *   [区块引用](#blockquote)
    *   [列表](#list)
    *   [代码区块](#precode)
    *   [分隔线](#hr)
*   [区段元素](#span)
    *   [链接](#link)
    *   [强调](#em)
    *   [代码](#code)
    *   [图片](#img)
*   [其它](#misc)
    *   [反斜杠](#backslash)
    *   [自动链接](#autolink)
*   [感谢](#acknowledgement)
*	[Markdown 免费编辑器](#editor)
```     
### strikethrough删除线（有的不支持）
代码示例：
>`~~我被弃用，删掉了~~`显示效果：  
> ~~我被弃用，删掉了~~

### 角标
代码：

	That's some text with a footnote.[^1]

	[^1]: And that's the footnote.
效果：
> That's some text with a footnote.[^1]
>
> [^1]: And that's the footnote.

### Table
*代码*：

```
A simple table looks like this(注意前后的空行):

First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell

If you wish, you can add a leading and tailing pipe to each line of the table:

| First Header | Second Header | Third Header |
| ------------ | ------------- | ------------ |
| Content Cell | Content Cell  | Content Cell |
| Content Cell | Content Cell  | Content Cell |

Specify alignement for each column by adding colons to separator lines（竖直对齐方式，冒号控制）:

First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right
```
*效果*：  
A simple table looks like this(注意前后的空行):

First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell

If you wish, you can add a leading and tailing pipe to each line of the table:

| First Header | Second Header | Third Header |
| ------------ | ------------- | ------------ |
| Content Cell | Content Cell  | Content Cell |
| Content Cell | Content Cell  | Content Cell |

Specify alignement for each column by adding colons to separator lines（竖直对齐方式，冒号控制）:

First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left         | Center        | Right
Left         | Center        | Right


