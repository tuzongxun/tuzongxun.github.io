---
title: java基础之多线程总结三（AQS、ThreadLocal和线程池）
date: 2022-2-27 18:43:58
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [java,多线程]
---
## AQS
多线程里很多新型锁实现的关键是AQS，AQS指的是AbstractQueuedSynchronizer这个类，整个锁实现过程的关键是CAS操作加volatile。
拿ReentrantLock非公平锁的lock和unlock举例，首先lock的源码中调用过程如下：
```
ReentrantLock.lock()-->ReentrantLock.Sync.lock()-->ReentrantLock.NonfairSync.lock()
```
<!--more-->
最后这个方法的实现逻辑是这样的：
```
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

很显然，这里首先使用CAS操作尝试直接加锁，如果没有成功就调用acquire方法获取锁，这里不论是CAS方法还是acquire方法都在AbstractQueuedSynchronizer类中实现，acquire方法内容如下：
```
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```

这里边首先尝试获取锁，这个方法实际调用的是NonfairSync.tryAcquire()，而这个方法的调用最终实现如下：
```
final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
 }
 ```
 
上边代码中也有CAS操作，同时有两个重要的点，即getState()和setState(nextc)，这里边很多操作实际都是围绕着这个进行。
如果点进去两个方法中，会发现实际都是操作了AbstractQueuedSynchronizer中的state属性，定义如下：
```
private volatile int state;
```
到这里，应该就比较容易理解为什么说AQS的底层是CAS加上volatile了。

## ThreadLocal
多个线程之间使用某一个对象数据，每个线程都会保留一份数据副本，一个线程修改之后会刷新回那个对象（不加volatile的情况下刷新回去的时机不确定），例如下边这样的代码：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/27
 */
public class ThreadLocalDemo {
	private static User user;
	public static void main(String[] args) {
		new Thread(()->{
			try {
				Thread.sleep(2000);
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(user.name);
		}).start();
		new Thread(()->{
			try {
				Thread.sleep(1000);
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
			user=new User();
			user.name="tzx";
		}).start();
	}
}
class User{
	String name;
}
```

这个代码执行之后，第一个线程会输出第二个线程中的数据，但是有的时候可能需要线程之前的某些数据就是独立的，这时候就可以使用ThreadLocal，那么上边的代码可以改成这样：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/27
 */
