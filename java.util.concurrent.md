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
//该元素还有多久过期，0或者负数表示已过期，这个方法的返回值
//是随着时间变化的
 public long getDelay(TimeUnit timeUnit);

}
```

这个TimeUnit是个枚举，从`NANOSECONDS`到`DAYS`。注意到`Delayed`接口继承了`Comparable`接口，这意味着元素间可以比较。因此，元素必须实现`compareTo`和`getDelay`两个方法。DelayQueue借助CompareTo，对元素根据expire时间进行排序，不用遍历整个Queue来检测过期。  
Delayed接口实现可参考[【java 之DelayQueue实际运用示例】](http://www.cnblogs.com/sunzhenchao/p/3515085.html)中缓存的例子。
### LinkedBlockingQueue
内部实现 linked nodes，FIFO，可以初始化时指定大小，如果不指定就Integer.MAX_VALUE。
### PriorityBlockQueue
没大小限制，和java.util.PriorityQueue有同样的排序规则，不能插入null。插入的元素必须实现java.lang.Comparable接口。它的Iterator不保证遵循优先级顺序。
### SychoronousQueue
同步队列。只包含一个元素，一个线程往里放一个元素，就阻塞了。。一个线程去取，如果没有元素，也会阻塞。。其实说是队列不如说是一个汇合点。
### BlockingDeque
阻塞双端队列（接口）。两端都能插入，取出。它同BlockingQueue一样，也有4种类型的方法，分别在失败的时候抛异常、返回一个状态值、阻塞等待、阻塞等待可设超时。  
它也继承了（因为它也是接口）BlockingQueue接口，所以。。可以当BQ用。
### LinkedBlockingQueue
它是BlockingDeque的一个实现。
### ConcurrentMap 接口
实现：ConcurrentHashMap.
### ConcurrentNavigableMap
java.util.NavigableMap添加了Concurrent的操作。对于它产生的subMap也有并发特性。如它的headMap（）、tailMap（）、subMap（）方法得到的map。  
headMap(T toKey)返回Key比toKey小的kv组成的map（是原map的一个view，不是新复制了一个）。subMap（T k1,T k2）返回key大于等于k1，小于k2...还有其他的几个方法。  
### CountDownLatch 闭锁
它是这样一个玩意，可以在初始化时指定一个数值，然后线程们可以用它的await()方法阻塞住，其他控制放行的线程，可以调用其countDown()方法，每次减一，当减为0的时候，大家就一起走了。他主要强调让一或多个线程等待其他线程一系列动作完成后，一起走。

### CyclicBarrier 环状栅栏
这个东东，让参与的N个线程都到达这个点时，大家才能接着走。他是这么创建的，需要指定等待的线程数达到多少（N）：

	CyclicBarrier barrier = new CyclicBarrier(2);
调用barrier.await()方法停住。当停住的个数达到N时，一起放行。还有await的超时方法，如果等待超时还没放行，就自动放行：
barrier.await(10, TimeUnit.SECONDS);
barrier会放行所有线程，当下面4项有一个满足时：

1. 最后一个线程来了（call await（）方法）。
2. 某个等待线程被其他的线程打断了（其他线程call该线程的interrupt（）方法）。
3. 某个等待线程await的timeout了。
4. barrier被调用了barrier.reset（）方法。  

栅栏支持`BarrierAction`，一个Runnable的东东，既在最后一个线程到达栅栏时（调barrier.await()方法，在这个方法里执行BarrierAction.run()）执行。

	Runnable      barrierAction = ... ;
	CyclicBarrier barrier       = new CyclicBarrier(2, barrierAction);
### Exchanger
交换器，2个线程可以通过它来交换东西（泛的是它），它是泛型。
如下：
	
	this.object = this.exchanger.exchange(this.object);
线程调用它的exchange方法，把自己的东西放进去，返回值是另一个线程放的东西。先放的进程会被阻塞直到另一个线程放东西进去。
### Semaphore 信号量（信号灯，旗语）
java.util.concurrent.Semaphore类是一个countingSemaphore，因此它有2个主要的方法：
`acquire（）和release（）`。初始化的时候设置计数值N，这样保证至多有N个线程可以获得它，其他的会阻塞，直到有一个获得的线程release它。  
这就是操作系统里信号量的实现，可以实现一个critical section，还有2个线程间信号传递（一个acquire（），另一个线程release（）。最后你用它实现了一个Producer-Consumer Blocking Queue...）。  
默认Semaphore不保证先acquire（）的线程一定会先进入critical section(非公平调度)，但Semaphore有一个Boolean参数构造函数可以强制实现公平，但是会影响点性能。

###ExecutorService 接口
异步，类似线程池，他是执行池。其实concurrent包里ExecutorService的实现就是一个线程池的实现。

```
ExecutorService executorService = Executors.newFixedThreadPool(10);

executorService.execute(new Runnable() {
    public void run() {
        System.out.println("Asynchronous task");
    }
});

