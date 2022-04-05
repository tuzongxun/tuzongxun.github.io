---
title: lambda表达式基础
date: 2022-2-28 14:43:58
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [java,lambda]
---
## 前言
早在JAVA8出来的时候，我实际就已经了解过一点点lambda表达式和stream流，但是由于写惯了原来的那种风格，再加上当时的项目中也没有什么人使用，所以只是一扫而过，没有深入了解，也没有进行任何的练习，甚至还有那么一点抵触，觉得lambda读起来太不好懂。
<!--more-->
但是现在过去好几年了，这份新工作的项目中也已经是大量的lambda和stream，因此是时候该好好的了解一番，以和项目组整体风格尽量一致。

## 示例
经过了解，lambda表达式的应用场景是，任何有函数式接口的地方。
什么是函数式接口呢，其实就是只有一个抽象方法的接口，例如常见的线程接口Runnable，只有一个run方法，还有Callable，只有一个call方法，以及常用来比较大小的Comparator，只有一个compare方法，这些都是函数式接口。
lambda表达式可以使代码更加的简洁，例如一个简单的Runnable，不用lambda会写成下边这样：
```
Runnable run1=new Runnable() {
   @Override
   public void run() {
      System.out.println("tuzongxun1....");
   }
};
run1.run();
```

改成lambda，则可以写成下边这样：
```
Runnable run2=()->{
   System.out.println("tuzongxun2....");
};
run2.run();
```

很明显，使用lambda表达式时代码简介了很多。但是实际上上边的代码还是不是最简洁的，可以进一步简写称下边这样：
```
Runnable run3=()-> System.out.println("tuzongxun3....");
run3.run();
```

Runnable是没有返回值的，如果要返回值，则需要使用Callable，下边就再拿Callable做示例，不用lambda时可能会写成这样：
```
Callable<Integer> cal1=new Callable<Integer>() {
   @Override
   public Integer call()
      throws Exception
   {
      return 1;
   }
};
System.out.println(cal1.call());
```

那么使用lambda就可以写成这样：
```
Callable<Integer> cal2=()->{
   return 1;
};
System.out.println(cal2.call());
```

同样的，上边写法也不是最简洁的，还可以进一步简化：
```
Callable<Integer> cal3=()->1;
System.out.println(cal3.call());
```

上边的Runnable的run方法是没有入参也没有返回值，而Callable是没有入参有返回值，下边就再以Comparator为例，这个是既有入参，也有返回值。
这里以list倒序排序为例，假设有这样一个list：
```
List<Integer> list=Arrays.asList(2,1,3);
```

不用lambda的写法如下：
```
list.sort(new Comparator<Integer>() {
   @Override
   public int compare(Integer o1, Integer o2) {
      return o2-o1;
   }
});
```

那么使用lambda可以写成这样：
```
list.sort((Integer o1,Integer o2)->{return o2-o1;});
```

转化为最简洁的写法就可以是这样：
```
list.sort((o1, o2) -> o2-o1);
```

## 总结
那么这里就可以总结一下lambda的基本写法，即：
```
()->{}；
```
**这里的()代表方法的参数，{}代表方法体。**
但是上边的写法是最标准的写法，在很多特殊情况下，都可以进一步简化。
例如在`Runnable run2=()->{System.out.println("running2....");}`这个里边，因为方法体例只有一行代码，所以就可以省略外边的大括号，进而简化为下边这样：
```
Runnable run3=()-> System.out.println("running3....");
```

再例如`Callable<Integer> cal2=()->{return 1;}`中，由于方法体只是简单返回了一个结果，就可以同时去掉大括号和return关键字，进一步简化为：
```
Callable<Integer> cal3=()->1;
```

像上边`list.sort((Integer o1,Integer o2)->{return o2-o1})`这一个简化为`list.sort((o1, o2) -> o2-o1)`，就是去掉了方法体的大括号和return。
但是同时，这里还去掉了入参的类型Integer，原因是lambda可以自动识别参数类型。

实际上，除了上边的内容，入参的小括号也是可以省略的，但是只有当仅有一个参数的时候才能省略，没有参数或者多于一个时，都不能省略。