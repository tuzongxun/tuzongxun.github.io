---
title: 《设计模式》学习笔记3——工厂模式
date: 2017-11-21 13:43:58
categories: 设计模式 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 设计模式
---
## 定义
工厂模式实际上有广义和狭义的分别，广义的工厂模式指的是简单工厂模式、工厂方法模式、抽象工厂模式三个，而狭义的工厂模式就是这里的工厂方法模式，一般情况下如果有人直接说工厂模式，多半指的就是工厂方法模式。工厂方法模式引用书中的定义如下：
<!--more-->
>工厂方法模式(Factory Method Pattern)：定义一个用于创建对象的接口，让子类决定将哪一个
类实例化。工厂方法模式让一个类的实例化延迟到其子类。工厂方法模式又简称为工厂模式
(Factory Pattern)，又可称作虚拟构造器模式(Virtual Constructor Pattern)或多态工厂模式
(Polymorphic Factory Pattern)。工厂方法模式是一种类创建型模式

## 理解
对于工厂方法模式的定义，我觉得首先需要注意的是几个概念上的东西，也就是这个模式的几个别称，如虚拟构造器模式、多态工厂模式，根据以往面试的经验，如果考察知识面，某些公司是极有可能用这些别称来代替大家熟知的工厂模式的。
那么要理解工厂方法模式，结合之前的简单工厂模式应该就很容易了。
在之前的简单工厂模式中，有一个抽象产品类、若干个具体产品类、一个工厂类和一个静态工厂方法、还有一个消费者类。
之前说到工厂类的工厂方法是提供给外部获取产品的方法，而实际上这个方法内部也负责了产品的创建，所以也有一些地方说这个工厂方法是创建产品对象的方法。
而我认为这些都不重要，重要的是我的目的只是想要`产品的创建和消费者分离`，实现松耦合。所以不论对那个工厂方法的描述和理解是什么，只要知道这个方法的作用是不再需要消费者自己创建产品对象就够了。
但是，简单工厂模式中的这个工厂却有这一定的缺陷存在，并不适用于所有的场合。
之前的例子中以一个生产儿童牙膏和普通牙膏的工厂作为例子，现在我的实现依然要停留在这个制造牙膏的工厂中。
如果这个工厂的经营一直一成不变，永远都只生产这两种牙膏，那么之前的简单工厂完全没有问题，足以胜任，但是这只是一个假设，现实中的工厂总是要发展壮大的。
随着时间的推移和利润的增长，工厂又引进了各种产品，例如特效美白牙膏、超级环保牙膏，不止是牙膏，还有儿童牙刷、成年人牙刷、普通刷牙杯、创意刷牙杯等等，那么之前的工厂方法将变成这样：
<pre>
	/**
	 *外部获取产品的工厂方法
	 */
	public static MyProduct getProduct(int type) {
		MyProduct product = null;
		if (0 == type) {
			// 普通牙膏
			product = new MyProduct1();
		} else if (1 == type) {
			// 儿童牙膏
			product = new MyProduct2();
		} else if (2 == type) {
			// 特效美白牙膏
		} else if (3 == type) {
			// 超级环保牙膏
		} else if (4 == type) {
			// 儿童牙刷
		} else if (5 == type) {
			// 成年人牙刷
		} else if (6 == type) {
			// 普通刷牙杯
		} else if (7 == type) {
			// 创意刷牙杯
		}
		return product;
	}
</pre>
上边的代码基本没有太具体的处理，已经有点让人眼花，如果再加上更具体的处理，可能每个if里边都会占满一个屏幕，那么这样的代码查找问题以及后期维护将会是一场噩梦。
如果把这个类回归实际生产工厂，每一个产品是一条生产线，那么如此多的if和else就会是如此多的生产线。
这么多的生产线放在一个工厂车间中，且不说空间的限制能否容得下，单说管理方面，就会是一种剪不断理还乱。
如果上边的工厂遇到这样的问题，势必会增加分厂，增加车间。然后会有一个管理所有分厂的总厂，不负责实际的生产。
而考虑到之前的产品发展时间长，需求量太大，因此只是单个的产品可能就需要一个单独的工厂来生产，因此分厂的设计便是一个分厂只负责一个产品。
代码的逻辑本身就来源于现实中，那么上边的工厂就需要改变，需要增加工厂类和工厂方法，犹如分厂。
还需要有一个抽象的工厂类和抽象工厂方法作为总厂，便是所谓的抽象父类。
那么优化后的工厂方法就需要改成如下，首先需要有一个抽象的工厂父类，可以使抽象类，可以使普通类，也可以是接口，但是通常可能就声明为接口或抽象类，我这里就声明为一个接口：
<pre>
package patterntest.factorypattern;
import patterntest.product.MyProduct;
public interface MyFactory {
	/**
	 *外部获取产品的工厂方法,父类工厂接口
	 */
	public MyProduct getProduct(int type);
}
</pre>
这里定义了一个提供给外部获取产品的统一接口，基本和简单工厂方法里的定义一样，不同的是这个方法不再是静态方法，也没有了方法实体。
至于为什么没有方法实体，这个很好理解，是因为这个接口中这个方法并不需要具体的生产产品，具体的生产工作需要分厂来做，所以他就应该是抽象的没有实体的方法。
而为什么简单工厂模式里的工厂方法是static静态的，这里却去掉了static呢？那是因为虽然static的方法也可以被继承，但是却“不能被覆盖”。
对于非static的方法，如果子类重写了这个方法，当我们用父类调用这个方法的时候，会调用实际子类的这个方法。
但是如果是static的方法，子类也“重写”了这个方法，那么再使用父类调用这个方法的时候，调用的还是父类的这个方法，也就是并不能实现多态性。
为什么上边的重写要打引号？是因为这个“重写”，不是我们平常理解的那个重写，看起来是和父类一模一样的方法，但是不能使用`@Override`注解，相当于就是互相独立没有关系的方法。
有了接口和抽象方法，我然后就需要有实际工厂类和方法，但是为了演示，首先还需要提供抽象产品类和实际产品类，这里只演示三个：
<pre>
package patterntest.product;
/**
 *产品类父类
 * @author tzx
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
public class MyProduct1 extends MyProduct {
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
<pre>
package patterntest.product;
/**
 *特效美白牙膏
 *@author tzx
 */
