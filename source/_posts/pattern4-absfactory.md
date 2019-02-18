---
title: 《设计模式》学习笔记4——抽象工厂模式
date: 2017-11-22 15:43:58
categories: 设计模式 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 设计模式
---
## 定义
在进行工厂方法模式学习的时候，发现工厂方法模式有一个明显的缺陷，即每增加一个具体的产品都需要至少增加两个类，一个产品类，一个对应的工厂类。
这种情况在产品特别多的情况下，显然就更有问题，然后便有了抽象工厂模式，来解决这个问题。
<!--more-->
抽象工厂模式引用书中的定义如下：
>抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，它是一种对象创建型模式

## 理解
抽象工厂模式相对于前两个工厂模式都要难以理解一些，根据书中所说，要理解抽象工厂模式，需要先引入`产品族`和`产品等级结构`两个概念。
实际上我觉得这两个概念似乎比抽象工厂模式的定义更难理解，就我个人而言，我认为`产品族`不如叫`产品组`，组的概念应该远比族要好理解。
不过呢，如果能够直接理解`产品族`，还是不要替换成我所谓的`产品组`比较好，因为`产品族`实际上是特殊的`产品组`。
产品组很容易理解，就是一组产品，但是这一组产品不一定要有什么共性，可以是随意放在一起的，我们可以说他们就是一组。
而产品族为什么是特殊的产品组，是因为它是具有共性的产品组，这个共性，需要根据实际情况进行分析。
所以，如果一开始无法直接理解产品族的概念，完全可以先理解一下产品组的概念，然后进行一个迂回的理解。
就如这样：
**海尔冰箱、康佳电视、美的空调，如果把他们都是电器的共性去掉的话，他们放在一起就是一个普通的产品组；**
**海尔冰箱、海尔电视、海尔空调，他们都是海尔制造的，这是共性，然后就是一个海尔牌的产品族；**


理解了产品族的概念，接下来就是产品等级结构，根据我的个人理解，简单点说产品等级结构就可以理解为同一类不同型号、或不同品牌、或者细节特点不同的产品，例如冰箱是一类产品，然后有海尔冰箱、美的冰箱、奥马冰箱，这三个不同品牌的冰箱组成了一个产品等级结构。
**那么结合我之前在工厂方法模式中的例子，就可以理解成儿童牙膏、普通牙膏、特效美白牙膏，他们都是牙膏的子类，是一个产品等级结构；再入普通牙刷、儿童牙刷、创意牙刷，都是牙刷，也可以是一个产品等级结构。**

**所以说，产品族就是不同类但有共性的一组产品，而产品等级结构是同类但有所区别的一组产品，他们都是特殊的产品组，这是我的理解。**
当然了，这种共性就看自己实际应用的时候怎么分析，就像我非要说冰箱、电视、空调也是一个产品等级结构似乎也不是不行，但这个产品的着重点就在于“电器”，也就是他们的父类。

那么理解了产品族和产品等级结构的概念后，接下来就是理解抽象工厂方法的定义了。
定义中`一系列相关或相互依赖对象`实际就是我们所说的产品族，一组有共性但不同类的产品，就例如儿童牙膏、普通牙膏、儿童牙刷、普通牙刷这四样东西的定义如下：
<pre>
package patterntest.product;
/**
 *牙膏类产品父类
 *@author tzx
 */
