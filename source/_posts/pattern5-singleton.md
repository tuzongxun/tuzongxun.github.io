---
title: 《设计模式》学习笔记5——单例模式【高并发拓展】
date: 2017-11-23 15:43:58
categories: 设计模式 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 设计模式
---
## 定义
单例模式又称为单件模式，这个模式大概是设计模式中最好理解的了，我起初就打算从这里开始学，甚至还记过另一篇单例模式学习的笔记。
但是之后跟着《设计模式》这本书系统的学，就索性从第一页开始，而单例模式算是复习，也算是再深入的理解一次。
<!--more-->
之所以要这么做，是因为上一次写的没有给出更标准的定义，同时，当时只介绍了基础的懒汉式和饿汉式，对于并发时候的单例却没有涉及，所以这篇学习的重点应当在于高并发时如何保证我们的单例依旧是单例。
单例模式引用书中的定义如下：
>单例模式(Singleton Pattern)：确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种对象创建型模式

## 理解
由于之前写过一篇单例模式的记录，里边对基础饿汉式和懒汉式的介绍都还算详细，因此整个基础饿汉式和懒汉式的使用及理解这里就没有必要赘述，以下是那篇的链接：</br>
<https://tuzongxun.github.io/2017/09/24/pattern1_danli/></br>
<https://yq.aliyun.com/articles/210401?spm=5176.8091938.0.0.1wWDAm></br>
<http://blog.csdn.net/tuzongxun/article/details/78009213></br>
</br>

## 要点
饿汉式和懒汉式的不同，在于单例对象的创建时机，一个是直到第一次被用的时候才创建，一个是不管有没有人用，反正先创建了放这里再说。
这就映射了现实生活中的两类人，一类是有备无患，一种是临时抱佛脚。
既然他们都是单例模式，自然就有属于单例模式的共性，总结下来基本如下：
1. 构造器私有化，不允许自己之外的他人创建 
2. 在上一条的前提下，就必须自己创建自己的实例对象
3. 为了保证实例对象唯一，这个实例对象必须是静态的，属于类 
4. 外部不能用new获得这个对象，那么就必须提供一个外部能访问的静态方法，返回自己的实例对象 

## 拓展
开篇说到这一篇的重点应该在于高并发时如何保证单例对象还是单例，那么首先必须是高并发时会出现不再单例的问题，这个问题实际上只会出现在懒汉式的情况中。（注：这似乎也警示着人们，人还是勤快点好啊）
那么我们先来看一下饿汉式的写法：
<pre>
package patterntest.singletonpattern;
/**
 *单例模式——饿汉式
 *@author tzx
 *@date 2017年11月23日
 */
public class SingletonPattern1 {
	private static SingletonPattern1 singletonPattern1 = new SingletonPattern1();
	private SingletonPattern1() {
	}
	public static SingletonPattern1 getInstance() {
		return singletonPattern1;
	}
}
</pre>
在饿汉式中，一开始初始化类的时候就创建了类的实例对象，也就是说在被使用之前就已经创建，这时候即便是多个线程同时来取这个对象，也依旧还是这同一个对象，不会有任何问题。
然后再看一下懒汉式的写法：
<pre>
package patterntest.singletonpattern;
/**
 *单例模式——懒汉式
 *@author tzx
 *@date 2017年11月23日
 */
public class SingletonPattern2 {
	private static SingletonPattern2 singletonPattern2;
	private SingletonPattern2() {
	}
	public static SingletonPattern2 getInstance() {
		if (singletonPattern2 == null) {
			singletonPattern2 = new SingletonPattern2();
		}
		return singletonPattern2;
	}
}
</pre>
在懒汉式中，当有消费者需要获取当前类的对象时，会先判断该对象是否是null，如果是，才会创建一个对象。
那么我们知道在高并发的时候，线程可能在任何时候让出cpu，也就是说如果有两条线程，当第一条读到`singletonPattern2 == null`为true后，还没有来得及new一个对象，这时候让出了cpu，另一个线程接手了。
然后第二个线程再进行if判断的时候会发现依旧是true，然后他就new了一个对象出来，然后让出cpu。
结果第一个线程接手后接着之前的判断结果进行处理，就会再new一个对象出来，这时候我们的单例便不再是单例了。
为了演示这种情况，我们对懒汉模式稍作改造：
<pre>
package patterntest.singletonpattern;
/**
 *单例模式——懒汉式
 *@author tzx
 *@date 2017年11月23日
 */
public class SingletonPattern2 {
	private static SingletonPattern2 singletonPattern2;
	private SingletonPattern2() {
	}
	public static SingletonPattern2 getInstance() {
		if (singletonPattern2 == null) {
			try {
				Thread.yield();
			} catch (Exception e) {
				e.printStackTrace();
			}
			singletonPattern2 = new SingletonPattern2();
		}
		return singletonPattern2;
	}
}
</pre>
在上边的代码中，我们调用`Thread.yield()`手动在new一个对象之前使当前线程让出剩余cpu时间，这时候下一个线程就会执行，然后写一个测试多线程调用的类及方法：
<pre>
package patterntest.singletonpattern;
/**
 *单例模式测试
 *@author tzx
 *@date 2017年11月23日
 */
