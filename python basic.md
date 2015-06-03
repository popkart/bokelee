# Python
## function args

### 1. Common usage

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
### 2. Arbitrary dictionary&tuple  args

```
#define
def f(arg1, *tupleargs, **dicargs):
	...
#！注意tuple要在dic之前

#call
f('foo', 'bar', 'xx', 'yy', key1='what', ky='when')

```
### 3. Unpacking args
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
## 流程控制

```
for i in range(1,5):
	do sth
	if conditin:
		break
else:
	do sth... #只要不是break出来的，for结束时就会执行。 
```
写个fib：

```
>>> def fib(n):
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