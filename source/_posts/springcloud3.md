---
title: springcloud微服务三：Eureka服务治理之注册服务提供者及服务的发现和消费
date: 2017-5-24 15:09:42
tags: springcloud
toc: true
categories: springcloud
---
# 说明
当服务注册中心成功建立以后，就需要有服务的注册和消费，否则这个服务注册中心就没有了存在的意义，而实际上，一个简单的服务注册也是非常简单的，仅仅需要实现四部曲就好。

# 服务注册
## 步骤一
首先，还是建立一个基本的spring boot的项目，创建的时候选择web，然后在pom.xml文件中加入Eureka的依赖，和服务注册中心的maven配置一样，这里就不再罗列代码。

## 步骤二
第二步，就是修改application.properties文件，指定端口和服务名称以及注册中心的地址：

<pre>
server.port=1001
spring.application.name=hello-service
eureka.client.serviceUrl.defaultZone=http://server1:1000/eureka/,http://server2:2000/eureka/
</pre>

因为上一步中实现了注册中心的集群模式，因此这里可以指定两个，如果有更多的服务注册中心，还可以指定更多，中间以逗号分隔。

## 步骤三
第三步，使用注解开启自动装配，使服务能够向注册中心注册、向注册中心服务租约（实际上就是定时器持续发送心跳告诉服务注册中心自己还活着）、取消租约、查询服务列表等，这个注解是@EnableDiscoveryClient，添加以后的代码如下：

<pre>
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaService1Application {
	public static void main(String[] args) {
		SpringApplication.run(EurekaService1Application.class, args);
	}
}
</pre>

## 步骤四
第四步，我们说要进行服务注册，当上边一切准备工作做好以后，自然需要有服务才能注册，而这个服务就是一个看起来很普通的controller类，如图：

<pre>
@RestController
public class HelloController {  
	@Autowired
	private DiscoveryClient client;
	@RequestMapping(value="/hello",method=RequestMethod.GET)
	public String sayHello(){
		ServiceInstance instance=client.getLocalServiceInstance();
		return "hello "+instance.getHost()+","+instance.getPort()+","+instance.getServiceId();
	}
}
</pre>

这里的DiscoveryClient可以获取注册到服务注册中心的服务的相关信息，并不是必须注入的，这里完全可以写成如下所示的代码：

<pre>
@RestController
public class HelloController {  
	@RequestMapping(value="/hello",method=RequestMethod.GET)
	public String sayHello(){
		return "hello";
	}
}
</pre>

当然了，因为只是了解服务注册的过程，所以这里仅仅这么简单，实际的业务逻辑自然不会是这样。

那么通过上边四步，一个能够注册到服务注册中心提供服务的服务就完成了，启动这个服务就会被注册到服务注册中心，我们可以通过服务注册中心的页面来查看：
![cloud1](/images/springcloud/s31.png)

成功的创建了一个服务并注册到注册中心，同样的，为了高可用，我们可以再建一个具有同样功能的服务，而这个过程就更加的简单，只需要更改一下端口号，也就是server.port，然后重新启动就可以，启动后会看到上边的页面如下（当然了，这里也是为了测试方便，实际情况也应该是在两台机器上运行的两套代码才对）：
![cloud2](/images/springcloud/s32.png)

这里我们会看到已经有了两个同应用名的服务，端口分别是1001和2001。
而在上述页面中还需要注意的是那一段醒目的红色提示，它的意思就是说开启了自我保护机制，也就是说当服务失效后，有可能注册中心还会保留一段时间该服务的信息，这样就可能导致服务消费方获取服务列表的时候依然能够获取到这个服务，但是实际上向该服务发送请求的时候又无法成功请求，那么学习的过程中可以使用以下配置关闭自我保护机制，然后如果再访问注册中心的页面便不会再出现这个提示，也不会出现那种服务失效后还保留在注册中心的情况。

<pre>
#设置是否开启自我保护机制，默认是true
eureka.server.enable-self-preservation=false
</pre>

# 服务发现
服务注册好以后，下边就是服务的发现和消费，通常有ribbon和feign两种方式，而feign实际上也是以ribbon为基础的，这两种后边都还需要具体详细的学习，所以这里只简单的演示一下消费的过程，不做太详细的解释。ribbon可以实现负载均衡，这里使用最简单的模板实现。
首先还是建立一个基础spring boot项目，然后pom.xml中加入eureka和ribbon的依赖以及eureka的版本信息：

<pre>
&lt;dependency>
	&lt;groupId>org.springframework.cloud&lt;/groupId>
	&lt;artifactId>spring-cloud-starter-eureka&lt;/artifactId>
&lt;/dependency>		
&lt;dependency>
	&lt;groupId>org.springframework.cloud&lt;/groupId>
	&lt;artifactId>spring-cloud-starter-ribbon&lt;/artifactId>
&lt;/dependency>
&lt;dependencyManagement>
	    &lt;dependencies>
	        &lt;dependency>
	           &lt;groupId>org.springframework.cloud&lt;/groupId>
	           &lt;artifactId>spring-cloud-dependencies&lt;/artifactId>
	           &lt;version>Brixton.RELEASE&lt;/version>
	           &lt;type>pom&lt;/type>
	           &lt;scope>import&lt;/scope>
	        &lt;/dependency>
	    &lt;/dependencies>
&lt;/dependencyManagement>
</pre>

然后application.properties文件内容如下：

<pre>
server.port=3001
spring.application.name=rabbit-client
eureka.client.serviceUrl.defaultZone=http://server1:1000/eureka/,http://server2:2000/eureka/
</pre>

再然后就是创建项目时生成的类中加入@EnableDiscoveryClient参数开启自动装配，以及注入服务消费模板并开启负载均衡：

<pre>
@Bean
@LoadBalanced
RestTemplate restTemplate(){
	return new RestTemplate();
}
</pre>

之后的这个步骤实际不是必要的，只是为了能够更直观的在页面中看到结果，所以加入一个controller类，输出服务的一些基本信息：

<pre>
@RestController
public class ConsumerController {	
	@Autowired
	RestTemplate restTemplate;
	@RequestMapping(value="/hello",method=RequestMethod.GET)
	public String hello(){
		return restTemplate.getForEntity("http://HELLO-SERVICE/hello", String.class).getBody();
	}
}
</pre>

那么当我们启动这个消费端以后，页面访问localhost:3001/hello就可以看到如下页面：
![cloud3](/images/springcloud/s33.png)
