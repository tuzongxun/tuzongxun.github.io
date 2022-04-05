---
title: 使用spring-cloud-zuul-rate-limit在zuul中为服务限流
date: 2022-2-25 14:43:58
categories: 服务限流 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [服务限流]
---
## 前言
几个月前做过一个springCloud服务限流的任务，当时选定的技术是spring-cloud-zuul-rate-limit。
之所以选这个而不是别的，是因为项目本身使用了zuul，而且业务需求上spring-cloud-zuul-rate-limit的功能刚好都可以满足，在这种情况下就可以在符合要求的同时尽可能快的完成。

<!--more-->

这个任务实际上只相当于一个技术调研，只要在现有技术框架的基础上能够集成和实现需求，并且成功测试就可以了。
和当时在公司略有不同的是，公司是直接使用内部maven仓库，springcloud依赖的版本号基本都是原始的。
而在我自己电脑上重新写的时候，是使用的aliyun的maven仓库，springcloud版本经过了一定的加工处理。
两个电脑不互通，也不能互传数据，所以在自己电脑上就要手动重来一遍。
于是，因为这个版本问题，再加上自己的不小心，在自己电脑上一开始无法触发限流。
我一度以为就是版本不同导致的，结果绞尽脑汁后才发现是因为自己敲错了一个符号。

言归正传，这里的集成分为了三个简单的服务，一个模拟业务服务的server，一个注册中心eureka，还有一个springcloud zuul。以下是代码示例：

## spring cloud eureka server注册中心
### pom文件
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
......
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### 启动类
```
@EnableEurekaServer
@SpringBootApplication
public class CloudEurekaApplication {
   public static void main(String[] args) {
      SpringApplication.run(CloudEurekaApplication.class, args);
   }
}
```

### 配置文件
```
spring.application.name=eureka-server
server.port=1000
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://127.0.0.1:1000/eureka/
```

可以看到，这里其实就是一个极简的eureka的微服务注册中心，所以其实没有太多需要额外解释的内容。

## srping cloud eureka client业务服务
### pom文件
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
......
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 启动类
```
@EnableDiscoveryClient
@SpringBootApplication
public class CloudServer1Application {
   public static void main(String[] args) {
      SpringApplication.run(CloudServer1Application.class, args);
   }
}
```

### controller
```
@RestController
public class TestController {
   @GetMapping("/hello")
   public String hello(){
      return "hello";
   }
   @GetMapping("/hello1")
   public String hello1(){
      return "hello1";
   }
   @GetMapping("/hello2")
   public String hello2(){
      return "hello2";
   }
}
```

### 配置文件
```
server.port=1001
spring.application.name=hello-service
#name
eureka.client.service-url.defaultZone=http://127.0.0.1:1000/eureka
```

可以看出，上边也就是一个极简的springcloud eureka的客户端服务，加上了一些非常简单的api接口。

## springcloud zuul
springcloud zuul ratelimit实际支持很多种数据存储方式，比如redis、jpa、jcache等，由于项目中没有使用redis，也没有使用jpa，然后当时的电脑又不能随便安装软件，所以就只能选用jcache存储，结合bucket4j一起使用。
### pom文件
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
......
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.netflix.servo</groupId>
    <artifactId>servo-core</artifactId>
    <version>0.9.4-rc.1</version>
</dependency>
<dependency>
    <groupId>com.marcosbarbero.cloud</groupId>
    <artifactId>spring-cloud-zuul-ratelimit</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-core</artifactId>
    <version>6.2.0</version>
</dependency>
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-jcache</artifactId>
    <version>6.2.0</version>
</dependency>
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
    <version>1.1.1</version>
</dependency>
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-hazelcast</artifactId>
    <version>6.2.0</version>
