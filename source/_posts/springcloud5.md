---
title: springcloud微服务五：客户端负载均衡ribbon
date: 2018-9-3 15:09:42
tags: [springcloud,java]
toc: true
categories: springcloud
---
# 基本概念理解
ribbon用在客户端，或者说消费端，被称作客户端负载均衡。
对于这个称呼，可以解析为三个部分，一个是客户端，一个是负载均衡，然后就是客户端负载均衡。
根据个人理解，所有发起请求的一端、去拿东西的一端，都可以称之为客户端或消费端。
而负载均衡，一般则是针对于至少两个以上的集群而言，重点在于均衡。从相对论的角度而言，一切都是相对的，那么也就不存在绝对的均衡，因此所谓的均衡就需要一定的标准，如何达到这个标准，则需要一定的控制，即负载均衡策略，也可以称之为规则。
对于客户端负载均衡，则可以很好的理解为，这是相对于服务端负载均衡而言。相对于服务端负载均衡而言，客户端负载均衡的好处就在于各个客户端自己实现负载均衡运算，从而降低了服务端的压力。坏处在于客户端要自己实现负载，就需要清楚服务端集群的状态，增加了客户端的复杂度以及多个客户端下可能看起来的功能冗余。
<!--more-->

# 基本用法
## pom.xml配置
ribbon的基本用法，实际上之前提过，首先还是一样需要创建一个springboot项目，然后配置maven的pom.xml文件
<pre>
&lt;parent>
	&lt;groupId>org.springframework.boot&lt;/groupId>
	&lt;artifactId>spring-boot-starter-parent&lt;/artifactId>
	&lt;version>1.5.10.RELEASE&lt;/version>
	&lt;relativePath/> 
&lt;/parent>
......
&lt;dependency>
	&lt;groupId>org.springframework.cloud&lt;/groupId>
	&lt;artifactId>spring-cloud-starter-eureka&lt;/artifactId>
&lt;/dependency>
&lt;dependency>
	&lt;groupId>org.springframework.cloud&lt;/groupId>
	&lt;artifactId>spring-cloud-starter-ribbon&lt;/artifactId>
&lt;/dependency>
......
&lt;dependencyManagement>
	&lt;dependencies>
		&lt;dependency>
			&lt;groupId>org.springframework.cloud&lt;/groupId>
			&lt;artifactId>spring-cloud-dependencies&lt;/artifactId>
			&lt;version>Edgware.SR2&lt;/version>
			&lt;type>pom&lt;/type>
			&lt;scope>import&lt;/scope>
		&lt;/dependency>
	&lt;/dependencies>
&lt;/dependencyManagement>
</pre>

**注：由于和之前四篇隔得时间较长，且目前工作实际项目使用的版本是springboot1.5.0和springcloud-Edgware.SR2，因此这之后的代码示例也均已这两个版本为主。**

## properties文件配置

<pre>
服务名
spring.application.name=ribbon-client
服务端口
server.port=9098
是否允许ip访问
eureka.instance.prefer-ip-address=true
指定注册中心的地址
eureka.client.serviceUrl.defaultZone=http://localhost:10010/eureka/
</pre>

## 基本代码
配置好pom.xml和properties文件后之后，在springcloud启动类类名上方加入springboot的启动注解以及springcloud客户端注解
<pre>
@EnableDiscoveryClient
@SpringBootApplication
</pre>

之后，在启动类中加入 RestTemplate的初始化代码，并使用@Bean注解告知springboot管理，同时加入@LoadBalanced注解，代表使用ribbon负载均衡来管理RestTemplate，这一段代码大致如下：
<pre>
@Bean
@LoadBalanced
RestTemplate restTemplate() {
    return new RestTemplate();
 }
</pre>

有了上边的代码，我们在其他被spring管理的组件中就可以使用@Autowired这样的方式注入RestTemplate，一个被ribbon管理的RestTemplate，如下边所示：
<pre>
@service
public interface RibbonService {
    @Autowired
    RestTemplate restTemplate;
    public String sayHello() {
        ResponseEntity<String> responseEntity = restTemplate.getForEntity("http://client/hello", String.class);
        return responseEntity.getBody();
    }}
</pre>

# 内置负载均衡策略
**RoundRobinRule线性轮询策略**
上边的简单实现，只使用了@LoadBalanced注解来开启ribbon负载均衡，那么就会使用ribbon默认的负载均衡策略，即线性轮询。
就如我们用酷狗等音乐播放器放歌一样，有很多方式，例如单曲循环、列表循环、随机播放，这里的线性轮询其实就是列表循环，放歌循环的是歌曲列表，线性轮询循环的是服务器列表，针对一系列的请求，从前到后依次访问列表中的服务器，到了最后边，就再回到开头。
虽然在多个服务端的情况下，线性轮询已经可以一定程度的实现负载均衡，大概也是实际使用率最高的一种方式，但是也有它的局限性，因此ribbon还内置了其他一些负载均衡策略供我们选择。

我们目前项目中就只使用了默认的负载均衡策略，不过也对其他策略有过一定的了解，下边几个是我个人认为可能也会用到的：
**RandomRule随机选择策略**
随机二字应该就不需要多做解释了；
**RetryRule线性轮询重试策略**
这个策略内部实际上还是用的线性轮询，与默认的线性轮询不一样的地方在于，这个策略在访问轮询到的服务有问题时，会在一定的时间内进行重试，这个时间可以通过maxRetryMillis参数进行配置。
**WeightedResponseTimeRule权重响应时间策略**
这个策略是对线性轮询的拓展，在线性轮询的情况下，不论服务器实例的性能如何，在一定的请求次数下都必然会被轮询到，这样一来，如果某个实例由于某些原因，响应时间远大于其他实例，就必然影响整体性能。
而WeightedResponseTimeRule则会对每个服务实例的响应时间进行一定的记录和计算，然后给每个实例一个权重值区间。
如果上边的权重值最大值小于0.001，那么之后的请求会采用线性轮询策略；如果大于0.001，那么对于每一个请求，会随机分配一个0到最大权重值之间的随机数，然后这个随机数在实力列表中哪个实例的权重区间内，就会选择哪个实例。
那么这里有一个细节，就是随机数算法中含头不含尾，也就是说在0到最大权重值之间的随机数，可以是0，却不会是最大权重值那个值。
另一个细节就是，根据这里的权重算法，平均响应时间越短的服务实例，它的权重区间就会越大，注意，是权重区间越大，而不是权重值。再明确点说，就是这个实力区间内能容纳的随机数个数更多。
因此，在生成随机数的时候，这个区间命中的概率就越大，这个实例被选择到的几率也更高，从而就使得平均响应时间短的实例更容易被选择到，就提升了系统效率。

除了上边四种，还有诸如**最好可用策略BestAvailableRule**等策略，就不再记录。
同时，**还可以自己定义负载均衡策略**，但是应该很少会被用到，因为内置的这些已经能够满足非常多的应用场景了。

那么，了解了不同的负载均衡策略之后，还需要知道的当然就是如何使用非默认的负载均衡策略了。
其实也比较简单，只需要在加一个让spring来管理的bean就好，如下边这样：
<pre>
@Bean
public IRule ribbonRule() {
    // return new RandomRule();
    // return new RetryRule();
    return new WeightedResponseTimeRule();
 }
</pre>
