---
title: java基础之结合源码理解字符串类的重要知识点
date: 2022-3-15 18:43:58
categories: 运维 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [运维,jenkins]
---
## 字符串类
字符串类主要是指String、StringBuffer和StringBuilder，从源码注释可以看到，String和StringBuffer都是jdk1.0就有的，而StringBuilder则是jdk1.5才有。
一般来说，最常用的是String，是不可变的，然后是可变的StringBuilder和StringBuffer，其中StringBuffer是线程安全的，因为里边的方法都是加了synchronized关键字的。
StringBuilder和StringBuffer都是继承自AbstractStringBuilder类，里边很多方法也都是共用的这个抽象类你的逻辑，所以除了是否线程安全，其他的基本都一样。
<!--more-->
## 理解字符串不能被继承
上述三个字符串类都不能被继承，原因是这几个类定义都是final的，如：
```
public final class String
```
fainal修饰类的时候不能被继承，修饰方法的时候不能被重写，修饰基础类型变量不能改变值，修饰引用类型变量，不能改变引用。

## 理解字符串是不可变的
String是不可变的，指的是一个字符串变量一旦创建后，所指向的引用的内容不能再变，如果要改变这个变量的值，实际上会同时改变变量的引用。
StringBuilder和StringBuffer可变，指的是创建之后可以在这个引用不变的情况下改变里边的值，而实际上是改变的这个引用对象里边的数组的值，在jdk1.8中指的就是字符数组，jdk12指的是byte数组。

## 理解String中的equals
equals方法是Object类中的方法，在不重写的情况下实际就是直接比较的两个对象的引用，Object中equals源码如下：
```
public boolean equals(Object obj) {
	return (this == obj);
}
```
String中重写了equals方法，所以在String中的equals方法不再是直接比较String对象的引用，jdk1.8里String中equals源码如下：
```
public boolean equals(Object anObject) {
	if (this == anObject) {
		return true;
	}
	if (anObject instanceof String) {
		String anotherString = (String)anObject;
		int n = value.length;
		if (n == anotherString.value.length) {
			char v1[] = value;
			char v2[] = anotherString.value;
			int i = 0;
			while (n-- != 0) {
				if (v1[i] != v2[i])
					return false;
				i++;
			}
			return true;
		}
	}
	return false;
}
```
上边的逻辑中可以看到，当两个对象引用相同时，就会返回true，当引用不同时，会先判断类型然后进行强转，之后再对两个字符串底层的字符数组元素进行遍历依次比较大小，所有元素都相等时返回true。
注：在jdk12中，String的equals进行了重写，逻辑进行了很大的改变，逻辑也没有上边这么直观了，根本原因好像是底层存储变了，jdk1.8底层是字符串数组，而jdk12则是byte数组，这种改动具体是从jdk哪个版本开始的，暂未细究。

## 从源码看compareTo
compareTo是Comparable接口中的方法，用来比较两个对象大小，String类实现了这个接口并实现了compareTo方法，在jdk1.8中相应的源码如下：
```
public int compareTo(String anotherString) {
	int len1 = value.length;
	int len2 = anotherString.value.length;
	int lim = Math.min(len1, len2);
	char v1[] = value;
	char v2[] = anotherString.value;

	int k = 0;
	while (k < lim) {
		char c1 = v1[k];
		char c2 = v2[k];
		if (c1 != c2) {
			return c1 - c2;
		}
		k++;
	}
	return len1 - len2;
}
```
这个方法的逻辑也比较直观，就是分别取了两个字符串的底层字符数组的长度，然后再取最小的那个的长度来做循环，之后一次判断每个位置的字符的大小。
注：同样的，由于jdk12底层存储的改变，compareTo的实现也发现了很大的变化。

## 从源码看replace
replace用来进行字符串内容的替换，jdk1.8中源码如下:
```
public String replace(CharSequence target, CharSequence replacement) {
	return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
			this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
}
```
可以看到replace里边使用了正则表达式相关的一些方法，如果再进去replaceAll方法，可以看到里边还用到了StringBuffer和StringBuilder。