executorService.shutdown();
```

`newFixedThreadPool()`是一个工厂类接口。第一行创建了一个执行池。然后，我们可以给他传Runnable对象让他执行。  
为什么是异步的呢？因为主线程把Runnable任务给执行池后，主线程会继续往下进行。  
注意最后的shutdown（）方法，当执行后，线程池不会接受新的任务，但是已经接受的任务会执行完的。注意不调用shutdown（）方法最后程序不会结束，不知道阻塞在哪里了。concurrent包里它的实现有2个：

1. ThreadPoolExecutor
2. ScheduledThreadPoolExecutor
 
		Executors.newScheduledThreadPool(10); 
		Executor.newSingleThreadExecutor();

怎么使用ExecutorService?  

1. `execute(Runnable)`: 这个方法得不到执行结果。只是异步执行。
2. `submit(Runnable)`: 这个方法返回一个Future对象，可以通过这个future.get()方法可以确定是否正确执行完毕（他会阻塞等待执行完毕的），正确执行完毕它返回null（什么鬼！只为了和下面那个参数是Callable的兼容吧？）。
3. `submit（Callable）`: Runnable.run()不能返回结果，它能。  

	```
	Future future = executorService.submit(new Callable<String>(){
    	public String call() throws Exception {
        	System.out.println("Asynchronous Callable");
        	return "Callable Result";
    	}
	});
	```
我们注意到它重载的call方法是有返回值的，可以使用future.get()方法得到这个返回值。
4. `invokeAny(Collection<? extends Callable<T>> tasks)`: 
执行一个Callable的集合，只要其中一个task完成或者抛出异常，则结束，不返回Future，返回一个Callable的返回值。注意他会阻塞等待返回。

		Set<Callable<String>> s = new HashSet<Callable<String>>();
		s.add(new Callable<String>(){public String call(){return "task1";}});
		s.add(new Callable<String>(){public String call(){return "task2";}});
		String result = executorService.invokeAny(s);

5. `invokeAll(Collection<? extends Callable<T>> tasks)`: 
这货和Any的区别：它返回一个Future结果的List，而且会阻塞等所有的task执行完。可以用future.get()得到task执行的结果。  
经测试如果task抛异常惨不忍睹啊，这个池子貌似不会自动结束？？

6. `shutdown()`: 
停止接收新任务，并等待所有task执行完毕后，才退出。注意即使main方法执行完毕，只要有任务没执行完`executor就会阻止jvm退出`。  
要想立即停止所有任务，可执行executorService.shutdownNow()。它会跳过未执行的task，并尽最大努力试图停止所有正在执行的task。注意它不保证能停止掉这些task，结果是不确定的。

### ThreadPoolExecutor
```
ExecutorService threadPoolExecutor =
        new ThreadPoolExecutor(
                corePoolSize,//如果有新的task，且当前执行线程数小于他，则创建一个新线程
                maxPoolSize,//核心线程都在跑着，如果来了新task，且当前线程数小于他，则创建新线程
                keepAliveTime,//超出核心线程个数的线程在idle时的生存时间
                TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>()
                );
```
当然这些参数不是必须的，可以用固定数量的线程池，或者设置某些属性。

### ScheduledExecutorService
这是一个接口。继承自`ExecutorService`，多了一些Schedule的部分。

```
ScheduledExecutorService scheduledExecutorService =
        Executors.newScheduledThreadPool(5);//创建一个5线程执行器

ScheduledFuture scheduledFuture =
    scheduledExecutorService.schedule(new Callable() {
        public Object call() throws Exception {
            System.out.println("Executed!");
            return "Called!";
        }
    },
    5,//几秒后执行
    TimeUnit.SECONDS);
```
创建了这个Service后，可以调用它的方法来添加task，schedule（xx），还可以周期性调用一个任务，见接口的定义。

### Fork&Join（jdk1.7新增）
把一个任务split成多个子任务（fork），执行完毕后把结果join成一个。  
java.util.concurrent.ForkJoinPool类。
可以提交给他的任务有两种：

1. 继承自RecursiveAction， 不带返回值的。
2. 继承自RecursiveTask，带返回值的。

这两个类都要实现一个 `computer（）`方法，只不过第二个可返回参数。在computer里根据情况创建subTask实例们，然后调用subtask.fork()使之运行，最后结果可用subtask.join（）得到。
用forkJoinPool.invoke(任务实例)来提交任务。

### Lock
和Synchronized类似，但更灵活精致。区别：

1. synchronized不保证等锁的线程获得锁的顺序。
2. 不能给synchronized块传递参数，因此无法设置访问等待超时时间。
3. synchornized块必须包含在一个方法里。而lock可以在一个方法里lock（），在另一个方法里unlock（）。

Lock有几个方法：lock，unlock，tryLock（）立即返回，lock上就返回true，`tryLock（time,timeUnit）`在一定时间内尝试lock，没lock上就返回false。  
实现：`ReenterantLock`

### ReadWriteLock
原则：

* 读：没有写线程锁定它，并且没有写线程试图锁定他（还未获得锁），多个线程就能同时锁定他来读。
* 写：没有任何读操作或者写操作。

实现：`ReenterantReadWriteLock`
lock.readLock().lock();lock.readLock().unlock();lock.writeLock.lock();  
实际上它的内部实现是维持了2个锁实例，一个管理读权限，一个管理写权限。

### AtomicBoolean
Atomic这种类有CAS（compare and swap）功能，so，原子。

```
AtomicBoolean aBoolean = new AtomicBoolean();
boolean b = aBoolean.get();
aBoolean.set(true);
boolean oldValue = aBoolean.getAndSet(false);//得到原值，并set新值。
boolean wasNewvalueSet = aBoolean.compareAndSet(false,true);//如果原来里面的值是false，则设置为true。返回值返回设置成功与否。它是原子的，任何时候只有一个线程可以访问它。
```
### AtomicInteger
除了上面的那些方法外，他还有：

* addAndGet(i);//先增加一个数，然后返回它的值。
* getAndAdd(i);
* getAndIncrement();//得到值，并且加一
* getAndDecrement();
* ...

### AtomicReference
它有泛型和非泛型的2种。  
泛型的在获得值时可直接获得，非泛型要手动转换类型。  
有常用的3个：`get()，set()，compareAndSet()`。


