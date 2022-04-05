---
title: java stream api熟悉和理解
date: 2022-2-28 14:43:59
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [java,stream]
---
## 前言
stream api和lambda表达式都是java8出来的，都可以简化代码，也经常会配合着使用。
对于上边的说法，网上是这么说的，但是我自己写的时候，总觉得好像有的时候确实简洁很多，但也有时候似乎反而更麻烦。
所以，根据我目前的理解，我觉得可能它有它的适用场景，但并不能说用它就一定好，但是以后会不会改变这种看法也很难说。
就我目前的了解，stream api主要用在对list集合及数组的操作，最大的优势在于各种中间操作。
stream api是把集合等数据转化成stream，然后提供了一系列的api，尤其是很多返回stram的中间操作api。
单纯的描述可能不好理解，下边是一些基础的示例和对比。
<!--more-->

## 示例
例如有这样一个实体类：
```
public class User {
   private String name;
   private int age;
   private String addr;
   private double money;
   //get/set等方法省略
}
```

然后有这样一个list集合，元素是User。
```
List<User> list = new ArrayList<>();
list.add(new User("tuzongxun", 31,"hubei", 1000));
list.add(new User("zhangsan", 22,"hunan", 2000));
list.add(new User("lisi", 12,"guangdong", 300));
list.add(new User("wangwu", 15,"hubei", 10000));
list.add(new User("zhaoliu", 52,"hubei", 20000));
list.add(new User("qianqi", 66,"jiangxi", 1500));
list.add(new User("huba", 73,"sichuan", 200));
list.add(new User("zhoujiu", 9,"hubei", 20));
```

### 遍历
遍历是list集合常见的操作，用之前的增强for循环可能写成下边这样：
```
for (User user : list) {
   System.out.println(user);
}
```

那么，如果使用stream api就可以写成下边这样：
```
list.stream().forEach(System.out::println);
```
这里首先把list转换成stream流，然后调用forEach进行遍历打印，在打印的时候，实际用到了lambda的方法调用，直接使用lambda表达式则应该是下边这样：
```
list.stream().forEach(e->System.out.println(e));
```
关于lambda的方法调用，我也还在理解中。

### 过滤求和
除了遍历，过滤也是list比较常用的操作。那么就再来一个稍微复杂点的，先过滤，再求和，比如这里求addr是hubei的人的money的和，不用stream api的话，可能写成这样：
```
double money=0;
for (User user : list) {
   if ("hubei".equals(user.getAddr())) {
      money+=user.getMoney();
   }
}
System.out.println(money);
```

那么如果使用stream api，则可以写成这样：
```
double money1=list.stream().filter(e -> "hubei".equals(e.getAddr())).mapToDouble(e->e.getMoney()).sum();
System.out.println(money1);
```
这里可能例子举得不是太好，因为感觉用stream反而麻烦了一些，先要把list转成stream，然后过滤，之后再把money进行求和计算。只是，很多都是stream自带的api，调用起来很方便。

### 过滤求最小值
那么除了上边的，stream里还比较常用的api可能有max、min、count等，例如求hubei的年纪最小的人的年纪，不用stream api时写成这样：
```
List<User> hList=new ArrayList<>();
for(User user:list){
   if ("hubei".equals(user.getAddr())) {
      hList.add(user);
   }
}
list.sort(new Comparator<User>() {
   @Override
   public int compare(User o1, User o2) {
      return o1.getAge()-o2.getAge();
   }
});
System.out.println(list.get(0).getAge());
```

上边sort部分实际还可以改成lambda表达式，例如：
```
list.sort((o1,o2)->o1.getAge()-o2.getAge());
list.sort(Comparator.comparingInt(User::getAge));
```

那么这里如果用stream api，就会看到会简洁很多：
```
int asInt = list.stream().filter(e -> "hubei".equals(e.getAddr())).mapToInt(e -> e.getAge()).min().getAsInt();
System.out.println(asInt);
```

再比如，我要求hubei的人数，不用stream时可以写成这样：
```
long count=0;
for(User user:list){
   if("hubei".equals(user.getAddr())){
      count++;
   }
}
System.out.println(count);
```

那么使用stream api就可以写成这样，看起来也要简洁一些：
```
long count1 = list.stream().filter(e -> "hubei".equals(e.getAddr())).count();
System.out.println(count1);
```

## 总结
stream api实际没有那么难以理解，只要不抵触，稍加练习其实也就知道怎么用了。
不同的api有不同的作用，远不止上边列出来的这些，需要的时候，看一看，练一练，也就会了。
我其实是因为现在项目中有大量的这种代码，所以才要熟悉和理解，也是为了风格上一致。
只是有时候确实会让代码看起来更简洁，也是一种代码趋势。