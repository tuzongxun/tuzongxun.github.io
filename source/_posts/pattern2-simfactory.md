---
title: 《设计模式》学习笔记2——简单工厂模式
date: 2017-11-20 18:43:58
categories: [[读书笔记,设计模式],[设计模式]] #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [设计模式,读书笔记]
---
## 定义
简单工厂模式并不属于GoF（Gang of Four四人组）23中设计模式，有些地方的解释说因为简单工厂模式太简单，所以23中设计模式就没有单独列出。
但是简单工厂模式在实际的应用中却很常用，因此在刘伟老师的《设计模式》一书中就还是列了出来。
简单工厂模式引用书中的定义如下：
<!--more-->
>简单工厂模式(Simple Factory Pattern)：定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。因为在简单工厂模式中用于创建实例的方法是静态(static)方法，因此简单工厂模式又被称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式

## 理解
对于这个模式，我觉得说静态工厂模式比说简单工厂模式更能让人记住。静态工厂模式，通过静态两个字就可以知道需要有一个静态的工厂方法，然后只要再理解一下何谓工厂，基本上就记住了这个模式。
那么何谓工厂呢，众所周知，生产产品的地方就是工厂。
对于工厂而言，需要有能正常运行的生产线，这个生产线可能是全自动化的，也可能是人工操作的，除此之外，还需要有被生产的产品。
根据上边的思路，我们便可以转换为代码，首先写出我们的产品类：
<pre>
package patterntest.simplefactorypattern;
/**
 *产品类
 *@author tzx
 */
public class MyProduct {
	/**
	 *产品名称
	 */
	private String productName;
	/**
	 *产品规格
	 */
	private String productSize;
	public String getProductName() {
		return productName;
	}
	public void setProductName(String productName) {
		this.productName = productName;
	}
	public String getProductSize() {
		return productSize;
	}
	public void setProductSize(String productSize) {
		this.productSize = productSize;
	}
	@Override
	public String toString() {
		return "MyProduct [productName=" + productName + ", productSize=" + productSize + "]";
	}
}
</pre>
有了这样一个产品类，便可以创建该类的具体产品实例，学习java的必然知道最基本的创建对象实例的方式就是使用new关键字，比如`new MyProduct()`。就像下边的代码：
<pre>
package patterntest.simplefactorypattern;
/**
 *消费者
 *@author tzx
 */
public class Consumer {
	public static void main(String[] args) {
		System.out.println("我需要一个产品，于是我愉快的制造了一个");
		MyProduct product = new MyProduct();
		System.out.println("我制造的产品是这样的：" + product.getProductSize());
	}
}
</pre>
这种方式其实相当于现实生活中某个人需要一个什么东西，然后自己制造一个出来。
这种情况现实生活中必然是存在的，但是也必然有一定的限制，不可能每个人都能制造每种东西，所以还有很多东西个人制造不出来，或者说个人制造的成本太高，因此便有了专门制造某类东西的工厂，也就是我们的工厂类：
<pre>
package patterntest.simplefactorypattern;
/**
 *简单工厂
 *@author tzx
 */
public class MySimpleFactory {
	/**
	 *外部获取产品的工厂方法
	 */
	public MyProduct getProduct() {
		return new MyProduct();
	}
}
</pre>
那么这个时候，我们需要用某个东西的人，简称为消费者，就不需要自己再制造这类东西，也不需要如何制造这类东西，可以更加方便的使用：
<pre>
package patterntest.simplefactorypattern;
/**
 *消费者
 *@author tzx
 */
public class Consumer {
	public static void main(String[] args) {
		System.out.println("我需要一个产品，于是我向工厂买了一个");
		MyProduct product = new MySimpleFactory().getProduct();
		System.out.println("我买的产品是这样的：" + product.getProductSize());
	}
}
</pre>
这样一来，我们需要的东西只需要调用工厂的购买方法，或者说工厂提供给外边的获取产品的方法，然后就能获取到想要的产品了。
然而，有一个很不愉快的问题出现了，上边的代码中，产品确实不需要自己创建，不需要自己new了，但是工厂却是被我们new出来的。
这样问题就大了，相当于我仅仅需要某个产品，然后为了这个产品，我必须自己创建了一个工厂，而且每个需要这个产品的人都要创建一个该产品的工厂，这显然是不合理也不可能的。
我们需要的仅仅是这样而已：
<pre>
MyProduct product = MySimpleFactory.getProduct();
</pre>
那么工厂方法也就需要改成静态的，使我们只要用类名就可以调用获取产品的方法：
<pre>
/**
 * 外部获取产品的工厂方法
 */
