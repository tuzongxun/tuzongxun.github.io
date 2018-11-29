---
title: 《设计模式》学习笔记6——原型模式
date: 2017-12-4 15:43:58
categories: [[读书笔记,设计模式],[设计模式]] #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [设计模式,读书笔记]
---
## 定义
在开发的过程中，可能会遇到需要为同一个类创建多个对象，而这多个对象的大部分属性值都一样的情况。这时候如果每一个对象都一一设值，就会显得有很多代码重复，原型模式就可以用来解决这种场景，用来精简代码。
<!--more-->
原型模式引用书中的定义如下：
>原型模式(Prototype Pattern)：使用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。原型模式是一种对象创建型模式

## 理解
原型模式可以简单的理解成就是克隆、复制，把某个对象拷贝一份出来。
在我既往开发的过程中，要把一个对象的属性给予另一个对象，通常有两种做法,为了更好的说明，下边先定义一个实体类：
<pre>
package patterntest.prototypepattern;
/**
 *班主任实体类
 *@author tzx
 *@date 2017年12月4日
 */
public class HeadmasterModel {
	/**
	 *老师姓名
	 */
	private String name;
	/**
	 *老师年龄
	 */
	private int age;
	/**
	 *管理班级的名称
	 */
	private String className;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public String getClassName() {
		return className;
	}
	public void setClassName(String className) {
		this.className = className;
	}
	@Override
	public String toString() {
		return "HeadmasterModel [name=" + name + ", age=" + age + ", className=" + className + "]";
	}
}
</pre>

### 做法一
把一个对象的引用直接给予另一个对象，代码如下：
<pre>
HeadmasterModel headmaster = new HeadmasterModel();
headmaster.setAge(32);
headmaster.setClassName("六一班");
headmaster.setName("张威");
HeadmasterModel headmaster2=headmaster;
</pre>

在这里首先有一个headmaster对象，然后直接把headmaster对象赋值给了headmaster2。
对java稍微了解的同学应该知道，这种赋值其实给的是对象的引用，如果我改变了headmaster2的属性值，那么headmaster也会跟着改变，也就是说这两个对象其实就是同一个对象。
那么这种情况的问题就很明显了，如果我只想改变headmaster2的值而并不想改变headmaster的值，似乎是无法做到的。

### 做法二
那么为了解决上边的问题，既想headmaster2使用headmaster的部分属性值，又想改变headmaster2的其他部分值时不影响headmaster，就可以这样做：
<pre>
HeadmasterModel headmaster = new HeadmasterModel();
headmaster.setAge(32);
headmaster.setClassName("六一班");
headmaster.setName("张威");
student.setHeadmaster(headmaster);
HeadmasterModel headmaster2 = new HeadmasterModel();
headmaster2.setAge(headmaster.getAge());
headmaster2.setName(headmaster.getName());
headmaster2.setClassName("六二班");
</pre>
上边的代码其实就是创建了两个对象，然后get第一个对象的值，接着set给第二个对象。
这种情况下，实现了两个对象部分属性的一致，同时如果更改headmaster2的值，也不会影响到headmaster，就解决了第一种做法的问题。
但是，这种做法的问题也是很明显的，如果这个对象的各种属性需要多次被复用，那就需要每一次复用的地方都使用类似`headmaster2.setAge(headmaster.getAge())`这样的代码，被复用的属性越多，重复代码也就越多，因为这种写法是在消费端。

### 原型模式基础
基于上边两种做法所出现的问题，原型模式就呼之欲出了，为了减少重复，我们需要把复用属性的代码从消费端移到服务端，这里的服务端其实就是具体的实现类。
具体做法也就是在实体类里增加一个方法供外部调用，这个方法会复用当前对象的属性赋值给另一个同类对象，然后返回回去，而这个方法通常就叫做clone，就像这样：
<pre>
public HeadmasterModel clone() {
	HeadmasterModel headmasterModel = new HeadmasterModel();
	headmasterModel.setAge(this.age);
	headmasterModel.setClassName(this.className);
	headmasterModel.setName(this.name);
	return headmasterModel;
}
</pre>
这一个方法看起来和上边第二种做法的处理逻辑一样，但是这里因为是写在实体类中，因此在具体消费端使用的时候只需要用原对象调用clone方法即可得到一个和原对象一样属性的新对象。
因为clone方法中new了一个新对象，所以和原对象也是不同的。

### 原型模式晋级
上边的实现方式比较简单，但是实际上还有更简单的。
在上边的例子中我们也还是自己写代码一一从原对象中拿出属性值，然后赋值给新的对象，但是实际上在java的Object类中就实现了clone方法，凡是继承了Object类的子类，都可以使用该clone方法创建新的对象。
**不过这里需要注意的是：需要使用Object的clone方法的对象必须实现Cloneable接口，否则会抛出异常**

因此优化后的clone方法就可以改成下边这样：
<pre>
public HeadmasterModel clone() {
	HeadmasterModel headmasterMode = null;
	try {
		headmasterMode = (HeadmasterModel) super.clone();
	} catch (CloneNotSupportedException e) {
		// e.printStackTrace();
	}
	return headmasterMode;
}
</pre>

同时需要在类上边加上对Cloneable接口的实现：
<pre>
public class HeadmasterModel implements Cloneable
</pre>

