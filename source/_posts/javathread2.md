---
title: java基础之多线程总结二（CAS和各种常用锁）
date: 2022-2-25 18:43:58
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [java,多线程]
---
## CAS
在java中，对很多常见的需要加锁的操作进行了封装，例如Atomic开头的一些类，这些类是线程安全的，但是内部却不是用synchronized加锁实现，而是CAS。
例如AtomicInteger的'incrementAndGet()'方法最终调用的实际是下边这个方法
```
@HotSpotIntrinsicCandidate
    public final native boolean compareAndSetInt(Object o, long offset,
                                                 int expected,
                                                 int x);
```
<!--more-->
这个本地方法在一个比较特殊的类中，即Unsafe，这个类可以直接进行内存管理等操作，且源码可以看出来这是一个典型的单例模式。 
CAS是compare and set的缩写，即比较并设值的意思，CAS之所以能保证线程安全，是因为这类操作是cpu原语支持的，执行过程中不能被打断。
CAS操作会有ABA问题，意思就是一个线程在操作的时候，另一个线程把数据修改了，然后又改了回去，单纯比较值似乎没有变化。
ABA问题对于基础数据类型的数据其实没有太大影响，如果不是基础类型，并且必须处理ABA问题，可以考虑增加版本号管理，在compare的时候连版本号一起比较。
就像AtomicInteger类，如果要处理ABA问题可以考虑使用AtomicStampedReference类。

Atomic这一类操作的CAS是无锁的，所以有的时候比synchronized的效率要高，而针对数据自增这种操作，jdk还自带了一些其他的类，例如LongAdder，LongAdder采用的也是CAS，但是是分段的，所以一些应用场景下可能性能更好，例如有这样一段测试代码：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/23
 */
public class LongAddrDemo {
	private static Long count1 = 0L;

	private static AtomicLong count2 = new AtomicLong(0);

	private static LongAdder count3 = new LongAdder();

	public static void main(String[] args) {
		Thread[] threads = new Thread[1000];

		Object o = new Object();
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(() -> {
				for (int j = 0; j < 100000; j++) {
					synchronized (o) {
						count1++;
					}
				}
			});
		}

