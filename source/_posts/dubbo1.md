---
title: dubbo学习——基础环境搭建过程及要点记录
date: 2018-03-12 15:09:42
tags: [dubbo,zookeeper,java]
toc: true
categories: java
---
网上关于dubbo的资料其实已经不少了，尤其是还有一个比较全面的官方文档教程。
但是正所谓知易行难，很多事不动动手，就总不知道里边的一些细节。
虽然早就想学一下dubbo，但是奈何需要学的东西太多，于是就以“工作为主导”的出发点，先学了其他技术。
这次，也是基于上边的出发点，我们某个项目需要重构成dubbo的项目，于是便正好从dubbo基础环境搭建开始学一下，并写一篇笔记备忘。
这篇笔记大概会包含如下一些内容：
1、搭建过程记录
2、遇到的问题和解决办法记录，这一点会在搭建过程记录的同时进行说明，不另外指出
<!--more-->

整个过程基本如下：
## zookeeper注册中心
### 注册中心的理解
由于之前学习过springcloud微服务，所以对于zookeeper注册中心的理解，我觉得还是比较容易的。具体的理解可以参考当时写的springcloud的注册中心那篇文章： <http://blog.csdn.net/tuzongxun/article/details/72650100>
比较明显的不同是，这里的zookeeper需要下载安装一个zookeeper软件。
### zookeeper安装
我在安装的时候主要是参考了“小宝鸽”的博客，他写的很是详细，所以我觉得没有必要在重复造轮子，只需要记住在哪里可以查到参考资料。他关于zookeeper安装的博客地址如下：
<http://blog.csdn.net/u013142781/article/details/50395650>

## dubbo管理平台
由于管理平台与业务逻辑本身没有关系，所以我没有深入了解，也是直接参考“小宝鸽”的博客：
<http://blog.csdn.net/u013142781/article/details/50396621>

## dubbo服务者
### 创建项目
根据目前的主流，我这里是在eclipse中创建了一个maven管理的web项目。
### 基础依赖
项目基础依赖主要参考dubbo官方指导文档的说明<http://dubbo.io/books/dubbo-user-book/dependencies.html>
按照这个文档上所说，需要包含如下一些依赖组件，分别是：
dubbo.jar
spring.jar
log4j.jar
commons-logging.jar
javassist.jar
netty.jar
在实际导入的时候会发现，dubbo.jar已经依赖了spring.jar和netty.jar，这个版本也是根据导入的dubbo.jar的版本来定的。
为了避免冲突，也为了使用自己想要的版本的spring和netty，于是在导入dubbo.jar时需要把这两个先进行排除配置，之后的pom.xml中依赖包的配置基本如下：
<pre>
&lt;dependency>
    &lt;groupId>log4j&lt;/groupId>
    &lt;artifactId>log4j&lt;/artifactId>
    &lt;version>1.2.15&lt;/version>
&lt;/dependency>
&lt;dependency>
    &lt;groupId>commons-logging&lt;/groupId>
    &lt;artifactId>commons-logging&lt;/artifactId>
    &lt;version>1.2&lt;/version>
&lt;/dependency>
&lt;dependency>
    &lt;groupId>org.jboss.netty&lt;/groupId>
    &lt;artifactId>netty&lt;/artifactId>
    &lt;version>3.2.5.Final&lt;/version>
&lt;/dependency>
&lt;dependency>
    &lt;groupId>org.springframework&lt;/groupId>
    &lt;artifactId>spring-context&lt;/artifactId>
    &lt;version>4.3.9.RELEASE&lt;/version>
&lt;/dependency>
&lt;dependency>
    &lt;groupId>com.alibaba&lt;/groupId>
    &lt;artifactId>dubbo&lt;/artifactId>
    &lt;version>2.5.3&lt;/version>
    &lt;exclusions>
	    &lt;exclusion>
	    	&lt;groupId>org.springframework&lt;/groupId>
	    	&lt;artifactId>spring&lt;/artifactId>
	    &lt;/exclusion>
	    &lt;exclusion>
	    	&lt;groupId>org.jboss.netty&lt;/groupId>
	    	&lt;artifactId>netty&lt;/artifactId>
	    &lt;/exclusion>
    &lt;/exclusions>
&lt;/dependency>
</pre>

### spring配置
dubbo本身就整合了spring，因此spring的配置是必不可少的，这里也是使用最简单的配置,文件头就先省略：
<pre>
&lt;context:property-placeholder location="classpath:config.properties" />
&lt;import resource="spring-dubbo-provider.xml" />
</pre>

### config.properties配置
配置dubbo服务，需要指定zookeeper注册中心的地址，以及使用dubbo暴露服务的端口等，这些配置配在config.properties文件中：
<pre>
dubbo.zookepper.address=127.0.0.1:2181
dubbo.provider.port=29880
</pre>

