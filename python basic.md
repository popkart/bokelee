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
自己创建的package，每个目录里必须有`__init__.py`这个文件才会被视为一个package，即使这是个空文件。它里面可以包含package的初始化代码或者`__all__`变量。  
导入包里的模块：

	import Sound.Effects.echo #这种方式import的最后只能是包或者模块，且只能用完整名称来引用，如Sound.Effects.echo.echofilter(input, delay=0.7)。注意这个和模块的引用的区别
	#或者：
	from Sound.Effects import echo #这种方式import那边可以是模块或者函数，同时from最后个可以是包或者模块，且可以用echo.echofilter()
	
而`from package import *`时，如果`__init__.py`文件定义了`__all__`:  

	__all__ = {"echo", "surround", "reverse"}
python会按照它来加载函数，不在它里面的就不加载了。 

## 输入输出
str() repr()  
### 文件

	f = open(fileName, mode)
	f.read(size) #返回size个字符的字符串，不指定size则读完。
	f.readline() #读一行，最后有'\n'，只有文件最后一行没有换行符（如果最后一行就是一个换行符呢？），返回一个空字符串。
	f.readlines(sizeint) #读多行返回一个list。没有sizeint读完。
	
	#loop over f，不用全读完，速度快
	for line in f:
		print line
	
	f.write(string) #写入string到文件。string需要包含’\n‘
	f.tell() #返回指针位置，自文件开头到指针处的字节数。
	f.seek(offset, from_what) #移动指针。offset 可以为负，from_what：0，文件开头，默认；1，当前位置；2，文件末尾。
#### 序列化

	pickle.dump(x,file) #x 存入文件file
	x = pickle.load(f) #反序列化

## 错误和异常

1. 捕获异常

```
try:
	...
	...
except IOError:
	...
except(RuntimeError, TypeError):  #多个Error
	...
except:	#其他异常会进入这个省略异常名的except
	print "other errors"
	raise #抛出去
else:	#try块可以附带一个else语句，如果未抛出异常，则执行。它是为了避免该try的except意外截获该else代码段里抛出的异常（相对于将else内代码写入try内）。
	...
finally:	#不管怎样都会执行，不管在try/except/else中发生异常、被continue/break/return、没发生异常，都会在之后执行。
	...
```

2. 处理异常信息	
except 的第二个参数可以是一个实例，就是抛出的异常实例。
	
	except IOError, e:
		print repr(e.args)
		x,y = e	#(把异常的2个属性赋予x，y)
		pring x,y 
我们可以通过e来得到一些信息，比如，Exception类内建的一些参数，message，args。  
args是异常作者在抛出异常时指定的参数列表（异常编号，异常信息字符串等），也可以用message.__getitem__(index)来访问第index（从0开始）个参数。  
python中喜欢直接用元组来接收这个实例e，比如IOError实例附带2个参数，因此一般这么接收：

	except IOError,(x,y):
		print x,y

3. 抛出异常

```
try:
	rasie Exception('xx','yy')	#抛出含有2个参数的异常，下面那个写法风格更好点？
	raise Exception,('xx','yy') #更常用的写法
	raise IOError,('a', 'b', 'c', 'd')	#伪造一个IOError
```

4. 用户定义异常		

```
>>> class MyError(Exception):
...     def __init__(self, value): 
		"""注意这里覆盖了Exception的初始化方法，
		   故，
		   args这个参数列表一直为空(打印出来是个”()“)，我们用value这个参数来代替
		”“”
...             self.value = value*2
...     def __str__(self):	#打印
...             return repr(self.value)
#测试下：
>>> try:
...     raise MyError,4
... except MyError,ins:
...     print ins
...
8
#再扯点别的，Python的动态xx
>>> def wt(self):
...     return 'xxx:'+repr(self.value)
>>> MyError.__str__ = wt
>>> try:
...     raise MyError,4
... except MyError,ins:
...     print ins
...
xxx:8

和javascript一样，动态改变类的行为。

```

common practice：定义一个异常基类，针对不同的异常派生子类。大多数异常命名都以`Error`结尾。

## 类
哎呀这个类。。。像类吗？感觉是多套了一层的模块。

```
class ClassName:
    <statement-1>
    .
    .
    .
    <statement-N>

```