public abstract class FatherProduct1 {
	/**
	 *产品名称
	 */
	protected String productName;
	/**
	 *产品规格
	 */
	protected String productSize;
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
<pre>
package patterntest.product;
/**
 *普通牙膏
 *@author tzx
 */
public class MyProduct1 extends FatherProduct1 {
	public MyProduct1() {
		addIngredient();
	}
}
</pre>
<pre>
package patterntest.product;
/**
 *儿童牙膏
 *@author tzx
 */
public class MyProduct2 extends FatherProduct1 {
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
<pre>
package patterntest.product;
/**
 *牙刷类产品父类
 *@author tzx
 */
public abstract class FatherProduct2 {
	/**
	 *产品名称
	 */
	protected String productName;
	/**
	 *产品规格
	 */
	protected String productSize;
	/**
	 *组装
	 */
	public void assemble() {
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
package patterntest.product;
/**
 *普通牙刷
 *@author tzx
 */
public class MyProduct3 extends FatherProduct2 {
	public MyProduct3() {
		assemble();
	}
}
</pre>
<pre>
package patterntest.product;
/**
 *普通牙膏
 *@author tzx
 */
public class MyProduct4 extends FatherProduct2 {
	public MyProduct4() {
		assemble();
		delead();
	}
	/**
	 * 去铅
	 */
	public void delead() {
	}
}
</pre>
这里有两个抽象产品类，一个是牙膏类，一个是牙刷类，他们各有两个子类。
任意一种牙膏和牙刷的组合，不去考虑都是洗刷工具的情况下，如果他们是同一个工厂生产的，可以说他们就是一个产品族，那么这里便又有了一个对设计模式中产品族的新理解，那就是需要被同一个工厂生产的。而不论是两个牙膏子类的组合，还是两个牙刷子类的组合，就是一个产品等级结构。
例如我这里把儿童牙刷和儿童牙膏作为一个产品族，普通牙刷和普通牙膏作为一个产品族，我的抽象工厂就应该是这样的：
<pre>
package patterntest.absfactorypattern;
import patterntest.product.FatherProduct1;
import patterntest.product.FatherProduct2;
/**
 *产品族的抽象工厂
 *@author tzx
 */
public interface MyAbsFactory {
	/**
	 *获取牙膏
	 *@return
	 */
	public FatherProduct1 getProGroup1();
	/**
	 *获取牙刷
	 *@return
	 */
	public FatherProduct2 getProGroup2();
}
</pre>
然后针对两个不同的产品族：普通牙膏和牙刷、儿童牙膏和牙刷，对用的实际工厂是这样
<pre>
package patterntest.absfactorypattern;
import patterntest.product.FatherProduct1;
import patterntest.product.FatherProduct2;
import patterntest.product.MyProduct1;
import patterntest.product.MyProduct3;
public class MyFactory1 implements MyAbsFactory {
	/**
	 *普通牙膏
	 */
	@Override
	public FatherProduct1 getProGroup1() {
		return new MyProduct1();
	}
	/**
	 *普通牙刷
	 */
	@Override
	public FatherProduct2 getProGroup2() {
		return new MyProduct3();
	}
}
</pre>
<pre>
package patterntest.absfactorypattern;
import patterntest.product.FatherProduct1;
import patterntest.product.FatherProduct2;
import patterntest.product.MyProduct2;
import patterntest.product.MyProduct4;
public class MyFactory2 implements MyAbsFactory {
	/**
	 *儿童牙膏
	 */
	@Override
	public FatherProduct1 getProGroup1() {
		return new MyProduct2();
	}
	/**
	 *儿童牙刷
	 */
	@Override
	public FatherProduct2 getProGroup2() {
		return new MyProduct4();
	}
}
</pre>

## 要点
根据上边的实例，抽象工厂模式基本可以归纳为如下几个要点：
1. 需要两个以上的抽象产品类，不同类的产品才能组成一个产品族
2. 每个抽象产品类需要有若干具体的产品类
3. 需要有一个抽象工厂类（根据需要也可以是多个），里边要有和产品等级结构个数对应的抽象工厂方法
4. 需要有至少一个具体工厂类，根据具体有几个产品族来定工厂类的个数
5. 客户端调用基本和工厂方法模式一样

## 总结
从之前工厂方法模式的例子以及上边的例子中可以看到，在之前工厂方法模式中，每一个产品都对应一个具体的工厂类，一个工厂类只有一个工厂方法。
**而现在，我们是每一个产品族对应一个工厂类，然后每一个产品等级结构对应一个工厂方法。**
这样一来必然大大减少工厂类的创建，如果再有新的产品族加入进来，例如有新的创意牙膏、创意牙刷组成一个产品族，那么就可以创建一个新的工厂。在这里，我们虽然加了两种产品，但只需要一个工厂类，这样也不用修改原来的代码，符合开闭原则。
但是这种模式也有一个很明显的问题，我们增加新的产品族是容易了，但是增加新的产品等级结构却必须要修改源代码。
例如现在的产品等级结构有两个，一个是牙刷类产品，一个是牙膏类产品，如果我们要再增加一个牙杯类产品，就必须在抽象工厂类和每个子类工厂都增加一个获取牙杯类产品的工厂方法，这就破坏了开闭原则。
但是每种模式偏重的点不一样，都有自己适合的地方，所以抽象工厂模式的缺点也需要实际应用的时候进行选择。

demo源码可在github下载：<https://github.com/tuzongxun/mypattern>

