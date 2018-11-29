---
title: 《设计模式》学习笔记7——观察者模式
date: 2017-12-5 15:43:58
categories: [[读书笔记,设计模式],[设计模式]] #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [设计模式,读书笔记]
---
## 定义
观察者模式是使用频率最高的设计模式之一，也是最容易理解的设计模式之一，这种模式在生活中随处可见。
<!--more-->
观察者模式引用书中的定义如下：
>观察者模式(Observer Pattern)：定义对象之间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式的别名包括发布-订阅（PublishSubscribe） 模式、模型-视图（Model/View） 模式、源-监听器（Source/Listener）模式或从属者（Dependents） 模式。观察者模式是一种对象行为型模式

## 理解
观察者模式在生活中随处可见，路口的红绿灯和行人及车辆的关系是一种观察者模式，演员和观众是一种观察者模式，甚至于一个人说话，其他人听，也是一种观察者模式。
说白了，就是一个对象发出了某种信号，其他对象会根据这个对象立即作出某些动作或反应（什么也不做也是一种动作和反应）。
根据理解，我写了一个新闻网站根据中央气象台气象信息更新各自网站主页天气信息的小例子。

### 被观察者
在这里，中央气象台充当被观察者，各大网站充当观察者，首先我们定义一个被观察者的接口（但从实现观察者模式来说可以不用接口，但为了更好的拓展，这里使用接口编程）。
<pre>
package patterntest.observerpattern.subject;
import java.util.ArrayList;
import java.util.List;
import patterntest.observerpattern.observer.WebSite;
/**
 *气象台接口
 *@author tzx
 *@date 2017年12月5日
 */
public interface MyObservatory {
	/*
	 *存放观察者集合
	 */
	List<WebSite> weblist = new ArrayList<WebSite>();
	/**
	 *增加新观察者
	 *@param webSite
	 */
	public void addWebSite(WebSite webSite);
	/**
	 *删除观察者
	 *@param webSite
	 */
	public void removeWebSite(WebSite webSite);
	/**
	 *发布天气信息
	 */
	public void pubMsg();
}
</pre>

在这个接口中有几个必要因素：
1. 保存所有观察者对象的集合
2. 新增观察者对象的方法
3. 删除观察者对象的方法
4. 发布信息的方法，也称为通知观察者的方法

然后写一个实现上述接口的实现类，把抽象方法实现：
<pre>
package patterntest.observerpattern.subject;
import patterntest.observerpattern.observer.WebSite;
public class WeatherObservatory implements MyObservatory {
	@Override
	public void addWebSite(WebSite webSite) {
		weblist.add(webSite);
	}
	@Override
	public void removeWebSite(WebSite webSite) {
		weblist.remove(webSite);
	}
	@Override
	public void pubMsg() {
		for (WebSite webSite : weblist) {
			webSite.update();
		}
	}
}
</pre>

add和remove方法应该不需要多说，就是向list集合加元素和删元素。重点在于pubMsg方法，也就是这里的发布天气信息的方法，这是观察者模式的核心所在。
在这个方法中，会遍历保存着所有观察者对象的集合，然后调用每个观察者对象的具体行为方法。当然了，这里边的逻辑可以自定义，未必就是通知所有观察者。

### 观察者
有了被观察者，接下来就是创建观察者，在上边的pubMsg方法中，我们调用了WebSite对象的update方法，但是现在还没有定义相应的对象。
观察者模式一般是一对多的关系，虽然说实际一对一也是没问题的，但是似乎就有些大材小用了。所以我们这里的观察者就使用多对象，这些对象都是观察者，所以有一个共同的父类：观察者接口。
<pre>
package patterntest.observerpattern.observer;
/**
 *观察者：网站接口
 *@author tzx
 *@date 2017年12月5日
 */
public interface WebSite {
	/**
	 *根据观察到的信息采取行动
	 */
	public void update();
}
</pre>

可以看到观察者接口的定义相对于被观察者就简单多了，只有一个update方法，也就是观察到被观察对象的状态变化后采取的行动。
观察者是一个抽象的说法，就需要有更具体的观察者，相对于这里的网站接口，我就给他两个观察者子类：新浪网和腾讯网。
<pre>
package patterntest.observerpattern.observer;
public class Sina implements WebSite {
	@Override
	public void update() {
		System.out.println("新浪天气更新");
	}
}
</pre>