类语句需要先执行才能生效，可以把它放在代码块或者函数里。。  
支持`属性引用`和`实例化`两种操作。  

```
class MyClass:
    """A simple example class"""
    i = 12345
    def f(self):
        return 'hello world'
```

***如果***定义了`__init__()`方法，实例化的时候会执行它，类似构造函数。

```
>>> class Complex:
...     def __init__(self, realpart, imagpart):
...         self.r = realpart
...         self.i = imagpart
...
>>> x = Complex(3.0, -4.5)
>>> x.r, x.i
(3.0, -4.5)
```

### 不在类体内的方法，一样能挂到类上

```
# Function defined outside the class
def f1(self, x, y):
    return min(x, x+y)

class C:
    f = f1
    def g(self):
        return 'hello world'
    h = g
    
   ```
 f,h,g都是C的方法。这也体现了python的动态性，可以随时给一个类加一个function。。。
### self
注意到方法第一个参数，`self`，python以前难道没有类？这个东西像是用函数曲线实现了类的方法。如果 `x = new C()`，则`C.g(x)`和`x.g()`效果一样。也就是后一种`instance.method`的写法等价于把自己作为第一个参数，和参数列表一起传给了C的函数。

### 每一个值都是对象
每个值都是对象，因此都有一个类。用`type(object)`查看object的类型，或者用`object.__class__`。  
### 继承

	class DerivedClassName(modname.BaseClassName):

如果在类中找不到请求调用的属性，就搜索基类。如果基类是由别的类派生而来，这个规则会递归的应用上去。父类搜索路径是从左到右，深度优先（旧式类）。新式类**super()**可以动态的改变解析顺序。为防止`菱形的继承`(多个父类有共同的祖先类)，每个祖先类只调用一次。	  
子类也可能覆盖父类方法，这时候可以用`BaseClassName.methodname(self, arguments)`来调用父类方法，但注意要import BaseClassName。

Python 有两个用于继承的函数：

**isinstance()** 用于检查实例类型： `isinstance(obj, int)`为`True`只有在 `obj.__class__` 是 **int** 或其它从 **int** 继承的类型。  
**issubclass()** 用于检查类继承： `issubclass(bool, int)` 为 `True` ，因为 **bool** 是 **int** 的`子类`。但是， `issubclass(unicode, str)` 是 `False` ，因为 **unicode** 不是 **str** 的子类（它们只是共享一个通用祖先类 `basestring` ）。

### 私有变量
python中严格说没有私有变量。约定以一个下划线开头的命名，会被处理为API的非公开部分。  
Python 提供了对命名的有限支持，称为 `name mangling`（命名编码） 。任何形如 `__spam` 的标识（前面至少两个下划线，后面***至多一个***，排除了`__init__`之类），被替代为 `_classname__spam` (之后动态加上去的属性不算)

```
class Mapping:
    def __init__(self, iterable):
        self.items_list = []
        self.__update(iterable)

    def update(self, iterable):
        for item in iterable:
            self.items_list.append(item)

    __update = update   # __update私有了，看dir()结果：
 >>> dir (Mapping)
 ['_Mapping__update', '__doc__', '__init__', '__module__', 'update']

class MappingSubclass(Mapping):

    def update(self, keys, values):
       #注意这里覆盖了update方法，但是，不影响__update这个方法哦
       #也就是不影响父类的__init__方法的调用
        for item in zip(keys, values):
            self.items_list.append(item)
            
```
>>> 要注意的是代码传入 exec ， eval() 或 execfile() 时不考虑所调用的类的类名，视其为当前类，这类似于 global 语句的效应，已经按字节编译的部分也有同样的限制。这也同样作用于 getattr() ， setattr() 和 delattr() ，像直接引用 `__dict__` 一样。

### addtional
实例方法对象也有属性：`m.im_self` 是一个实例`方法`所属的`instance`，而 `m.im_func` 是这个方法对应的`函数`对象(注意这个不是方法！)。  
python的动态性让这些可以很随意实现，比如一个"结构体"：

```
class Employee:
    pass

john = Employee() # Create an empty employee record

# Fill the fields of the record
john.name = 'John Doe'
john.dept = 'computer lab'
john.salary = 1000
```