public class MyProduct3 extends MyProduct {
	public MyProduct3() {
		addIngredient();
		addWhitening();
	}
	/**
	 *增加特效美白成分
	 */
	public void addWhitening() {
	}
}
</pre>
有了具体的三个产品，根据上边的理念，每个工厂生产一种产品，也就是每个产品要对应一个实际工厂，所以我们需要有三个对应的工厂,这些工厂均实现工厂接口的抽象方法：
<pre>
package patterntest.factorypattern;
import patterntest.product.MyProduct;
import patterntest.product.MyProduct1;
/**
 *生产普通牙膏的分厂 
 *@author tzx
 */
public class MyFactory1 implements MyFactory {
	@Override
	public MyProduct getProduct() {
		return new MyProduct1();
	}
}
</pre>
<pre>
package patterntest.factorypattern;
import patterntest.product.MyProduct;
import patterntest.product.MyProduct2;
/**
 *生产儿童牙膏的分厂
 *@author tzx
 */
public class MyFactory2 implements MyFactory {
	@Override
	public MyProduct getProduct() {
		return new MyProduct2();
	}
}
</pre>
<pre>
package patterntest.factorypattern;
import patterntest.product.MyProduct;
import patterntest.product.MyProduct3;
/*
 *生产特效美白牙膏的分厂
 */
public class MyFactory3 implements MyFactory {
	@Override
	public MyProduct getProduct() {
		return new MyProduct3();
	}
}
</pre>
这样一来，工厂就可以正常开始运行了，接下来是消费者找工厂购买商品，只需要根据实际需要创建时间的工厂，然后调用生产方法，就可以得到想要的产品：
<pre>
package patterntest.consumer;
import patterntest.factorypattern.MyFactory;
import patterntest.factorypattern.MyFactory1;
import patterntest.factorypattern.MyFactory2;
import patterntest.factorypattern.MyFactory3;
/**
 *消费者
 *@author tzx
 */
public class Consumer {
	public static void main(String[] args) {
		MyFactory factory = null;
		// 我需要普通牙膏
		factory = new MyFactory1();
		factory.getProduct();
		System.out.println(factory.getProduct().getClass());
		// 我需要儿童牙膏
		factory = new MyFactory2();
		factory.getProduct();
		System.out.println(factory.getProduct().getClass());
		// 我需要特效美白牙膏
		factory = new MyFactory3();
		factory.getProduct();
		System.out.println(factory.getProduct().getClass());
	}
}
</pre>

## 要点
那么，根据上边的实例可以归纳出工厂方法的要点基本如下：
1. 需要有一个抽象产品类
2. 需要有若干个具体的产品类
3. 需要有一个抽象工厂和抽象工厂方法
4. 需要每个具体产品对应一个具体的工厂和工厂方法
5. 抽象工厂中的抽象工厂方法需要是非static的
6. 消费者需要自己根据实际需要创建实际的工厂

## 总结
工厂方法模式相对于简单工厂模式，就是减轻了工厂类和工厂方法的工作，把简单工厂模式中的一个工厂类根据具体产品拆分成不同的工厂，并提供一个父类接口。
这样做不仅实现了产品和消费者的分离，也实现了具体工厂的轻负荷，使用面向对象的多态性在运行期选择具体的工厂。
但是这种模式中，缺点也是显而易见的，每一个产品都需要有一个产品工厂，就相当于每增加一个产品，都至少要增加两个类，大大增加了类的个数。
另外，根据简单工厂模式那里的理解，自己new一个工厂，相当于为了一个牙膏创建一个工厂，这似乎也是不合理的。
但是需要明白的是，现在的场景不一样了，每个工厂创造的是一批牙膏而不是一支，所以这里new一个工厂，其实可以理解成我需要一批牙膏，于是我找了个工厂代理生产了一批。

注：书中实际上还提到了对工厂方法模式的优化，那就是在使用的时候使用xml配置文件和反射来得到具体的工厂类，而不是直接new，是否要这样优化可以根据每个人的实际需要选择。

demo源码可在github下载：<https://github.com/tuzongxun/mypattern>