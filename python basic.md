# Python
### function args

#### 1. Common usage

```
#define
 def f(arg1, xx='defaultyy', foo='bar'):
	...

#call
f('foo')
f(arg1='fool',foo='bb')
f('fool','bar')
#wrong call
f()
f('foo',arg1='bar') # duplicate value for argument
f(xx='foo','bar') # non-keyword argument should be ahead

```
#### 2. Arbitrary dictionary&tuple  args

```
#define
def f(arg1, *tupleargs, **dicargs):
	...
#！注意tuple要在dic之前

#call
f('foo', 'bar', 'xx', 'yy', key1='what', ky='when')

```
#### 3. Unpacking args
上一个情况是定义是一个tuple&dic，调用是一个一个的参数，这个刚好相反。

```
#define
def f(vol, foo='foo', bar='bar'):
	...
#call
args={'foo':'f', 'bar':'b'}
f('vol',**args)
或者：
args={'vol':'vv', 'foo':'f', 'bar':'b'}
f(**args)
再把tuple的调用加上：
args={'foo':'f', 'bar':'b'}
tupleargs=['v']
f(*tupleargs, **args)

```
### 流程控制

```
for i in range(1,5):
	do sth
	if conditin:
		break
else:
	do sth... #只要不是break出来的，for结束时就会执行。 
```
###函数

```
>>> def fib(n):
...		"""这里的内容可以用fib.__doc__变量访问到"""
...     result = []
...     a,b,cnt = 0,1,0
...     while cnt < n:
...             result.append(b)
...             a,b,cnt = b,a+b,cnt+1
...     return result
...
>>> fib(6)
[1, 1, 2, 3, 5, 8]
```

如果函数没有返回值，默认返回`None`
#### lambda表达式
创建短小的匿名函数。方式：

	lambda a, b: a+b #返回a+b的值
它只能有一个单独的表达式，只是一个语法技巧。

### 比较操作符

`a < b == c` 测试 `a是否小于b 并且 b等于c`。  
and、 or、 not的优先级：  
A and not B or C 等同于 (A and (not B)) or C。优先级和其他语言相同。  
逻辑 and or的短路性：

```
>>> str1,str2,str3 = "","hhaa","haha"
>>> not_null=str1 or str2 or str3   #注意第二个为真表达式已经为真，跳出了
>>> not_null
'hhaa'
>>> sure_null=str2 and str1 and str3 #测一下and，注意str1为空
>>> sure_null
''
```

	

## 数据结构

### List
[2,3,4,9]
#### Used As Queue和Stack
方法有`append(x),extend(L),insert(i,x),pop([i]),index(x),count(x),sort(),reverse()`等。运用这些方法，可以把链表当做`队列Queue`或者`栈Stack`使用。sort和reverse方法都能直接改变List本身。
#### 函式编程工具
这些东西可以对sequence类东西进行操作。  
有3个内置函数：`Filter(), map(), reduce()`，和List、set、tuple等一起使用很OK的。  
filter(function, sequence)：对sequence里的元素依次调用function，返回True的包含在filter的结果里。  
map(function,sequence...):对sequence里的元素依次调用function，返回的结果包含在filter里。可以传入多个sequence，当然function的参数也要有对应多个。如果某个序列比较短，则空位用`None`代替。  
reduce(function, sequence)：对sequence中的两个元素调用function，计算结果再和第三个元素作为参数调用function，以此类推，最后返回一个结果。  
#### 链表推导式

```
>>> [x*x for x in xrange(1,6) if x<3]
[1, 4]
```

`for`前面是表达式，后面是条件。表达式可后以是嵌套链表，也可以是复杂的函数。`for`后面可以跟条件等。

```
>>> [[x,y,x*y] for x in xrange(1,6) for y in range(2,3)]
[[1, 2, 2], [2, 2, 4], [3, 2, 6], [4, 2, 8], [5, 2, 10]]
>>> [(x,y,x*y) for x in xrange(1,6) for y in range(2,3)]
[(1, 2, 2), (2, 2, 4), (3, 2, 6), (4, 2, 8), (5, 2, 10)]
```

