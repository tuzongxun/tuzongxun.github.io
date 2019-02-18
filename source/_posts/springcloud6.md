---
title: springcloud微服务六：声明式服务调用Feign
date: 2018-9-9 15:09:42
tags: springcloud
toc: true
categories: springcloud
---
在有了eureka服务注册中心、ribbon负载均衡及服务消费、hystrix断路器三部分之后，其实在某种程度上就应该能够创建一个基本的spring cloud微服务应用了，只不过这种应用无论是代码层面还是架构层面都还有一定的缺陷。
从代码层面来讲，ribbon和hystrix是一个标准化springcloud应用最基本的模块，通常也都是同时出现的，因此为了简化开发，有一个更高层次的工具来对他们进行了封装整合，这就是Feign。
feign在整合了这两者强大功能的同时，还提供了一种声明式的web服务客户端定义方式及可插拔式的注解，同时支持spring mvc的注解。
在ribbon和hystrix的基础上，feign看起来就比较简单。
<!--more-->

# 一、基本使用
## maven依赖
需要在eureka依赖的基础上再引入对feign的依赖：
<pre>
&lt;dependency>
		&lt;groupId>org.springframework.cloud&lt;/groupId>
		&lt;artifactId>spring-cloud-starter-feign&lt;/artifactId>
&lt;/dependency>
</pre>

application.properties基本配置
<pre>
服务名
spring.application.name=feign-client
端口
server.port=9099
允许ip访问
eureka.instance.prefer-ip-address=true
指定注册中心地址
eureka.client.serviceUrl.defaultZone=http://localhost:10010/eureka/
</pre>

启动类注解，在项目启动类加入如下三个注解，就可以使用feign：
<pre>
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
</pre>

在加入了上边注解的前提下，就可以编写feign声明式调用服务的接口：
<pre>
@FeignClient("client")
public interface FeignService {
    @RequestMapping("/hello")
    String sayHello();
}
</pre>

其中@FeignClient后的client即之前写好的eureka服务的服务名，接口里的方法定义和client里controller类中的定义一摸一样，区别就是没有具体的实现，也不需要具体的实现。
当我们在当前系统中调用FeignService .sayHello方法的时候，就会去服务名为client的服务中调用具体的同名同定义的方法。
如果服务名为client的有多个，那么由于feign内部整合的是ribbon，所以实际上是使用ribbon的默认负载均衡策略进行选择，也就是线性轮询。
如果有用过webservice和hessian这类远程调用服务技术的，应该会发现这里的写法和他们如出一辙。

# 二、优化
## 增加fallback
上边的代码就可以基本实现feign的功能，可以正常调用eureka了，但是和单独使用ribbon一样，当服务故障的时候，系统就会出现问题。所以一般@FeignClient这里可能会写成下边这样：
<pre>
@FeignClient(name = "client", fallback = ClientFallBack.class)
</pre>

name后边的client和上边的直接一个字符串声明等同，后边的ballback指定当服务出现故障时服务降级之后的处理类。
这个类是一个很简单的java类，需要继承FeignService接口，并实现接口中的方法。这个方法在有的地方被称作本地方法，当服务降级后就会调用这个方法里写的逻辑。
<pre>
public class ClientFallBack implements FeignService {
    @Override
    public String sayHello() {
        return "say hello";
    }
}
</pre>

## 捕获服务异常
上边的实现，可以一定程度上保证服务降级后继续一定的逻辑处理，但是如果需要在降级后的逻辑中捕获降级前的异常信息，就需要把fallback改成fallbackFactory，这个降级处理类就可以写成下边这样：
<pre>
@Component
public class ClientFallBackFactory implements FallbackFactory<FeignService> {
    @Override
    public FeignService create(Throwable cause) {
        return new FeignService() {
            @Override
            public String sayHello() {
                System.out.println(cause.getMessage());
                return "say hello";
            }
        };
    }
}
</pre>