### 原型模式深入
通过上边的一系列分析，使用原型模式克隆出一个和原对象属性值一样的新对象就基本实现了，但是其实还存在一个问题，对于非基础类型的属性，我们上边的克隆方法实际上克隆的只是引用。
例如我们这里又有一个学生类，班主任是学生类的其中一个属性,该学生类也拥有clone方法：
<pre>
package patterntest.prototypepattern;
/**
 *学生实体类
 *@author tzx
 *@date 2017年12月4日
 */
public class StudentModel {
	/**
	 *学生姓名
	 */
	private String name;
	/**
	 *学生年龄
	 */
	private int age;
	/**
	 *所属班级名称
	 */
	private String className;
	/**
	 *性别
	 */
	private String sex;
	/**
	 *班主任
	 */
	private HeadmasterModel headmaster;
	public StudentModel clone() {
		StudentModel studentModel = null;
		try {
			studentModel = (StudentModel) super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return studentModel;
	}
	...get、set以及toString方法省略
}
</pre>
然后生成一个学生对象，以此为基础克隆一个新对象：
<pre>
public static void main(String[] args) {
	StudentModel student = new StudentModel();
	student.setName("张三");
	student.setAge(12);
	student.setClassName("六一班");
	student.setSex("男");
	HeadmasterModel headmaster = new HeadmasterModel();
	headmaster.setAge(32);
	headmaster.setClassName("六一班");
	headmaster.setName("张威");
	student.setHeadmaster(headmaster);
	System.out.println(student.getHeadmaster().hashCode());
	System.out.println(student.clone().getHeadmaster().hashCode());
}
</pre>
通过运行上边的代码可以看到克隆之前的student对象的headmaster属性的hashcode和克隆后的student对象的headmaster属性的hashcode是一模一样的，也就是说这两个headmaster对象使用了同一个引用，实际对象还是同一个。
那么这时候如果我们就改动了克隆后的student对象的headmaster的属性，很显然克隆前的那个对象的headmaster的属性也会跟着变化，这明显也是不合理的。
**那么就里就又涉及到原型模式的两个概念：浅克隆和深克隆。**

浅克隆，就是上边示例的这种，虽然克隆的对象是新的，但是对象里边的非基础类型的属性还是旧的。
而深克隆，就是不仅克隆出的对象时新的，连这个新对象的属性也都是新的对象。
由于super.clone()方法调用Object的clone方法实现的是浅克隆，所以深克隆就不能使用这个方法了，深克隆需要实现Serializable接口对对象进行序列化，通过io流生成全新的对象，具体代码如下：
<pre>
public class StudentModel implements Serializable {
	private static final long serialVersionUID = 1L;
	/**
	 *学生姓名
	 */
	private String name;
	/**
	 *学生年龄
	 */
    private int age;
	/**
	 *所属班级名称
	 */
	private String className;
	/**
	 *性别
	 */
	private String sex;
	/**
	 *班主任
	 */
	private HeadmasterModel headmaster;
	public StudentModel clone() {
		StudentModel studentModel = null;
		ByteArrayOutputStream byteArrayOutputStream=new ByteArrayOutputStream();
		try {
			ObjectOutputStream objectOutputStream=new ObjectOutputStream(byteArrayOutputStream);
			objectOutputStream.writeObject(this);
			ByteArrayInputStream byteArrayInputStream=new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
			ObjectInputStream objectInputStream=new ObjectInputStream(byteArrayInputStream);
			studentModel=(StudentModel)objectInputStream.readObject();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		return studentModel;
	}
	...get、set以及toString方法省略
}
</pre>
这个时候如果再运行上边的测试main方法，会发现会抛出异常，原因是深克隆不仅被克隆的对象需要实现Serializable接口，被克隆的对象里边需要被深克隆的属性对象在定义时也需要实现Serializable接口：
<pre>
public class HeadmasterModel implements Cloneable, Serializable 
</pre>

然后再运行测试方法，就会发现这时候不仅被克隆的对象和原对象不一样了，克隆后的对象的非基本类型的属性也不再是原属性对象。这时候如果修改克隆之后的对象的改属性，就不会再影响克隆之前的对象。

## 要点
根据上边的分析可知原型模式的要点大概有这样几个：
1. 需要有一个对象作为原型
2. 可以自己实现克隆方法，实现对原型的复用，但如果是浅克隆则使用Object的clone方法更简洁
3. 深克隆必须自己实现
4. 使用Object的clone方法必须实现Cloneable接口
5. 实现深克隆时，必须保证被克隆对象及属性对象都实现序列化接口Serializable
6. 为了更好的拓展及更简洁的代码，可以考虑在父级抽象类中实现克隆方法

## 总结
通过上边的分析和示例可以发现，原型模式的主要优点就是：在其他同类对象需要大量重用某个对象的属性时，可以使代码更简洁，大大减少代码的重复度。
而这种模式的缺点也是很明显的，首先是被克隆的对象需要有克隆方法，需要根据情况实现Cloneable和 Serializable接口；其次就是实现深克隆的对象的每个非基本类型的属性对象，在定义时也都必须实现Serializable接口，这从某种程度上说也增加了代码量。

demo源码可在github下载：<https://github.com/tuzongxun/mypattern>