<pre>
package patterntest.observerpattern.observer;
public class Tencent implements WebSite {
	@Override
	public void update() {
		System.out.println("腾讯天气更新");
	}
}
</pre>

上边两个观察者做的事太过于简单，所以就不多说了。接下来就模拟新浪网、腾讯网相继开始观察中央气象台的天气信息，然后中央气象台发布新的气象并通知两大网站。
<pre>
import patterntest.observerpattern.observer.Sina;
import patterntest.observerpattern.observer.Tencent;
import patterntest.observerpattern.observer.WebSite;
import patterntest.observerpattern.subject.MyObservatory;
import patterntest.observerpattern.subject.WeatherObservatory;
import patterntest.observerpattern.subject.WindObservatory;
/**
 *观察者模式测试
 *@author tzx
 *@date 2017年12月5日
 */
public class ObserverTest {
	public static void main(String[] args) {
		// 建立中央气象台
		MyObservatory observatory = new WeatherObservatory();
		// 创建新浪网
		WebSite sina = new Sina();
		// 新浪网开始观察中央气象台信息公告
		observatory.addWebSite(sina);
		// 创建腾讯网
		WebSite tencent = new Tencent();
		// 腾讯网开始观察中央气象台信息公告
		observatory.addWebSite(tencent);
		// 中央气象台发布新的天气信息
		int i = 0;
		do {
		observatory.pubMsg();
			i++;
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		} while (i < 10);
	}
}
</pre>

上边的测试方法中，先创建了一个中央气象台对象，然后创建新浪网和腾讯网对象并加入到气象观察队列。
之后循环10次调用气象台信息发布方法，新浪网对象和腾讯网对象就都会执行自己的update方法。
如果以后有新的观察者，例如搜狐网也要根据中央气象台的信息更新网站天气，那么只需要new一个搜狐网对象，然后调用addWebSite方法就行了。

## 要点
根据上边的实例和分析，观察者模式包含如下一些要点：
1. 需要一个被观察者，也称作观察目标
2. 观察目标中需要有一个保存所有观察者对象的集合
3. 观察目标中需要有增加观察者对象的方法
4. 观察目标中需要有删除观察者对象的方法
5. 观察目标中需要有发布信息，也称作通知观察者对象的方法
6. 观察者需要有一个提供给被观察者的方法

由上边总结的一些要点，应该很容易就写出一个观察者模式了，但是在观察者模式中还有一些需要特别说明的地方：
1. java提供了对观察者模式的支持，java.util包中有关于观察者模式的两个接口Observer和Observable。所以要写一个观察目标的类，可以实现Observable接口；要写一个观察者的类，可以实现Observer接口。
2. 观察者模式的实现逻辑上，可能与现实中的某些理解稍有差异。就如上边气象台的例子，原则上来说，气象台并不知道有哪些网站在观察气象信息，网站更新天气信息的主动权在网站手上。但是在代码实现观察者模式的时候，却是由观察目标来通知观察者的，观察目标那里需要保存所有观察者对象。

## 总结
观察者模式不论是理解还是具体实现都比较简单，可能也是因为简单，所以生活中随处可见，也可能是因为生活中随处可见，所以理解起来才显得简单。
观察者模式中保存了所有观察者对象，当使用抽象编程的时候，只需要维护一个观察者父类的集合，这样以后有了新的观察者，也不用改变观察目标，实现了一定程度的松耦合。
但是从上边示例中就可以看到，通知观察者的方法会遍历观察者对象集合，然后依次调用每个观察者对象的相应方法。所以如果这个观察者集合非常大，观察者非常多，或者相应的方法并不能很快结束。则必然影响整体的效率，也会出现数据延迟的问题。
同时，虽然可以通过参数传递的方式使观察者知道观察目标的状态，却无法知道观察目标的具体状态具体是如何变化的。


demo源码可在github下载：<https://github.com/tuzongxun/mypattern>