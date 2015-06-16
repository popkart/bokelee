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




## 模块与import