### dubbo配置
要把我们的项目作为一个服务注册到zookeeper注册中心，就需要进行相应的配置，配置如下：
<pre>
&lt;beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        
    http://www.springframework.org/schema/beans/spring-beans.xsd        
    http://code.alibabatech.com/schema/dubbo        
    http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    &lt;!-- 提供方应用信息，用于计算依赖关系-->
    &lt;dubbo:application name="dubboServerDemo-pro"/>
    &lt;!-- 注册中心配置，file值为application name加实例名 -->
    &lt;dubbo:registry protocol="zookeeper" address="${dubbo.zookepper.address}" file="dubboServerDemo-pro-1.cache"/>
    &lt;!-- 用dubbo协议在20880端口暴露服务,dubbo访问日志输出到应用日志 -->
    &lt;dubbo:protocol name="dubbo" port="${dubbo.provider.port}" accesslog="true"/>
    &lt;!-- 声明需要暴露的服务接口  超时时间90分钟,超时不重试-->
    &lt;dubbo:service interface="dubboServerTest.DubboServerService" ref="dubboServiceTest" cluster="failfast" timeout="5400000" />
    &lt;!-- 和本地bean一样实现服务 -->
    &lt;bean id="dubboServiceTest" class="dubboServerTest.DubboServerServiceImpl" /> 
&lt;/beans>
</pre>

### dubbo具体服务接口和实现
dubbo的具体服务接口和实现就是很普通的java接口和类，如下：
<pre>
package dubboServerTest;
public interface DubboServerService {
	public void sayHello();
}
</pre>

<pre>
package dubboServerTest;
public class DubboServerServiceImpl implements DubboServerService {
	@Override
	public void sayHello() {
		System.out.println("dubbo测试");
	}
}
</pre>

### web.xml配置
spring.xml文件需要被加载，可以用java代码加载，也可以用其他方式，我这里就选择使用web.xml的配置来加载，web.xml配置如下：
<pre>
&lt;context-param>
    &lt;param-name>contextConfigLocation&lt;/param-name>
    &lt;param-value>classpath:spring.xml&lt;/param-value>
&lt;/context-param>
&lt;listener>
    &lt;listener-class>org.springframework.web.context.ContextLoaderListener&lt;/listener-class>
&lt;/listener>
&lt;welcome-file-list>
    &lt;welcome-file>index.jsp&lt;/welcome-file>
&lt;/welcome-file-list>
</pre>

### 问题说明
按理说,照上边的做法程序应该是可以运行，并且可以再管理平台中显示出服务者的，但是实际上我启动后却抛出了如下异常：
<pre>
java.lang.NoClassDefFoundError: org/I0Itec/zkclient/exception/ZkNoNodeException
</pre>
原因是少了zookeeper的依赖包，需要在pom.xml中补上如下的依赖配置：
<pre>
&lt;dependency> 
    &lt;groupId>com.101tec&lt;/groupId> 
    &lt;artifactId>zkclient&lt;/artifactId>
    &lt;version>0.10&lt;/version>
&lt;/dependency>
</pre>

## 消费者
### 项目搭建
消费者的项目搭建其实和服务者差不多，区别在于dubbo的配置略有区别，消费者的配置大致如下：
<pre>
&lt;!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
&lt;dubbo:application name="consumer-of-helloworld-app"  />
&lt;!-- 使用multicast广播注册中心暴露发现服务地址 -->
&lt;dubbo:registry address="${dubbo.zookepper.address}" />
&lt;!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
&lt;dubbo:reference id="demoService" interface="dubboServerTest.DubboServerService" />
</pre>

同样的，这里的也需要指定注册中心的地址,也是配置在config.properties中：
<pre>
dubbo.zookepper.address=zookeeper://127.0.0.1:2181
</pre>

需要注意的是，这里配置的zookeeper注册中心地址的写法，和服务中心配置的写法也略有不同。

### 接口声明
消费端，需要一个和服务端一样的接口，但是不用实现这个接口。这里的做法实际上和webservice接口比较类似，像cxf、xfire、hessian都差不多。

### 服务调用
在消费端可以直接调用上边声明的接口的抽象方法，而实际上运行的就是服务端的具体实现，调用处的代码如下：
<pre>
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import dubboServerTest1.DubboServerService;
public class MyController {
	public static void main(String[] args) {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
				"classpath:spring.xml");
		DubboServerService dubboServerService = (DubboServerService) applicationContext
				.getBean("demoService");
		dubboServerService.sayHello();
	}
}
</pre>

### 问题说明
消费者这里相对比较简单，但是除了说对于zookeeper注册中心地址的配置处有细节需要注意外，对于接口的声明也需要特别注意。
为了验证是否和webservice一样的，消费端接口包括包名在内都必须和服务端一样，我在声明接口时特意使用了不一样的包名，使得实际的类内容以及在dubbo配置文件中的配置如下：
<pre>
package dubboServerTest1;
public interface DubboServerService {
	public void sayHello();
}
</pre>

<pre>
&lt;dubbo:reference id="demoService" interface="dubboServerTest1.DubboServerService" />
</pre>

然后运行main方法的时候，就会抛出如下异常：
<pre>
Caused by: java.lang.IllegalStateException: Failed to check the status of the service dubboServerTest1.DubboServerService. No provider available for the service
</pre>

由此也就证明，消费端的接口声明，包括包名在内，都必须和服务端一模一样。针对这个问题，有一种推荐的做法，就是把这个接口类打包，让服务端和消费端公用。