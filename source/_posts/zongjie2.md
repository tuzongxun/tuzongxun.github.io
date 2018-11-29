---
title: 记一次开发过程中的思维转换
date: 2018-2-271 11:43:58
categories: [总结，技术总结] #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [总结，技术总结]
---
**一、问题：**
有一个同事接到一个新的开发任务，需要把spring框架改成springboot，但是他遇到了一个久久不能解决的问题。
<!--more-->
原本的项目中有一个类似这样的类：
<pre>
public class MyTest{
    private String name;
    public String getName(){
         return name;
    }
    public void setName(String name){
        if(name!=null&&name.length &lt= 0){
                this.name="defName";
        }
    }
}
</pre>

这是一个比较常规的拥有get、set方法的类，稍微特殊的是这个set方法里边有一定的逻辑处理。
之前的项目中，与这个类配套的是一个spring的xml配置文件，在配置文件中给这个类实例化，配置文件大概如下：
<pre>
&lt;bean id="myTest" class="com.test.MyTest">
    &lt;property name="name">
          &lt;value>${name}&lt;/value>
    &lt;/property>
&lt;/bean>
</pre>
然后就是在properties文件中给name设置了具体的值。
那么他把spring改成springboot的打算，是去掉xml中的配置，直接用注解的方式给name属性赋值。
问题就在于，如果直接在name属性上加@value注解，就丢失了原本set方法中的逻辑处理，如果保留set方法中的逻辑处理，就无法成功从properties文件中获取需要的值。

**二、思路转换**
然后，他找到我一起讨论这个问题，而我实际上也不知道怎么两者兼容。
但是，经过思考和分析，我觉得在这件事上，并不是说必须要保证上述两者的兼容。
在这里，我们set的目的只是为了给name赋予一个正确的、我们想看到的值，重要的并不是set，而是get，因为get到的内容才是后边我们要拿来用的。
那么很显然我们就可以转换一下思路，并不是说一定要那段逻辑在set方法中，完全可以换到get中写。
虽然逻辑实现的地点变了，但是最终拿到的内容并没有区别，也一样实现了应有的功能。

**三、感想**
软件开发的过程，就是一个设计的过程，设计虽说有一些规律可循，但细节上并不是一成不变。
很多时候直线走不通，完全可以绕一点路，或许到达目的地的时间能比直线更快。
软件设计的目的并不在于软件设计的本身，而是用软件来解决实际问题，如果一味纠结与代码本身，而不是要解决的实际问题，可能很容易陷入一个死胡同，严重拉低工作的效率。