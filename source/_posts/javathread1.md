---
title: java基础之多线程总结一（创建、状态、synchronized和volatile）
date: 2022-2-23 18:43:58
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [java,多线程]
---
## 线程基本创建和启动方式
不考虑线程池的情况下，创建和启动线程的基本方式有如下几种
1.直接new Thread类或者子类,
2.实现runnable接口然后传递给Thread，这种方式更加灵活
3.使用lambda表达式，实际上是实现runnable另一种写法
<!--more-->
示例代码：
```
public class CreateThread {
	public static void main(String[] args) {
		Thread t1=new Thread();
		t1.start();

		Thread t2=new MyThread();
		t2.start();

		Thread t3=new Thread(new MyRunnable());
		t3.start();

		Thread t4=new Thread(()-> System.out.println("lambda runnable start"));
		t4.start();
	}
}
class MyThread extends Thread{
	@Override
	public void run() {
		super.run();
		System.out.println("MyThread start");
	}
}

class MyRunnable implements Runnable{
	@Override
	public void run() {
		System.out.println("MyRunnable start");
	}
}
```

## 线程状态
一个线程从创建到运行再到结束，中间有很多种状态，这些状态之间经过不同的操作可以相互转换，总的来说可能会有如下这几种：
![在这里插入图片描述](https://img-blog.csdnimg.cn/512eafc0f5804e328d9b0f69bb42cbea.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

sleep会睡眠、暂停，把执行权让给其他线程，当时间结束后重新回到等待队列。
yield进入等待队列，有可能马上再次被执行，返回到就绪状态。
join等待另一个线程结束再执行当前线程，或者说把另一个线程加入到当前线程中，相当于变成了同步操作。
wait放出执行权等待唤醒，同时释放锁，notify和notifyAll唤醒其他等待的线程，但是不释放锁。

## synchronized
synchronized同步锁，锁一个对象，同时保证原子性和可见性。
这个锁是可重入的，可重入指的是同一个线程再次调用的时候可以直接拿到锁，比较典型的例子是父类和子类具有继承关系的方法都加了同步锁，然后子类中使用super调用父类的方法，如果不可重入则会发生死锁。
jdk1.5之前，synchronized锁是重要级的实现，每次加锁都需要找操作系统申请系统锁，因此性能相对较低。jdk1.5之后这个锁进行了升级，会先从无锁状态升级为偏向锁，如果有线程争用则升级为自旋锁，如果自旋结束还是没有拿到锁，则会升级为重量级系统锁。
偏向锁实际是无锁的，会记录线程id信息。
需要注意的是，目前jvm的实现是锁升级只能升不能降，同时，如果出现异常，默认锁被释放。
自旋锁相对系统锁更快，但是也更消耗cpu性能，所以如果是自己的代码加锁，则加锁代码执行时间短、线程数少的时候才用自旋锁，否则用系统锁。
另一个需要注意的是，锁的对象尽量不用String（常量）、Integer、Double等类型，因为它们的底层实现会导致可能预期不同的对象实际却是同一个对象，从而出现原本应该无关的线程结果会锁上同一个锁对象。
```
synchronized经常用来锁方法或者代码块，如下边这样：
public static synchronized void addNum(){
	num++;
}
public static void addNum1(){
	Object o=new Object();
	synchronized (o){
		num++;
	}
}
```

上边两个方法效果其实是一样的，只不过第一个锁的对象是.class对象，而后一个是自定义的锁对象，第二种写法相对来说可能更灵活，当一个方法里只是一部分逻辑需要上锁的时候就很适用。
## volatile
volatile有两个重要作用，一个是保证线程可见性，另一个是禁止指令重排序。
保证线程可见性借助的是cpu的缓存一致性协议。
对于保证线程可见性，有如下这样一段代码：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/22
 */
public class VolatileDemo {
	private int num=0;
	public static void main(String[] args) {
		VolatileDemo volatileDemo=new VolatileDemo();

		new Thread(()->{
			while(true){
				if(volatileDemo.num==20000){
					System.out.println(volatileDemo.num);
					break;
				}
			}
		}).start();

		new Thread(()->{
			volatileDemo.num=20000;
		}).start();
	}
}
```

这段代码很简单，期望的情况就是运行后输出一个结果，然后整个程序会停止。
但是实际运行时并不总是会停止，多次运行就会发现有的时候会一直运行着不会停止，实际上就是第一个线程一直读不到期望的结果。
导致这样问题的原因就是线程可见性问题，两个不同的线程在运行时实际是各自保留了一份数据的副本，彼此是不可见的，修改数据后会刷新到源数据里，但是什么时候刷新是不一定的，因此也就使得修改后有的时候可以被另一个线程读到，有时候不能被另一个线程读到。
但是当我们给这个变量加上volatile之后，不管运行多少次，这个程序都可以正确停止。即如下这样：
```
private volatile int num=0;
```

之所以这样，就是因为volatile修饰后，会使得线程之间的数据可见。
但是需要注意的是，volatile能保证线程可见性，却不能保证原子性，例如上述代码如果改成这样：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/22
 */
public class VolatileDemo {
	private volatile int num=0;
	public static void main(String[] args) {
		VolatileDemo volatileDemo=new VolatileDemo();
		CountDownLatch cd=new CountDownLatch(2);
		new Thread(()->{
			for(int i=0;i<100000;i++){
				volatileDemo.num++;
			}
			cd.countDown();
		}).start();

		new Thread(()->{
			for(int i=0;i<100000;i++){
				volatileDemo.num++;
			}
			cd.countDown();
		}).start();

		try {
			cd.await();
			System.out.println(volatileDemo.num);
		}
		catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

上述代码运行后，如果能保证原子性，则应该每次都输出200000，但实际上每次结果都可能不一样。
禁止指令重排序，比较典型的应用是单例模式双重检查，代码示例如下：
```
/**
 * @Author tuzongxun
 * @Date 2022/2/22
 */
public class VolatileDemo2 {
	private static volatile VolatileDemo2 volatileDemo2;
	private VolatileDemo2(){
	}
	public static VolatileDemo2 getInstance(){
		if(volatileDemo2==null){
			synchronized (VolatileDemo2.class){
				if(volatileDemo2 ==null) {
					volatileDemo2 = new VolatileDemo2();
				}
			}
		}
		return volatileDemo2;
	}
}
```

这里volatile的主要作用其实就是防止指令重排序，这里指令重排序指的是，new对象的过程实际会有申请内存、初始化、赋值等若干个操作，在多线程的情况下，如果不加volatile，则可能使这几个操作被打断，从而导致指令重排序，例如上边这个代码使用idea插件查看指令结果如下：
```
0 getstatic #2 <cn/tzxcode/demo/java/d03_thread/VolatileDemo2.volatileDemo2 : Lcn/tzxcode/demo/java/d03_thread/VolatileDemo2;>
3 ifnonnull 37 (+34)
6 ldc #3 <cn/tzxcode/demo/java/d03_thread/VolatileDemo2>
8 dup
9 astore_0
10 monitorenter
11 getstatic #2 <cn/tzxcode/demo/java/d03_thread/VolatileDemo2.volatileDemo2 : Lcn/tzxcode/demo/java/d03_thread/VolatileDemo2;>
14 ifnonnull 27 (+13)
17 new #3 <cn/tzxcode/demo/java/d03_thread/VolatileDemo2>
20 dup
21 invokespecial #4 <cn/tzxcode/demo/java/d03_thread/VolatileDemo2.<init> : ()V>
24 putstatic #2 <cn/tzxcode/demo/java/d03_thread/VolatileDemo2.volatileDemo2 : Lcn/tzxcode/demo/java/d03_thread/VolatileDemo2;>
27 aload_0
28 monitorexit
29 goto 37 (+8)
32 astore_1
33 aload_0
34 monitorexit
35 aload_1
36 athrow
37 getstatic #2 <cn/tzxcode/demo/java/d03_thread/VolatileDemo2.volatileDemo2 : Lcn/tzxcode/demo/java/d03_thread/VolatileDemo2;>
40 areturn
```