		Long t1 = System.currentTimeMillis();
		for (int i = 0; i < threads.length; i++) {
			threads[i].start();
		}
		for (int i = 0; i < threads.length; i++) {
			try {
				threads[i].join();
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		Long t2 = System.currentTimeMillis();
		System.out.println("count1-synchronized:" + count1 + ":" + (t2 - t1));
		//############################################
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(() -> {
				for (int j = 0; j < 100000; j++) {
					count2.incrementAndGet();
				}
			});
		}

		Long t11 = System.currentTimeMillis();
		for (int i = 0; i < threads.length; i++) {
			threads[i].start();
		}
		for (int i = 0; i < threads.length; i++) {
			try {
				threads[i].join();
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		Long t12 = System.currentTimeMillis();
		System.out.println("count2-Atomic:"+count2 + ":" + (t12 - t11));

		//############################################
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(() -> {
				for (int j = 0; j < 100000; j++) {
					count3.increment();
				}
			});
		}

		Long t21 = System.currentTimeMillis();
		for (int i = 0; i < threads.length; i++) {
			threads[i].start();
		}
		for (int i = 0; i < threads.length; i++) {
			try {
				threads[i].join();
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		Long t22 = System.currentTimeMillis();
		System.out.println("count3-LongAdder:"+count3 + ":" + (t22 - t21));
	}
}
```

上述代码运行后结果如下：
```
count1-synchronized:100000000:6060
count2-Atomic:100000000:2149
count3-LongAdder:100000000:499
```

很明显，针对当前的测试，LongAdder的性能高于AtomicLong，AtomicLong的性能要高于synchronized。
但是需要注意的是，上边的结论只是针对于当前场景，并不是说什么时候都是这样，具体应用的时候还需要进行实际分析和测试确定。

## 各种锁
jdk中有很多使用CAS实现的锁，例如ReentrantLock，ReenTrantLock相比synchronized更加灵活，不过从实现上来说可能稍微麻烦些，其中有一点就是需要手动解锁，例如如下代码：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/23
 */
public class LockDemo {
	private static int count=0;

	public static void main(String[] args) {
		ReentrantLock lock=new ReentrantLock();
		Thread t1=new Thread(()->{
			lock.lock();
			for (int i = 0; i < 1000000; i++) {
				count++;
			}
			lock.unlock();
		});
		Thread t2=new Thread(()->{
			lock.lock();
			for (int i = 0; i < 1000000; i++) {
				count++;
			}
			lock.unlock();
		});
		try {
			t1.start();
			t2.start();
			t1.join();
			t2.join();
		}
		catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(count);
	}
}
```

上边的代码总能输出2000000，是线程安全的。需要注意的是，这里的unlock最后必须手动调用，上边的代码比较简单，所以没有过多处理，正常来说应该加入到finally代码块中以保证一定被调用。
之所以说ReentrantLock灵活，是因为它可以被打断，使用lockInterruptibly()，在创建lock对象的时候，还可以选择使用公平锁还是非公平锁，默认是非公平的，如果要公平，则可以在后边参数中传true，例如：
```
ReentrantLock lock=new ReentrantLock(true);
```

所谓的公平锁可以简单地理解为先来后到，而不是来了就直接·抢。

上边的代码，在main线程中使用了join方法等待两个线程结束，然后输出最终结果，实际上还可以用CountDownLatch替换这种写法，例如：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/24
 */
public class CountDownLatchDemo {
	private static int count=0;

	public static void main(String[] args) {
		ReentrantLock lock=new ReentrantLock(true);
		CountDownLatch cd=new CountDownLatch(2);
		new Thread(()->{
			lock.lock();
			for (int i = 0; i < 1000000; i++) {
				count++;
			}
			lock.unlock();
			cd.countDown();
		}).start();
		new Thread(()->{
			lock.lock();
			for (int i = 0; i < 1000000; i++) {
				count++;
			}
			lock.unlock();
			cd.countDown();
		}).start();
		try {
			cd.await();
		}
		catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(count);
	}
}
```

需要注意的是，CountDownLatch关键在于创建对象的时候定义的数量以及调用countDown方法，所以实际上可以在一个线程里多次调用countDown把数量减到零，这是需要写程序的时候自己控制的。

上边用到的ReentrantLock和synchronized比较类似，都是排他锁，这种锁在读多写少需要读写分离的场景中就有些不够用，相对来说效率也不够高，例如如下代码：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/24
 */
public class ReentrantLockDemo2 {
	private static int count=0;
	private static ReentrantLock reentrantLock=new ReentrantLock();
	public static void readCount(Lock lock){
		try{
			lock.lock();
			Thread.sleep(1000);
			System.out.println(new Date() +"----"+count);
		}catch (Exception e){
			e.printStackTrace();
		}finally {
			lock.unlock();
		}
	}

	private static void writeCount(Lock lock){
		try{
			lock.lock();
			Thread.sleep(1000);
			count++;
			System.out.println(new Date() +"----"+count);
		}catch (Exception e){
			e.printStackTrace();
		}finally {
			lock.unlock();
		}
	}

	public static void main(String[] args) {
		for (int i = 0; i < 5; i++) {
			new Thread(()->{
				readCount(reentrantLock);
			}).start();
		}
		for (int i = 0; i < 5; i++) {
			new Thread(()->{
				writeCount(reentrantLock);
			}).start();
		}
	}
}
```

上述代码运行结果如下：
```
Thu Feb 24 20:32:38 CST 2022----0
Thu Feb 24 20:32:39 CST 2022----0
Thu Feb 24 20:32:40 CST 2022----0
Thu Feb 24 20:32:41 CST 2022----0
Thu Feb 24 20:32:42 CST 2022----0
Thu Feb 24 20:32:43 CST 2022----1
Thu Feb 24 20:32:44 CST 2022----2
Thu Feb 24 20:32:45 CST 2022----3
Thu Feb 24 20:32:46 CST 2022----4
Thu Feb 24 20:32:47 CST 2022----5
```

可以看到这里不论是读还是写，都会独自占用一秒时间，总共花费10秒。
实际上，有一种读写分离的锁可以使的读锁共享，写锁排他，从而在适当的应用场景下提升效率，例如上边代码可以改成这样：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/24
 */
public class ReentrantLockDemo2 {
	private static int count=0;
	private static ReentrantReadWriteLock reentrantReadWriteLock=new ReentrantReadWriteLock();
	private static Lock readLock=reentrantReadWriteLock.readLock();
	private static Lock writeLock=reentrantReadWriteLock.writeLock();

	public static void readCount(Lock lock){
		try{
			lock.lock();
			Thread.sleep(1000);
			System.out.println(new Date() +"----"+count);
		}catch (Exception e){
			e.printStackTrace();
		}finally {
			lock.unlock();
		}
	}

	private static void writeCount(Lock lock){
		try{
			lock.lock();
			Thread.sleep(1000);
			count++;
			System.out.println(new Date() +"----"+count);
		}catch (Exception e){
			e.printStackTrace();
		}finally {
			lock.unlock();
		}
	}

	public static void main(String[] args) {
		for (int i = 0; i < 5; i++) {
			new Thread(()->{
				readCount(readLock);
			}).start();
		}
		for (int i = 0; i < 5; i++) {
			new Thread(()->{
				writeCount(writeLock);
			}).start();
		}
	}
}
```

上述代码运行结果如下：
```
Thu Feb 24 20:36:18 CST 2022----0
Thu Feb 24 20:36:18 CST 2022----0
Thu Feb 24 20:36:18 CST 2022----0
Thu Feb 24 20:36:18 CST 2022----0
Thu Feb 24 20:36:18 CST 2022----0
Thu Feb 24 20:36:19 CST 2022----1
Thu Feb 24 20:36:20 CST 2022----2
Thu Feb 24 20:36:21 CST 2022----3
Thu Feb 24 20:36:22 CST 2022----4
Thu Feb 24 20:36:23 CST 2022----5
```

可以看到这里实际上只花费了5秒，写的操作每个占用了1秒，所有读的操作都在同一秒内完成了。这里的代码和上边的相比，只是用了不同的锁。

jdk中还有一个类，可以实现指定数量的线程都到齐之后再开始运行，这个类就是CyclicBarrier，示例代码如下：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/24
 */
public class CyclicBarrierDemo {

	public static void main(String[] args) {
		CyclicBarrier cb=new CyclicBarrier(20,()->{
			System.out.println("数量凑齐，开始运行-------------------:"+new Date());
		});

		for (int i = 0; i <41 ; i++) {
			System.out.println("创建线程："+(i+1)+new Date());
			new Thread(()->{
				try {
					cb.await();
					System.out.println("开始运行："+new Date());
				}
				catch (InterruptedException e) {
					e.printStackTrace();
				}
				catch (BrokenBarrierException e) {
					e.printStackTrace();
				}
			}).start();
			try {
				Thread.sleep(1000);
			}
			catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```

上述代码输出结果如下：
```
创建线程：1Thu Feb 24 19:52:27 CST 2022
创建线程：2Thu Feb 24 19:52:28 CST 2022
创建线程：3Thu Feb 24 19:52:29 CST 2022
创建线程：4Thu Feb 24 19:52:30 CST 2022
创建线程：5Thu Feb 24 19:52:31 CST 2022
创建线程：6Thu Feb 24 19:52:32 CST 2022
创建线程：7Thu Feb 24 19:52:33 CST 2022
创建线程：8Thu Feb 24 19:52:34 CST 2022
创建线程：9Thu Feb 24 19:52:35 CST 2022
创建线程：10Thu Feb 24 19:52:36 CST 2022
创建线程：11Thu Feb 24 19:52:37 CST 2022
创建线程：12Thu Feb 24 19:52:38 CST 2022
创建线程：13Thu Feb 24 19:52:39 CST 2022
创建线程：14Thu Feb 24 19:52:40 CST 2022
创建线程：15Thu Feb 24 19:52:41 CST 2022
创建线程：16Thu Feb 24 19:52:42 CST 2022
创建线程：17Thu Feb 24 19:52:43 CST 2022
创建线程：18Thu Feb 24 19:52:44 CST 2022
创建线程：19Thu Feb 24 19:52:45 CST 2022
创建线程：20Thu Feb 24 19:52:46 CST 2022
数量凑齐，开始运行-------------------:Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
开始运行：Thu Feb 24 19:52:46 CST 2022
创建线程：21Thu Feb 24 19:52:47 CST 2022
创建线程：22Thu Feb 24 19:52:48 CST 2022
创建线程：23Thu Feb 24 19:52:49 CST 2022
创建线程：24Thu Feb 24 19:52:50 CST 2022
创建线程：25Thu Feb 24 19:52:51 CST 2022
创建线程：26Thu Feb 24 19:52:52 CST 2022
创建线程：27Thu Feb 24 19:52:53 CST 2022
创建线程：28Thu Feb 24 19:52:54 CST 2022
创建线程：29Thu Feb 24 19:52:55 CST 2022
创建线程：30Thu Feb 24 19:52:56 CST 2022
创建线程：31Thu Feb 24 19:52:57 CST 2022
创建线程：32Thu Feb 24 19:52:58 CST 2022
创建线程：33Thu Feb 24 19:52:59 CST 2022
创建线程：34Thu Feb 24 19:53:00 CST 2022
创建线程：35Thu Feb 24 19:53:01 CST 2022
创建线程：36Thu Feb 24 19:53:02 CST 2022
创建线程：37Thu Feb 24 19:53:03 CST 2022
创建线程：38Thu Feb 24 19:53:04 CST 2022
创建线程：39Thu Feb 24 19:53:05 CST 2022
创建线程：40Thu Feb 24 19:53:06 CST 2022
数量凑齐，开始运行-------------------:Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
开始运行：Thu Feb 24 19:53:06 CST 2022
创建线程：41Thu Feb 24 19:53:07 CST 2022
```

从上述结果可以看出，只有20个线程都准备好了之后才会开始运行，并且这个程序如果不手动关闭，则会一直处理运行等待状态。

jdk中还有一个线程相关的类，可以实现类似限流的操作，可以设定允许同时运行的线程数量，这个类就是Semaphore，示例代码如下：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/24
 */
public class SemaphoreDemo {

	public static void main(String[] args) {
		Semaphore sd=new Semaphore(2);
		for (int i = 0; i < 10; i++) {
			new Thread(()->{
				try {
					sd.acquire();
					System.out.println(new Date());
					Thread.sleep(1000);
				}
				catch (InterruptedException e) {
					e.printStackTrace();
				}finally {
					sd.release();
				}
			}).start();
		}
	}
}
```

这里创建Semaphore对象时传入了参数2，意思就是同时运行有2个线程运行，上述代码执行结果如下：
```
Thu Feb 24 20:48:57 CST 2022
Thu Feb 24 20:48:57 CST 2022
Thu Feb 24 20:48:58 CST 2022
Thu Feb 24 20:48:58 CST 2022
Thu Feb 24 20:48:59 CST 2022
Thu Feb 24 20:48:59 CST 2022
Thu Feb 24 20:49:00 CST 2022
Thu Feb 24 20:49:00 CST 2022
Thu Feb 24 20:49:01 CST 2022
Thu Feb 24 20:49:01 CST 2022
```

可以看到，每秒只有两个结果是一样的。如果把上边对象的2改成5，则运行结果如下：
```
Thu Feb 24 20:53:21 CST 2022
Thu Feb 24 20:53:21 CST 2022
Thu Feb 24 20:53:21 CST 2022
Thu Feb 24 20:53:21 CST 2022
Thu Feb 24 20:53:21 CST 2022
Thu Feb 24 20:53:22 CST 2022
Thu Feb 24 20:53:22 CST 2022
Thu Feb 24 20:53:22 CST 2022
Thu Feb 24 20:53:22 CST 2022
Thu Feb 24 20:53:22 CST 2022
```

即同一秒有5个线程在运行。
需要注意的是，这个类也是支持公平锁和非公平锁的，默认是非公平，如果要使用公平锁，则可以这样增加第二个参数，设置为true，例如：
```
Semaphore sd=new Semaphore(5,true);
```

除了上述加锁用法，还有一个也比较常用的锁相关的类LockSupport，用法示例如下：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/24
 */
public class LockSupportDemo {
	public static void main(String[] args) {
		Thread t = new Thread(() -> {
			for (int i = 0; i < 10; i++) {
				if (i == 3) {
					LockSupport.park();
				}
				System.out.println(i + ":" + new Date());
			}
		});
		t.start();
		try {
			Thread.sleep(3000);
			LockSupport.unpark(t);
		}
		catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

上述代码输出结果如下：
```
0:Thu Feb 24 23:58:50 CST 2022
1:Thu Feb 24 23:58:50 CST 2022
2:Thu Feb 24 23:58:50 CST 2022
3:Thu Feb 24 23:58:53 CST 2022
4:Thu Feb 24 23:58:53 CST 2022
5:Thu Feb 24 23:58:53 CST 2022
6:Thu Feb 24 23:58:53 CST 2022
7:Thu Feb 24 23:58:53 CST 2022
8:Thu Feb 24 23:58:53 CST 2022
9:Thu Feb 24 23:58:53 CST 2022
```

可以很明显的看到线程在park加锁后就进入了阻塞状态，在调用了unpark之后才开始继续运行。
线程锁相关的用法很多，各种锁都有自己的适用场景，没有绝对的哪个更好，甚至有的时候可能用哪个都差不多，这些都需要具体需要的时候分析以及测试。