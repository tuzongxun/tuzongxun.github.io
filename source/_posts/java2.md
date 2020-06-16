---
title: 记java动态加载配置文件不成功的问题
date: 2018-3-2 15:09:42
tags: java
toc: true
categories: java
---
## 一、问题背景：
在我们之前的项目中，很多数据是配置在application.properteis文件中的，这样很多时候修改了数据后，只需要重启应用，而不需要重新打包编译。
但是近期有一个新的需求，运维希望不仅不用重新打包编译，即便是重启，也要省掉。

<!--more-->
## 二、问题描述：
之所有之前修改了数据后需要重启应用，是因为我们的项目中application.properties文件只会加载一次，然后就放在内存中供程序调用。
那么要实现不重启的思路，很容易就能想到一个，那就是每次需要调用配置文件里的类容的时候，就再加载一次配置文件，代码有很多种，其中一个可以是类似下边这样的：
<pre>
public void sayAaa(){
Properties pro = new Properties();
String path = this.class.getClassLoader().getResource("aaa.properties").getPath();
try {
	InputStream is = new FileInputStream(path);
	pro.load(is);
	aaa = pro.getProperty("aaa");
	is.close();
	pro.clear();
} catch (IOException e) {
	e.printStackTrace();
}
System.out.println(aaa);
}
</pre>

按照正常的想法，这样应该是可以实现每次修改了配置文件后，再调用sayAaa方法就打印更改后的数据的，我那个负责这个任务的同事也就是这样做的。
但是当他实际上在eclipse中测试的时候，却发现怎么改，再次调用sayAaa之后都是原来的值，让他一度怀疑代码有问题。

## 三、问题分析：
实际上，他的这种代码是没问题的，确实可以实现每次调用都再次读取相应的文件。
那么他eclipse中测试却并不能得到想要的结果，这又是怎么回事呢？
原因很简单，那就是他的这个功能实现的过程实际上分了这样几个步骤：
1、写代码，开发阶段；
2、测试，代码编译阶段；
3、测试、代码运行阶段。
这样一分，可能问题就比较清晰了：他在eclipse中改配置文件，实际上改的是开发阶段的文件，在编译前的路径中；而测试时，这个文件会在编译的时候放到编译后的路径中。
代码运行之后，修改了配置文件，然后也重新调用了sayAaa方法，但是调用的是编译后的路径下的文件。
而配置文件修改之后，并没有重新编译过，因此编译后的路径下的文件还是修改之前的那个文件的内容，因此也就出现那种怎么测试都感觉代码没生效的错觉了。