public class ThreadLocalDemo2 {
	private static ThreadLocal<User2> threadLocal=new ThreadLocal<>();
	public static void main(String[] args) {
		new Thread(()->{
			try {
				Thread.sleep(2000);
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(threadLocal.get());
		}).start();

		new Thread(()->{
			try {
				Thread.sleep(1000);
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
			User2 user=new User2();
			user.name="tzx";
			threadLocal.set(user);
		}).start();
	}
}
class User2{
	String name;
}
```

这个代码在执行之后，第一个线程输出结果就是null，而不会拿到第二个线程设置的值。
ThreadLocal的set方法源码如下：
```
public void set(T value) {
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null)
		map.set(this, value);
	else
		createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}
```

可以看到，这里set的数据实际是保存在一个map中，而这个map是当前线程对象里边的map，也就是说，首先从map容器来说，不同线程就已经进行了线程隔离。
ThreadLocal不用的时候需要remove，这部分涉及到jvm垃圾回收弱引用问题。

## 线程池
起初，创建和运行线程就是用Thread和Runnable，是在业务代码中直接创建和使用，这种方式在并发高的时候就需要频繁的创建和销毁线程，由于线程的创建和销毁都需要消耗较多的资源，因此就出现了线程池技术，实际简单点理解，线程池就是为了线程的复用，从而减少刚并发情况下的资源消耗。
线程池最上层接口是Executor，最基础的创建线程池的方式是使用Executors工具类，例如：
```
ExecutorService executorService= Executors.newFixedThreadPool(2);
executorService.execute(()->{
	int num=0;
	for (int i = 0; i < 10000; i++) {
		num++;
	}
	System.out.println(num);
});
```

这种使用线程池的方式是可以运行的，但是一般却不建议这样使用，原因是这种默认的方式，里边默认等待队列的最大长度太大了，容易造成内存问题，例如上边的这个创建固定线程池的方法源码如下：
```
public static ExecutorService newFixedThreadPool(int nThreads) {
	return new ThreadPoolExecutor(nThreads, nThreads,
								  0L, TimeUnit.MILLISECONDS,
								  new LinkedBlockingQueue<Runnable>());
}

public LinkedBlockingQueue() {
	this(Integer.MAX_VALUE);
}
```

可以看到这里默认的LinkedBlockingQueue长度就是Integer.MAX_VALUE。
因此，正确的使用线程池应该是使用Threadpoolexecutor，例如：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/27
 */
public class ThreadPoolDemo2 {
	public static void main(String[] args) {
		ThreadPoolExecutor pool=new ThreadPoolExecutor(2, 4, 60, TimeUnit.SECONDS, new LinkedBlockingDeque<Runnable>());
		pool.execute(()->{
			System.out.println("hello");
		});
	}
}
```

上边的代码创建线程的时候只给了5个参数，但是点进去这个ThreadPoolExecutor构造方法会看到里边的代码是这样的：
```
public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue) {
	this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
		 Executors.defaultThreadFactory(), defaultHandler);
}
```

可以看到，上边调用的5个参数的方法内部实际调用了一个7个参数的方法，只是后两个使用了默认值。
ThreadPoolExecutor实际上也是继承自ExecutorService，只不过是直接继承自AbstractExecutorService，而AbstractExecutorService实现了ExecutorService。
上边说到了创建线程池的7个参数比较重要，代表的意思如下：
**int corePoolSize**：核心线程数，可以理解为默认创建之后就不会被回收的线程数。
**int maximumPoolSize**：最大执行线程数，理解为线程池中能够存在的最大线程数，除了核心线程数之外的线程空闲时间到达最大空闲等待时间后会被销毁掉。
**long keepAliveTime**：空闲等待时间，决定非核心线程空闲存活时间。
**TimeUnit unit**：空闲等待时间单位，上一个参数的单位。
**BlockingQueue<Runnable> workQueue**：等待任务队列数，可以理解为到达最大线程数之后再进来等待的任务的最大数量。
**ThreadFactory threadFactory**：线程工厂，也就是怎么创建及管理线程生命周期的相关逻辑。
**RejectedExecutionHandler handler**：拒绝策略，即达到最大等待任务队列之后，再有新任务进来时候的处理逻辑。
为了理解上边几个参数，对上边main方法里的逻辑进行修改，如下：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/27
 */
public class ThreadPoolDemo2 {
	public static void main(String[] args) {
		ThreadPoolExecutor pool =
			new ThreadPoolExecutor(2, 4, 60, TimeUnit.SECONDS, new LinkedBlockingDeque<Runnable>(5));
		for (int i = 0; i <8; i++) {
			pool.execute(() -> {
				try {
					Thread.sleep(3000);
				}
				catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("hello");
			});
			System.out.println(
				"corePoolSize:" + pool.getCorePoolSize() + ",maxPoolSize:" + pool.getMaximumPoolSize() + ",poolSize:" + pool
					.getPoolSize() + ",queueSize:" + pool.getQueue().size());
		}

		pool.shutdown();
	}
}
```

这里创建线程池的时候，为任务等待队列指定了长度5，然后在后边使用for循环模拟8个任务请求，同时，任务在执行的时候均阻塞3秒，然后打印线程池的一些数据，运行结果如下：
```
corePoolSize:2,maxPoolSize:4,poolSize:1,queueSize:0
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:0
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:1
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:2
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:3
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:4
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:5
corePoolSize:2,maxPoolSize:4,poolSize:3,queueSize:5
hello
hello
hello
hello
hello
hello
hello
hello

Process finished with exit code 0
```

从输出结果可以看到最终8个任务都输出了，同时可以看到，核心线程是不管任务等待队列是否有任务的，只有核心线程数够了之后才会把任务往等待队列放，而非核心线程则是当任务等待队列满了之后才会开始创建。
因此从结果来看，虽然最大线程数是设置的4，但是实际上只创建了3个线程。
按上边解释的意思来看，我这个代码中应该最多只允许同时有9个任务，即4个运行中的加上5个等待队列里的，那么如果把上边程序再做一点改动，就可以验证这个问题，可以把for循环的次数改为10，如下：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/27
 */
public class ThreadPoolDemo2 {
	public static void main(String[] args) {
		ThreadPoolExecutor pool =
			new ThreadPoolExecutor(2, 4, 60, TimeUnit.SECONDS, new LinkedBlockingDeque<Runnable>(5));
		for (int i = 0; i <10; i++) {
			pool.execute(() -> {
				try {
					Thread.sleep(3000);
				}
				catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("hello");
			});
			System.out.println(
				"corePoolSize:" + pool.getCorePoolSize() + ",maxPoolSize:" + pool.getMaximumPoolSize() + ",poolSize:" + pool
					.getPoolSize() + ",queueSize:" + pool.getQueue().size());
		}

		pool.shutdown();
	}
}
```

上述代码运行结果如下：
```
corePoolSize:2,maxPoolSize:4,poolSize:1,queueSize:0
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:0
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:1
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:2
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:3
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:4
corePoolSize:2,maxPoolSize:4,poolSize:2,queueSize:5
corePoolSize:2,maxPoolSize:4,poolSize:3,queueSize:5
corePoolSize:2,maxPoolSize:4,poolSize:4,queueSize:5
Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task cn.tzxcode.demo.java.d03_thread.ThreadPoolDemo2$$Lambda$1/2046562095@6e2c634b rejected from java.util.concurrent.ThreadPoolExecutor@37a71e93[Running, pool size = 4, active threads = 4, queued tasks = 5, completed tasks = 0]
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2047)
	at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:823)
	at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1369)
	at cn.tzxcode.demo.java.d03_thread.ThreadPoolDemo2.main(ThreadPoolDemo2.java:17)
hello
hello
hello
hello
hello
hello
hello
hello
hello
```

可以看到，这里虽然模拟了10个任务，但是实际上只有9个成功执行了，第10个直接抛出了异常，被拒绝执行。
由于这里的异常没有进行处理，所以最终的shutdown方法也没有成功调用，因此这个程序运行后会一直阻塞，无法结束。
线程池的核心实际上就是这7个参数，使用不同的数值，不同的队列，以及不同的线程工厂和拒绝策略，就可以应对各种不同的需求场景。
在这7个参数的基础上，根据一些常用的应用场景，jdk中也内置了其他一些常用的线程池，例如singlethreadpool、newcachedthreadpool、fixedthreadpool、scheduledpool，实际上就是上边几个参数的一些特定情况。
在线程池使用过程中还有一个经常相关的内容就是Callable接口，这个和Runnable很相似，增加了返回值，返回值类型的Future以及相关的FutureTask、CompletableFuture这些，能够使得程序更加灵活。
除此之外，还有一个ForkJoinPool，不太常用，但是有些大任务的应用场景下效率特别高，主要在于他的实现是把大任务切分成很多小任务，最后还可以进行结果的汇总。