public static MyProduct getProduct() {
	return new MyProduct();
}
</pre>
好了，现在对于某个产品，工厂就合理的提供了一个给外边获取产品的方法，而外边的消费者也能通过一声`MySimpleFactory.getProduct()`的呼唤愉快的获取到需要的产品了。
然而，如果后边这个工厂扩大了，开始运营的产品不再单一了，例如牙膏，一开始只是成年人用的牙膏，后边又增加了儿童牙膏，那么很显然这时候我们工厂提供的获取产品的无参方法就难以继续担当大任了。
因为只通过一声`我要买牙膏`的呼唤，工厂根本无法知道消费者需要的是哪种，所以我们的工厂方法需要有参数，并且根据不同的参数给予不同的产品：
<pre>
public static MyProduct getProduct(int type) {
	return new MyProduct();
}
</pre>
现在，基本解决了上边不知道消费者具体要什么的问题，但是，工厂内部却犯难了，因为根据原本产品制造说明说（产品类的定义），工厂能制造出来的产品只有一种啊。
所以，原本的产品也需要升级，也就是本身的默认无参构造器不能满足现有的需求了，我们可能需要增加有参数的构造方法，需要升级原本的产品制造说明书，并通过参数来确定具体的产品。
<pre>
public MyProduct(int type) {
	if (0 == type) {
		// 提供成年人牙膏
	} else if (1 == type) {
		// 提供儿童牙膏
	}
}
</pre>
那么以上的一些问题也暂时解决了，不管是需要儿童牙膏还是普通牙膏，工厂都能正确的提供，消费者也都能正确的获取。
<pre>
public static MyProduct getProduct(int type) {
	return new MyProduct(type);
}
</pre>
只是，如果后边工厂又扩大了，还要加入环保牙膏、特效美白牙膏呢？那么我们就需要不断的修改原本的产品构造方法，就如一本产品制造说明书要继续加厚。
很显然，这将导致那个有参数的构造方法越来越臃肿，如果每种产品的制造方法都异常复杂的话，我们的一本产品制造说明书也将可能达到需要人抬的地步。
所以，这样不是办法，我们需要把各自具体产品的制造说明书分开，也就是我们的产品类需要由一个变成多个：
<pre>
/**
 *普通牙膏
 *@author tzx
 */
public class MyProduct1 {
	/**
	 * 产品名称
	 */
	private String productName;
	/**
	 * 产品规格
	 */
	private String productSize;
	public MyProduct1() {
		addIngredient();
	}
	/**
	 * 添加必要成分
	 */
	public void addIngredient() {
	}
	public String getProductName() {
		return productName;
	}
	public void setProductName(String productName) {
		this.productName = productName;
	}
	public String getProductSize() {
		return productSize;
	}
	public void setProductSize(String productSize) {
		this.productSize = productSize;
	}
	@Override
	public String toString() {
		return "MyProduct [productName=" + productName + ", productSize=" + productSize + "]";
	}
}
</pre>
<pre>
/**
 *儿童牙膏
 *@author tzx
 */
public class MyProduct2 {
	/**
	 *产品名称
	 */
	private String productName;
	/**
	 *产品规格
	 */
	private String productSize;
	public MyProduct2() {
		addIngredient();
		delead();
	}
	/**
	 *去铅
	 */
	public void delead() {

	}
	/**
	 *添加必要成分
	 */
	public void addIngredient() {
	}
	public String getProductName() {
		return productName;
	}
	public void setProductName(String productName) {
		this.productName = productName;
	}
	public String getProductSize() {
		return productSize;
	}
	public void setProductSize(String productSize) {
		this.productSize = productSize;
	}
	@Override
	public String toString() {
		return "MyProduct [productName=" + productName + ", productSize=" + productSize + "]";
	}
}
</pre>
明眼人应该一眼就看出了上述代码的问题，这两个类几乎一模一样，唯独不同的就是儿童牙膏增加了一个去铅的方法。
很显然，这样的代码是严重重复，不被看好的代码，十足的浪费资源。所以我们需要把这些共同的东西给提取出来，形成一个父类，而这个父类不是具体的产品类，就需要声明为抽象类，然后我们的产品类就应该是这样了：
<pre>
/**
 *产品类父类 
 *@author tzx
 */
public abstract class MyProduct {
	/**
	 *产品名称
	 */
	protected String productName;
	/**
	 *产品规格
	 */
	protected String productSize;
	/**
	 * 添加必要成分
	 */
	public void addIngredient() {
	}
	public String getProductName() {
		return productName;
	}
	public void setProductName(String productName) {
		this.productName = productName;
	}
	public String getProductSize() {
		return productSize;
	}
	public void setProductSize(String productSize) {
		this.productSize = productSize;
	}
	@Override
	public String toString() {
		return "MyProduct [productName=" + productName + ", productSize=" + productSize + "]";
	}
}
</pre>
<pre>
/**
 *普通牙膏
 *@author tzx
 */
public class MyProduct1 extends MyProduct {
	public MyProduct1() {
		addIngredient();
	}
}
</pre>
<pre>
/**
 *儿童牙膏
 *@author tzx
 */
public class MyProduct2 extends MyProduct {
	public MyProduct2() {
		addIngredient();
		delead();
	}
	/**
	 *去铅
	 */
	public void delead() {
	}
}
</pre>
那么工厂类中具体产品的选择，就可以改成下边这样：
<pre>
/**
 *外部获取产品的工厂方法
 */
public static MyProduct getProduct(int type) {
	MyProduct product = null;
	if (0 == type) {
		product = new MyProduct1();
	} else if (1 == type) {
		product = new MyProduct2();
	}
	return product;
}
</pre>

## 要点
好了，一个比较标准的简单工厂模式就出来了，由上边的分析可以知道简单工厂模式的几个要点：
1. 需要有一个生产产品的产品类（工厂）和静态的输出产品的方法，这个方法包含一些必要判断逻辑；
2. 需要有一个抽象的产品父类，定义产品的公共属性和方法；
3. 需要有具体的产品类；
4. 只要提供正确的参数，就能通过静态工厂方法获取到具体的产品。

## 总结
通过上边的一系列分析和实例，可以知道简单工厂模式的一些优点，他可以使消费者不必关心具体产品的创建，只需要一个参数就能得到需要的产品，实现了一定程度的松耦合。
同时，不同的产品不同的子类，也使得后续更容易拓展。
但是由于所有产品都通过工厂创建，不同产品获取的逻辑判断都在工厂方法中，因此具体的产品和工厂的耦合度就必然增大，同时在产品类的基础上也额外的增加了工厂类。
所以，如果产品很多的话，这种简单工厂模式便会变得臃肿而不易维护。

demo源码可在github下载：<https://github.com/tuzongxun/mypattern>