public class Consumer {
	public static void main(String[] args) {
		Thread thread1 = new Thread(new MyRunnable());
		thread1.start();
		Thread thread2 = new Thread(new MyRunnable());
		thread2.start();
	}
}
class MyRunnable implements Runnable {
	public void run() {
		SingletonPattern2 singletonPattern2 = SingletonPattern2.getInstance();
		System.out.println(singletonPattern2 + ":" + singletonPattern2.hashCode());
		SingletonPattern1 singletonPattern1 = SingletonPattern1.getInstance();
		System.out.println(singletonPattern1 + ":" + singletonPattern1.hashCode());
	}
}
</pre>

上述代码应该比较简单，启动两个线程，在线程的run方法里获取我们的单例对象，然后打印出hashcode。我们知道，同一个对象的hashcode应该是一样的，但是运行上边的代码之后结果如下：
<pre>
patterntest.singletonpattern.SingletonPattern2@51ab0bbe:1370164158
patterntest.singletonpattern.SingletonPattern2@4ab73ee:78345198
patterntest.singletonpattern.SingletonPattern1@5f6cfffb:1600978939
patterntest.singletonpattern.SingletonPattern1@5f6cfffb:1600978939
</pre>
很明显，两个Pattern2的hashcode值不一样，而两个Pattern1的hashcode是完全一样的，进一步证明了懒汉式的单例模式并不能保证在多线程、高并发时保证单例，那么这时候就需要涉及到线程同步的知识，可以使用同步锁synchronized锁住我们的getInstance方法，代码就可以改成这样：
<pre>
public synchronized static SingletonPattern2 getInstance() {
	if (singletonPattern2 == null) {
		try {
			Thread.yield();
		} catch (Exception e) {
			e.printStackTrace();
		}
		singletonPattern2 = new SingletonPattern2();
	}
	return singletonPattern2;
}
</pre>
这时候无论我们再运行多少次测试方法，可以保证Pattern2的所有对象的hashcode都是同一个，也就是保证了Pattern2对象是一个单例对象，就像下边这样：
<pre>
patterntest.singletonpattern.SingletonPattern2@4ab73ee:78345198
patterntest.singletonpattern.SingletonPattern2@4ab73ee:78345198
patterntest.singletonpattern.SingletonPattern1@a6c9d0b:174890251
patterntest.singletonpattern.SingletonPattern1@a6c9d0b:174890251
</pre>
上边的问题确实解决了多线程高并发时单例对象不单例的问题，然后由于锁定的是方法，所以必然在高并发时影响性能，所以我们需要进一步优化，从锁方法缩小到只锁住创建对象的那一段代码,也就是所谓的锁定代码块：
<pre>
// public synchronized static SingletonPattern2 getInstance() {
	public static SingletonPattern2 getInstance() {
		// if (singletonPattern2 == null) {
		// try {
		// Thread.yield();
		// } catch (Exception e) {
		// e.printStackTrace();
		// }
		// singletonPattern2 = new SingletonPattern2();
		// }
		if (singletonPattern2 == null) {
			try {
				Thread.yield();
			} catch (Exception e) {
				e.printStackTrace();
			}
			synchronized (SingletonPattern2.class) {
				singletonPattern2 = new SingletonPattern2();
			}
		}
		return singletonPattern2;
	}
</pre>
但是仔细看上边的代码，或者运行一下测试方法会发现，这种写法实际上并不能解决单例对象不单例的情况，此时的锁其实是个无意义的锁。所以我们需要把这个锁进一步优化，使用`双重检查锁定`机制，然后代码就应该是这样：
<pre>
public static SingletonPattern2 getInstance() {
	if (singletonPattern2 == null) {
		try {
			Thread.yield();
		} catch (Exception e) {
			e.printStackTrace();
		}
		synchronized (SingletonPattern2.class) {
			if (singletonPattern2 == null) {
			singletonPattern2 = new SingletonPattern2();
			}
		}
	}
	return singletonPattern2;
}
</pre>
这样一来，在调用这个getInstance方法的时候就不会被直接锁住，而是先进行一个判断后才锁定，提升了系统性能。
同时，由于锁里边又进行了判断，就可以进一步保证对象的唯一性。
然而，上边的结论其实是我想当然的结论，按书中所说，这样也并不能完全保证单例，还需要给对象增加一个修饰词`volatile`，然后整个类就应该是这个样子：
<pre>
package patterntest.singletonpattern;
/**
 *单例模式——懒汉式
 *@author tzx
 *@date 2017年11月23日
 */
public class SingletonPattern2 {
	private volatile static SingletonPattern2 singletonPattern2;
	private SingletonPattern2() {
	}
	public static SingletonPattern2 getInstance() {
		if (singletonPattern2 == null) {
			try {
				Thread.yield();
			} catch (Exception e) {
				e.printStackTrace();
			}
			synchronized (SingletonPattern2.class) {
				if (singletonPattern2 == null) {
				singletonPattern2 = new SingletonPattern2();
				}
			}
		}
		return singletonPattern2;
	}
}
</pre>
不过那种不加volatile修饰符而出现不单例的情况应该并不多见，因为为了验证这一情况，我写了一个for循环创建线程来测试，即便是循环数万次，也依旧没有出现不一致的情况。

demo源码可在github下载：<https://github.com/tuzongxun/mypattern>