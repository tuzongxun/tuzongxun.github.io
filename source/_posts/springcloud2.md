---
title: springcloud微服务二：Eureka服务治理之服务注册中心
date: 2017-5-23 15:09:42
tags: [springcloud,java]
toc: true
categories: springcloud
---
# 理解
当初步的学习了spring boot，了解了spring boot的基本实现过程后，我就正式开始学习spring cloud，首先就从Eureka服务治理开始。

**服务治理包含三个核心的角色：服务注册中心、服务提供者和服务消费者**，他们相对独立，新的服务要向服务注册中心注册，新的消费者会向服务注册中心索引服务列表。

一番了解之后，让我想到了人才招聘。在我看来，现在普遍存在的招聘形式也是分为了三个部分：招聘网站或者人才市场、发布招聘需求的企业、需要找工作的人。当然了，也可以把企业和人换一下位置，那就是：招聘网站或者人才市场、找工作的人发布的简历、需要招聘的企业。
<!--more-->

那么这里的招聘网站或者人才市场就如同Eureka服务治理中的服务注册中心，不管是企业的招聘需求还是人的简历，都可以看做一种需求的同时看成一种服务。当有了一个这种新的服务或者需求后会发布在网站上或者人才市场，实际上就相当于Eureka服务治理中，新的服务通知注册到服务注册中心。而我们企业查找相关简历，或者找工作的人查找相关的招聘需求，就是客户端向注册中心索引服务。

在这个过程中，招聘网站、企业和找工作的人是各自独立的，由招聘网站把企业和人联系起来，前提就是企业和人知道网站的地址，这就如同在Eureka服务治理的服务提供者和服务消费者的application.properties文件中配置eureka.client.serviceUrl.defaultZone来指定服务注册中心。

# 使用
有了一点简单的理解之后，就是实际的项目构建，首先需要构建一个基础的spring boot项目作为服务注册中心，勾选web选项，这样会创建一个上一篇结构的项目。然后在pom.xml文件中加入Eureka的以来配置：

<pre>
&lt;dependency>
	&lt;groupId>org.springframework.cloud&lt;/groupId>
	&lt;artifactId>spring-cloud-starter-eureka-server&lt;/artifactId>
&lt;/dependency>
</pre>

以及如下的dependencyManagement，指定spring cloud的版本：

<pre>
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

这里**需要注意的是，spring cloud的版本号是以英国伦敦地铁站的名字命名的，按首字母A-Z的顺序排，不同的spring cloud版本相对应的spring boot的版本是有限制的，否则无法使用。**
具体的对应需要参考官网说明，目前比较新的spring boot版本是1.5.2，而对应的spring cloud的版本是Dalston，我这里例子中用的是1.3.8的spring boot，因此spring cloud版本是Brixton。

配置好依赖之后，下一步就是在创建项目时自动生成的那个类中加入@EnableEurekaServer注解，来启动一个服务注册中心，如下：

<pre>
@EnableEurekaServer
@SpringBootApplication
public class EurekaServer1Application {
	public static void main(String[] args) {
		SpringApplication.run(EurekaServer1Application.class, args);
	}
}
</pre>

然后便是在基本的配置文件application.properties中进行一些必要的配置：

<pre>
#应用名，服务名，如果不指定就是unkown
spring.application.name=eureka-server
#服务端口，实际上就是tomcat端口，浏览器访问或者其他地方调用时需要用的端口
server.port=1000
#实例的主机名
eureka.instance.hostname=server1
#设置是否向注册中心注册，默认是true
eureka.client.register-with-eureka=false
#是否需要去检索寻找服务，默认是true
eureka.client.fetch-registry=false
#指定注册中心
eureka.client.serviceUrl.defaultZone=http://server1:1000/eureka/
</pre>

就这样，一个简单的Eureka服务注册中心就完成了，除了引入依赖包的配置、properties的基本配置外，就是在代码中加了一个@EnableEurekaServer，非常的简单。
然后浏览其访问localhost:1000或者server1:1000（这里使用主机名需要配置ip和主机的映射）就可以看到如下界面：
![cloud1](/images/springcloud/s21.png)

如果有新的服务注册到服务注册中心，那么在上图中红框标注的地方就会有所显示，就比如如果我修改application.properties文件，去掉这一段配置：

<pre>
#设置是否向注册中心注册，默认是true
eureka.client.register-with-eureka=false
#是否需要去检索寻找服务，默认是true
eureka.client.fetch-registry=false
</pre>

那么该服务注册中心就会把自己也当做一个服务注册到这个注册中心中，那么就会如下图所示：
![cloud2](/images/springcloud/s22.png)
上边所实现的是单机版的服务注册中心，而微服务架构的九大特征之一就是高可用性，也就是需要注册中心从单机模式变为集群模式，使其中某个注册中心出现故障的情况下不影响整个业务系统的运行。
在spring cloud中，这个实现过程就十分的简单，简单到我们只需要在上述的过程中仅仅做一个很小的配置修改就可以了。
一是去掉eureka.client.register-with-eureka=false和eureka.client.fetch-registry=false的配置，使得注册中心可以自己把自己注册为服务，二是在指定注册中心的时候指定所有的注册中心，三是修改端口和主机实例名，修改后的配置如下：

<pre>
spring.application.name=eureka-server
server.port=1000
eureka.instance.hostname=server1
eureka.client.serviceUrl.defaultZone=http://server1:1000/eureka/,http://server2:2000/eureka/
</pre>

同样的，我们需要在另一个注册中心中把配置文件改为类似下边的样子：

<pre>
spring.application.name=eureka-server
server.port=2000
eureka.instance.hostname=server2
eureka.client.serviceUrl.defaultZone=http://server1:1000/eureka/,http://server2:2000/eureka/
</pre>

然后我们启动其中一个注册中心后再访问各自的页面，会看到如下所示：
![cloud3](/images/springcloud/s23.png)

这时候会看到我们的服务注册中心已经成为了集群模式，但是server2是unavailable的，并不可用，而当我们把另一个注册中心server2启动后，就会看到有些内容有了变化，server2也是可用的了，同样的，如果访问server2的页面，会看到这里列出server1的状态。
![cloud4](/images/springcloud/s24.png)

正常情况来说，两个注册中心是应该在不同的服务器中运行的，这样才能更好的实现高可用，所以就是两套代码。
但是在学习的过程中由于环境的限制，也为了方便，实际上就可以用一套代码，然后弄两个分别代表server1和server2的配置文件就可以了，文件命名需要遵循spring boot的约定规则，然后再application.properties文件中使用spring.profiles.active属性来指定运行的时候需要具体启用的篇配置就可以了，如图：
![cloud5](/images/springcloud/s25.png)