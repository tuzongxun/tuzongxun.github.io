---
title: springcloud微服务七：网关zuul
date: 2018-9-15 15:09:42
tags: springcloud
toc: true
categories: springcloud
---
# 理解网关概念
根据我个人的理解，对于一个系统来说，不管是微服务架构还是非微服务架构，如果单从功能实现来说，网关其实不是必要的。
但是，如果一个系统的业务涉及到了和外部网络的交互，多数情况下都会加入网关功能，最根源的目的就是提升系统的网络安全性。
因此网关可以理解为是在大的系统架构层面，一个网络策略方面的东西，最基本的需求就是实现请求的路由转发。
<!--more-->
但是有的时候，可能网关是一种功能上的实现，而没有叫做网关，就例如在我们目前的一个支付系统中，大的架构大致如下：
![cloud1](/images/springcloud/s71.png)
我们把支付核心功能放在了内网中，dmz域的子系统可以访问，而外部网络的其他系统不能直接访问，这里dmz域的前置系统实际上就做了一部分网关的功能，但是没有被称作网关。
之所以没有被称作网关，可能是因为我们这两个前置不仅做了部分请求转发功能，还实现了一部分业务相关的逻辑。
而比较纯粹的网关，一般来说应该是不直接涉及业务层面的，就如这里要说的spring cloud整合的zuul，他就主要包含两个功能：路由转发和请求过滤。
这里的路由转发又分为传统路由转发和面向服务的路由转发，我认为zuul的优势实际上就是后一种，面向服务的路由转发。
因为传统路由转发，转发的目标都是要配置具体的ip或主机名的，在子系统特别多的情况下，就会导致配置异常繁琐。
面向服务的路由转发，就结合ribbon负载均衡，面向服务名进行转发，由于多个部署节点服务名相同，这就大大降低了配置复杂度。
所以，这里我就不再记录传统路由转发的方式，只记录面向服务的路由转发。

# zuul面向服务路由转发的基础使用
一个带zuul网关面向服务转发的springcloud系统，至少需要包含这样三个模块：
eureka注册中心
eureka服务提供者
zuul网关
那么这里，结合之前的项目，我就还需要创建一个zuul网关服务，之后就可以进行测试。
zuul首先也需要引入相关的依赖，包含zuul本身的依赖，和eureka的依赖，eureka的依赖是为了实现面向服务转发的关键：
<pre>
&lt;dependency>
		&lt;groupId>org.springframework.cloud&lt;/groupId>
		&lt;artifactId>spring-cloud-starter-zuul&lt;/artifactId>
&lt;/dependency>
&lt;dependency>
		&lt;groupId>org.springframework.cloud&lt;/groupId>
		&lt;artifactId>spring-cloud-starter-eureka&lt;/artifactId>
&lt;/dependency>
</pre>

有了依赖之后，就是application.properties文件的配置：
<pre>
server.port=5000
spring.application.name=zuul-service
zuul.servlet-path=/
zuul.routes.zuul-service.path=/zuul/*
zuul.routes.zuul-service.serviceId=client
eureka.instance.prefer-ip-address=true
eureka.client.serviceUrl.defaultZone=http://127.0.0.1:10010/eureka/
</pre>

这里的重点在于后边五项，配置最后一项是因为网关服务本身也是一个服务，要被外部访问。倒数第二项之前说过，为了能够直接使用ip访问。
最关键的是中间三项：zuul.routes.zuul-service.path指定需要被过滤的请求uri，这里的意思就是凡是访问zuul这个项目的请求时/zuul开头的，都进行转发。转发到哪里呢？转发到zuul.routes.zuul-service.serviceId指定的服务，如果zuul.routes.zuul-service.serviceId指定的服务实际上有多个实例，那么就会由ribbon负责负载均衡选择。
最最重要的一项，是zuul.servlet-path，之所以说最最重要，是因为我参考的很多文章里都没有提这一项，而结果就是后边访问的时候直接抛出500错误：
<pre>
No route found for uri
</pre>

有了上边的配置后，还需要在启动类上加上开启网关的注解，当然了，还需要springcloud项目的标准注解：
<pre>
@EnableZuulProxy
@SpringCloudApplication
</pre>

至此，一个基本的网关服务就建好了，陆续启动注册中心、client服务和zuul网关项目，然后浏览器直接访问localhost:5000/zuul/hello，会发现返回“String”的字符串。
我们上边的zuul项目除了启动类的默认代码外，没有任何其他代码，之所以返回了数据，就是因为zuul系统对该请求进行了转发，内部又请求了client服务这个系统。

# zuul请求过滤
网关路由转发其实也是权限控制的一个环节，是在网络层面的控制，而权限控制通常都还会涉及到用户登录验证之类的功能。
在zuul里边就提供了请求过滤功能，可以使我们在这里脱离业务进行权限控制，从而使整个系统看起来耦合度更低。
zuul请求过滤其实理解起来比较简单，需要继承ZuulFilter类，这个类是个抽象类，实现了IZuulFilter接口。我们在继承这个类的时候，必须实现ZuulFilter和IZuulFilter里的四个抽象方法。
下边是代码示例：
<pre>
/**
 * zuul网关过滤器
 * @author tuzongxun
 * @create 2018年9月15日
 */
 @Component
public class FirstZuulFilter extends ZuulFilter {
    @Override
    public Object run() {
        // 具体权限控制的业务逻辑
        return null;
    }
    @Override
    public boolean shouldFilter() {
        return true;
    }
    @Override
    public int filterOrder() {
        return 0;
    }
    @Override
    public String filterType() {
        return "pre";
    }
}
</pre>

其中，run是主要逻辑代码的方法，比如进行一定的权限校验等，就是在这个方法里做。
shoouldFilter方法相当于一个开关，返回true的时候代表这个过滤器生效，返回false代表此过滤器不生效。我这里直接返回true，实际上可以做一定的逻辑处理。
filterOrder方法代表这个过滤器的执行顺序，当我们写了多个过滤器的时候，就可以给定不同的数值返回，数值越小的越先执行。还需要知道的是，在zuul内部有若干个内置的过滤器，有很多执行顺序值设为负数的，还有设置为1000的。
filterType方法指定过滤器的类型，可以理解为spring aop中before、after、around这种。filterType的值有四个：pre代表请求被路由之前执行，routing代表在路由请求时调用，post代表在routing和error之后被调用，error请求出错是被调用。