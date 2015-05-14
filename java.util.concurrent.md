# java.util.concurrent练习
---
文章链接：[java.util.concurrent - Java Concurrency Utilities, by Jakob Jenkov](http://tutorials.jenkov.com/java-util-concurrent/index.html)  
作者是一个自由职业的软件工程师。

OK, let's go!
## Interface:BlockingQueue
一个线程安全的队列，可以安全地放进取出。

 \ |Throws exception | Special value | Blocks | Times out
-- | ----------      | -------       | -----  | ---------
 Insert |  add(e)  |offer(e) | put(e)  | offer(e, time, unit)
Remove	|remove()	|poll()	|take()	|poll(time, unit)
Examine	|element()	|peek()	|not applicable	|not applicable

解释下：第一列的方法如果出问题（如，插入时，发现队列满了）会抛异常，第二列会返回特定的值（true/false），第三列会等待（如，put方法会等到队列出现空闲位置）。第四列会等一段时间，然后像第二列一样返回一个值指示是否成功。
它可以操作队列中的任何元素，并不局限于开头和结尾，但是效率会受影响，一般不推荐这么做。
它的实现：

1. ArrayBlockingQueue
2. DelayQueue
3. LinkedBlockingQueue
4. PriorityBlockingQueue
5. SynchronousQueue  
下面一个一个来看.

### ArrayBlockingQueue
它必须在初始化时指定容量，内部是一个数组来存储元素。存储元素遵循FIFO原则。好像Concurrent里的lock都用可重入锁来实现，在初始化时可以指定使用的可重入锁的锁定策略（公平还是不公平，默认不公平，false）。

    BlockingQueue<String> queue = new ArrayBlockingQueue<String>(1024);
    queue.put("1");
    String string = queue.take();
### DelayQueue
延迟队列，进去的元素必须在一定的时间之后才能被取出。元素必须实现`java.util.concurrent.Delayed`接口:

```
public interface Delayed extends Comparable<Delayed> {

 public long getDelay(TimeUnit timeUnit);

}
```

这个TimeUnit是个枚举，从`NANOSECONDS`到`DAYS`。注意到`Delayed`接口继承了`Comparable`接口，这意味着元素间可以比较，***可能***DelayQueue借助这一特点，对元素根据expire时间进行有序释放。