</dependency>
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <version>4.2</version>
</dependency>
```

### 启动类
```
@EnableZuulProxy
@SpringCloudApplication
public class CloudZuulApplication {
   public static void main(String[] args) {
      SpringApplication.run(CloudZuulApplication.class, args);
   }
}
```

### 配置文件
```
server.port=5001
spring.application.name=zuul-service
zuul.routes.hello-service.path=/hello-service/**
zuul.routes.hello-service.service-id=hello-service
eureka.client.service-url.defaultZone=http://127.0.0.1:1000/eureka/

zuul.ratelimit.key-prefix=springcloud-book
zuul.ratelimit.enabled=true
zuul.ratelimit.repository=BUCKET4J_JCACHE

zuul.ratelimit.policy-list.hello-service[0].limit=2
zuul.ratelimit.policy-list.hello-service[0].quota=1
zuul.ratelimit.policy-list.hello-service[0].refresh-interval=20
zuul.ratelimit.policy-list.hello-service[0].type[0]=url
```

**这里配置含义如下：**
**zuul.ratelimit.key-prefix=springcloud-book**；缓存存储的自定义键前缀
**zuul.ratelimit.enabled=true**；启用速率限制，即启用ratelimit限流
**zuul.ratelimit.repository=BUCKET4J_JCACHE**；指定数据存储方式

下面的就是具体服务的限流规则配置，hello-service是服务id，尽量与eureka-client的服务名保持一致。
**zuul.ratelimit.policy-list.hello-service[0].limit=2**；每个刷新时间窗口对应的请求数限制；
**zuul.ratelimit.policy-list.hello-service[0].quota=1**；每单位时间允许访问的总时间（这个解释来自网上，需要进一步验证。目前多次测试的结果好像和网上解释的含义不太一样）
**zuul.ratelimit.policy-list.hello-service[0].refresh-interval=20**；刷新时间窗口的时间，单位是秒，默认60秒；
**zuul.ratelimit.policy-list.hello-service[0].type[0]=url**；支持的限流类型，如ORIGIN、USER、URL、URL_PATTERN等。

### cache配置类
```
@Configuration
public class HazelcastConfiguration {
   @Bean("RateLimit")
   Cache<String, GridBucketState> cache(){
      Config config=new Config();
      config.setLiteMember(false);
      CacheSimpleConfig cacheSimpleConfig=new CacheSimpleConfig();
      cacheSimpleConfig.setName("springcloud-book");
      config.addCacheConfig(cacheSimpleConfig);
      HazelcastInstance hazelcastInstance= Hazelcast.newHazelcastInstance(config);
      ICacheManager cacheManager=hazelcastInstance.getCacheManager();
      Cache<String,GridBucketState> cache=cacheManager.getCache("springcloud-book");
      return cache;
   }
}
```

### 过滤器
```
@Component
public class ErrorFilter extends ZuulFilter {
   @Override
   public String filterType() {
      return FilterConstants.ERROR_TYPE;
   }
   @Override
   public int filterOrder() {
      return 10;
   }
   @Override
   public boolean shouldFilter() {
      return true;
   }
   @Override
   public Object run()
      throws ZuulException
   {
      RequestContext context=RequestContext.getCurrentContext();
      Throwable throwable=context.getThrowable();
      if(throwable.getCause().getMessage().contains("ZuulException: 429 TOO_MANY_REQUESTS")){
         System.out.println("429 TOO MANY REQUESTS");
      }
      return null;
   }
}
```

## 功能验证
上边的服务中，定义了三个api，分别是/hello、/hello1和/hello2。主要验证limit、quota、refresh-interval、type的配置。
根据上边zuul的配置，目前规则是20秒的刷新时间窗口，20秒内只允许2次请求，否则会触发限流。由于上边配置的类型是url，所以这里更准确的说，是20秒内同一个url只允许2次请求，否则触发限流。
这里涉及limit、refresh-interval、type三个配置，由于多次验证quota都没效果，所以后续还需要进一步验证。

### 验证limit
首先，浏览器访问http://localhost::5001/hello-service/hello，20秒内发起多个请求，会发现前两个请求正常返回，后边的会出现429 TOO_MANY_REQUESTS。
然后，修改limit的值为4，重启zuul，然后浏览器再访问上边的url，20秒内发起多个请求，会看到前4次请求正常返回，后边的会出现429 TOO_MANY_REQUESTS。

**以上证明这个参数对限流是生效的。**

### 验证refresh-interval
接着上边的测试，修改refresh-interval为10，然后重启zuul。浏览器还是访问上边的url，保证20秒内超过4个请求，但是每10秒内都不超过4个，会看到都会正常返回结果。
然后，在10秒内发起超过4个的请求，会看到前4个正常返回，后边的就出现429 TOO_MANY_REQUESTS。

**以上证明refresh-interval也是生效的。**

### 验证type
为了提高效率，把上边limit改回2，refresh-interval改回20，然后重启zuul。
浏览器访问同一个url，保证20秒内超过2次，会看到不论是http://localhost::5001/hello-service/hello、http://localhost::5001/hello-service/hello1，还是http://localhost::5001/hello-service/hello2，前两次都正常返回，后边的就出现429 TOO_MANY_REQUESTS。

然后还是保证20秒内访问超过2次，但是三个url交叉访问，保证每个20秒内不会有同一个url超过两次，会看到全部都是正常返回。

**以上证明当type是url时，针对的就是同一个url。**

修改type为origin并重启zuul，然后20秒内同一个浏览器访问url超过2次，会看到不论是同一个url超过2次，还是不同的url超过2次，都是前两次正常返回，后边就出现429 TOO_MANY_REQUESTS。

### url_pattern
type是url其实已经基本能够满足大部分需求了，但是有时候可能就是需要对某一类url进行限流，而不是单纯的针对某一个url，那就可以用到url_pattern。
url_pattern的配置相对于url的稍微复杂一点，为了方便验证，我先增加一些api接口，使他们有一些有相同的部分，修改后如下：
```
@GetMapping("/hello")
public String hello(){
   return "hello";
}
@GetMapping("/hello1")
public String hello1(){
   return "hello1";
}
@GetMapping("/hello2")
public String hello2(){
   return "hello2";
}
@GetMapping("/test/hello3")
public String hello3(){
   return "hello3";
}
@GetMapping("/test/hello4")
public String hello4(){
   return "hello4";
}
@GetMapping("/test1/hello5")
public String hello5(){
   return "hello5";
}
@GetMapping("/test1/hello6")
public String hello6(){
   return "hello6";
}
```

然后再修改配置，如下：
```
zuul.ratelimit.policy-list.hello-service[0].limit=2
zuul.ratelimit.policy-list.hello-service[0].quota=1
zuul.ratelimit.policy-list.hello-service[0].refresh-interval=20
zuul.ratelimit.policy-list.hello-service[0].type[0]=url_pattern=/hello-service/test/*

zuul.ratelimit.policy-list.hello-service[0].limit=2
zuul.ratelimit.policy-list.hello-service[0].quota=1
zuul.ratelimit.policy-list.hello-service[0].refresh-interval=20
zuul.ratelimit.policy-list.hello-service[0].type[0]=url_pattern=/hello-service/test1/*

zuul.ratelimit.policy-list.hello-service[0].limit=2
zuul.ratelimit.policy-list.hello-service[0].quota=1
zuul.ratelimit.policy-list.hello-service[0].refresh-interval=20
zuul.ratelimit.policy-list.hello-service[0].type[0]=url_pattern=/hello-service/hello
```

重启zuul和client服务后，再次测试会看到，没有/test或者/test1开头的url以及/hello这个，无论访问多少次，都会是正常返回，而对于/test开头的，无论是同一个还是不同的url，只要达到限制条件，都会是429，/test1开头的也是一样。
和type为url时候一样，如果是在/test开头和/test1开头的url之间切换，保证相同前缀的url不触发限制，那么即使不同前缀的看起来超过了次数，最终也不会触发限流。
以上证明，type为url_pattern时，可以进一步把限流细化到某一个或者某一类url，可以对不同url进行不同的限制，不会一竿子打死，也能保证互不影响。

## 补充说明
文章开头提到，由于粗心大意，我一度认为是版本问题导致本机无法触发限流，实际只是因为一个符号敲错，那就是其他所有都是对的，但是zuul.ratelimit.enabled=true在最开始被我敲成了zuul-ratelimit.enabled=true。
这个地方实际我检查了不下5遍，但不知为何始终没有发现问题，直到放了差不多一个月重新再看时，才终于看到了问题所在。
这是个多么低级的错误，就如同以前遇到的数据库密码多一个末尾空格一样，很有吐血的冲动。