链表推导式可以简化循环的写法，很好很强大。其实他可以实现map、filter等的功能。

#### del
`del 变量名`删除变量。
对于一个List：[1,2,43,2,2,1,3]  
del a[2:4]  
结果：[1,2,2,1,3]

### Tuple
元组也是Sequence（还有字符串，列表等）的一种。

```
>>> t = 1,2,3  #定义一个元组，封装操作
>>> t
(1, 2, 3)
>>> u = ("a","abc"),t #可嵌套
>>> u
(('a', 'abc'), (1, 2, 3))
>>> e = ()   #空元组，长度为0
>>> e
()
>>> one = "hello",  #一个元素的元组，注意逗号
>>> one
('hello',)
>>> x,y,z=t  #元组的拆封，联系函数的元组参数自动拆封。
             #拆封操作也可作用于其他Sequence，而封装操作只会产生元组。
>>> print x,y,z,t
1 2 3 (1, 2, 3)
```

元组是不可变的，但是可以包含可变元素，如List。  

### Set
集合是无序不重复元素的集。集合支持并、交、差等运算。

```
s=set([1, 2, 3, 4])
>>> 1 in s
True
>>> a =set("I'm a human. Hello world!")
>>> a
set(['a', ' ', '!', 'e', 'd', "'", 'I', 'h', 'm', 'l', 'o', 'n', 'r', 'u', 'w', 'H', '.'])
>>> s-a
set([1, 2, 3, 4])
>>> a|s
set(['!', 2, 3, 4, 1, 'I', 'H', '.', 'a', ' ', 'e', 'd', "'", 'h', 'm', 'l', 'o', 'n', 'r', 'u', 'w'])
```


### Dictionary
字典的key只能是不可变类型，故List和包含不可变类型的元组等不能作为key。


```
>>> tel={'jack':121,"rose":222}
>>> tel
{'rose': 222, 'jack': 121}
>>> position=dict([{1,2},{3,4}])
>>> position
{1: 2, 3: 4}
>>> del position[1] #注意这里“1”是key，不是索引
>>> position
{3: 4}
```

遍历字典k，v：
	
	>>> for k,v in positon.iteritems():
	...     print k,v
	...
	3 4

遍历Sequence，带索引：

```
>>> s
set([1, 2, 3, 4])
>>> for i,v in enumerate(s):
...     print i,v
...
0 1
1 2
2 3
3 4
```

同时循环2个或者更多Sequence，用zip()函数：

```
>>> for qi,ai in zip(q,a):
...     print 'What\'s %s? It\'s %s.' % (qi, ai)
...
What's sky? It's tian.
What's land? It's di.
```

sorted()也是一个builtin函数，可以返回一个排过序的List（输入参数可以是set，tuple，dict等）。  
试试dir(__builtins__)查看内建函数。func.__doc__查看函数内的文档。

### 
	
## Module
模块是包含Python定义和声明的文件。文件名是`模块名.py`。模块的模块名可由`__name__`得到。  
引入模块：

	import module_name
从模块里引入函数：

	from module_name import func1,func2
	import module_name.func1
	from module_name import * #导入除了_开头的所有names。
引入模块后，模块内的函数并不可见。但是已经可以用`模块名.函数名`来使用了。  
模块内可包含执行语句（没写在函数里的），他们只在模块第一次导入时执行一次。
模块中的函数有自己的私有变量表，因此模块中的变量不会和函数中的冲突，当然可以在函数中通过`模块名.模块变量`来获取模块中的全局变量。  
模块导入其他模块后（import），被导入的模块名在本模块的全局语义表中类似下面的命令行：

```
>>> dir()
['__builtins__', '__doc__', '__name__', '__package__']
>>> import sys
>>> dir()
['__builtins__', '__doc__', '__name__', '__package__', 'sys']
```

#### 模块搜索路径
import一个模块时，搜索路径为：

	当前目录->环境变量PYTHONPATH中的路径列表->安装目录
事实上，这个顺序的目录是在`sys.path`变量里指定的。
###package
包是模块的一种组织形式。和java类似是按照目录名组织的。`import`语法同样适用。

	import com.apache.module_name.func1

最后一个可以是模块名或者函数名。

		