注：同样的，jdk12中这个方法的逻辑也发生了很大变化。

## 关于字符串拼接的详细分析
字符串拼接如果是String，一般都是直接用"+"，如果是StringBuilder或者StringBuffer，则是使用append。
实际上，jdk8中用"+"拼接也不全是一样的，例如如下代码：
```
public static void main(String[] args) {
	String a1="ab"+"cd";

	String a="ab";
	String b="cd";
	String d=a+b;

	StringBuilder sb=new StringBuilder("ab");
	sb.append("cd");
}
```
上边代码有三个字符串的拼接操作，一个是直接字面量拼接，一个是字符串变量之间拼接，还有一个是StringBuilder的拼接。
结果javap工具执行"javap -c xxx.class"可以查看编译的过程，编译过程如下：
```
0: ldc           #2                  // String abcd
2: astore_1
3: ldc           #3                  // String ab
5: astore_2
6: ldc           #4                  // String cd
8: astore_3
9: new           #5                  // class java/lang/StringBuilder
12: dup
13: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
16: aload_2
17: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
20: aload_3
21: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
24: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
27: astore        4
29: new           #5                  // class java/lang/StringBuilder
32: dup
33: ldc           #3                  // String ab
35: invokespecial #9                  // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
38: astore        5
40: aload         5
42: ldc           #4                  // String cd
44: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
47: pop
48: return
```
上边内容很多，需要有一些jvm基础后才能较全面的理解，但是这里其实可以仅针对后边的注释先进行一定的理解，主要关注这样几行：
```
0: ldc           #2                  // String abcd

3: ldc           #3                  // String ab
6: ldc           #4                  // String cd
9: new           #5                  // class java/lang/StringBuilder
13: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
17: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
21: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
24: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;

29: new           #5                  // class java/lang/StringBuilder
33: ldc           #3                  // String ab
35: invokespecial #9                  // Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
42: ldc           #4                  // String cd
44: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
```
从上边的内容可以看到，针对字符串字面量的加号拼接，实际上jvm在编译的时候就进行了优化，编译出来的class文件就已经拼成了一个字符串。
针对变量的加号拼接，会先定义两个String字符串变量，然后创建StringBuilder对象再进行初始胡和append的拼接操作，最后再使用toString方法转回String。
针对StringBuilder的，先创建StringBuilder对象，然后创建了String类型的变量，之后再初始化和append拼接。

## 关于StringBuild扩容
String和StringBuilder底层都是用的数组存储，jdk8中是字符数组，之后有的版本是byte数组，而数组长度是不可变的，因次使用StringBuilder的append进行字符串拼接的时候就涉及到底层数组的扩容，在jdk8源码中主要是下边的一些代码：
```
public AbstractStringBuilder append(String str) {
	if (str == null)
		return appendNull();
	int len = str.length();
	ensureCapacityInternal(count + len);
	str.getChars(0, len, value, count);
	count += len;
	return this;
}

private void ensureCapacityInternal(int minimumCapacity) {
	// overflow-conscious code
	if (minimumCapacity - value.length > 0) {
		value = Arrays.copyOf(value,
				newCapacity(minimumCapacity));
	}
}

private int newCapacity(int minCapacity) {
	// overflow-conscious code
	int newCapacity = (value.length << 1) + 2;
	if (newCapacity - minCapacity < 0) {
		newCapacity = minCapacity;
	}
	return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
		? hugeCapacity(minCapacity)
		: newCapacity;
}
```
这里首先会取一个实际使用的数组长度和新增字符串的长度的和，然后传给扩容的方法。
在扩容方法里拿这个参数和底层数组长度进行比较，当超过数组长度时则进行底层数组的扩容。
在扩容的时候可以看到，如果长度超过了Integer.MAX_VALUE，则抛出内存溢出异常，也就是这个数组最大长度是Integer.MAX_VALUE，正常情况下扩容是在新的实际字符串长度基础上乘以2，然后加2。