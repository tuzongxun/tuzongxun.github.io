---
title:  一篇文章学会spring boot（包括jms和hessian的集成）
date: 2017-07-30 15:57:44
tags: springboot
categories: springboot #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
---
之前在学习spring cloud微服务的时候，由于spring cloud的基础是spring boot，因此曾简单地了解过spring boot，但也只是简单的了解过而已。 
而现在，需要把struts2项目改为spring boot，一开始时以为是整个项目重构，不仅限于struts2部分，因此就相对更系统、更细致的学了一下spring boot。 
整个过程由易到难，大概分成了这么些模块：
<!--more-->
## 创建简单的spring boot web项目 
很多时候学一个新的东西，都需要从最简单的地方开始，然后让自己看到成功后的效果，这样才能更有信心继续下去，至于这第一步，我在写spring cloud相关学习记录的时候，第一篇就是spring boot，因此便不需要再写一遍，那一篇博客是： 
springcloud微服务一：spring boot基础项目搭建及问题处理

## 配置文件加载类 
虽然说spring boot原则上是建议去掉xml配置文件，但是对于一个已经成型并上线运行着的项目而言，不论是代码还是配置都是非常庞大而且复杂的。 
这些配置文件不只是spring的、struts2的，还有spring和mq集成的，还有spring和hessian集成的等等。 
仅涉及到spring和struts2的配置文件比较好改，而spring和mq以及hessian集成的就稍微有些麻烦。 
因此在一开始我心中便有了一个设想，就是mq集成和hessian集成的部分先还是以配置文件的方式不变，等简单易改的部分搞定后再说，于是便需要知道在没有了web.xml文件的spring boot项目中，我需要如何引入加载这些暂时不能去掉的xml文件。 
经了解之后发现其实还是很简单的，只需要自己再写一个空的java类，然后使用两个注解就好：
<pre>
@Configuration
@ImportResource(value = "classpath:spring.xml")
public class ConfigClass {
}
</pre>

`@Configuration`是向spring声明这是个配置类，`@ImportResource`则是指明需要引入的配置文件路径。

## 使用注解给属性赋值 
在过去的项目中，有一些数据存放在properties文件中，然后在spring的xml文件中引用，从而给特定的对象的属性赋值。 
而如果整个项目改成spring boot，就需要尽可能去掉能去掉的xml配置文件，就没有了这种注入赋值，所以还需要知道怎样在spring boot中达到xml文件配置一样的赋值效果。 
经了解后发现也是比较简单地，只需要在对应的属性上使用@Value注解：
<pre>
public class User {
    @Value("${tuser.name}")
    private String name;
    @Value("${tuser.age}")
    private int name;
}
</pre>

当然了，这种写法的前提是在properties文件中有相应的属性配置，例如：
<pre>
tuser.name=testtttttt
tuser.age=123
</pre>

并且spring boot项目启动后能够加载到这个properties文件，spring boot默认会加载resources目录下的application.properties文件，如果我们这些配置不在这个文件中，还需要进行其他的配置，例如在application.properties文件中使用spring.profiles.active进行指定。

## 使用注解给对象赋值： 
虽然说在java中有一切皆对象的说法，属性和属性所属对象都统称对象，但是这里为了区分，我只能在上边更精确的说到属性。 
而上边那种赋值方式有个很明显的缺陷，就是每个属性都需要使用@Value注解，如果这个类恰好很大，有几十个属性，那么这种赋值将是一个很让人烦躁的工作。 
而spring boot中也正好提供了一个更加方便的方式，只需要一个注解，就可以引用配置文件中的内容给整个对象赋值。 
例如配置文件中有如下内容：
<pre>
mytest:
  user:
    name: test
    age: 22
    phone: 13533559797
</pre>

上边这种写法是yaml文件的写法，也就是在spring boot中默认识别properties类型的配置文件和yaml的配置文件。 
上边的写法实际上等同于properties文件中如下的写法：
<pre>
mytest.user.name=test
mytest.user.age=22
mytest.user.phone=13533559797
</pre>

而我们要把这样一些配置直接赋值给特定对象的方式，就是使用@ConfigurationProperties注解：
<pre>
@ConfigurationProperties(prefix = "mytest.user")
public class User {
    // @Value("${test.user.name}")
    private String name;
    // @Value("${age}")
    private int age;
    // @Value("${phone}")
    private String phone;
}
</pre>

类上边的那个注解等同于下边被注释掉的三个注解，当然了，不论哪种方式，实现的前提都是这个user类能够被spring 扫描到，也就是说能够被spring所管理。

## filter过滤器 
在原本的spring web项目中，web.xml文件是必不可少的，启动tomcat后就会来加载这个文件。 
通常我们都会在这个文件中配置一些servlet以及过滤器，例如字符集过滤器，而spring boot不再使用web.xml，就需要我们能够用其他方式解决原本web.xml中的过滤器的问题。 
这里写一个过滤器也很简单，要实现javax.servlet.Filter接口，并借助几个注解，只不过这个功能应该是java自己的，而不是spring boot新提供的：
<pre>
/**
*@auth tuzongxun
*/
@WebFilter(filterName = "myFilter", urlPatterns = "/*")
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("进入filter");
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
        chain.doFilter(request, response);
    }
    @Override
    public void destroy() {
        System.out.println("走出filter");
    }
}
</pre>

## listener监听器 
说到了过滤器，接下来自然要说的就是监听器，在java web项目中，过滤器和监听器都用的很多，也都常常在web.xml文件中有相应的配置。 
在spring boot中写一个监听器就更加简单，比如就仅仅是下边的几行代码就够了：
<pre>
@WebListener
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("contextlistener init");
    }
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("contextlistener destroy");
    }
}
</pre>

同样的，上边的类所实现的接口以及相应的注解也都不是spring boot的，而是javax.servlet的。 
上边例子中我们写的是一个contextListener，同样的也可以写一个requestListener以及其他的listener，例如：
<pre>
@WebListener
public class MyListener1 implements ServletRequestListener {
    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        System.out.println("requestlistener destroy");
    }
    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        System.out.println("requestlistener init");
    }
}
</pre>

## ERROR错误页面 
在web.xml中除了配置了filter和listener外，比较常见的就还有配置error错误页面，我们项目中也同样这么做了，因此在抛弃web.xml的时候，也必然要考虑error页面的问题，那么在spring boot中也同样是非常的简单，比如写这样一个类：
<pre>
@Component
public class Test {
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer() {
return new EmbeddedServletContainerCustomizer() {
@Override
public void customize(ConfigurableEmbeddedServletContainer container) {
  ErrorPage error404Page = new ErrorPage(HttpStatus.NOT_FOUND, "/404.html");
  ErrorPage error500Page = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500.html");
  container.addErrorPages(error404Page, error500Page);
            }
        };
    }
}
</pre>

需要注意的是，我这种写法是jdk1.6可用的，还有一种写法需要jdk1.7以上版本才行，这里就不举例了。 
并且我这里是写了一个新的类，我们也可以直接把具体的方法写到启动类那里，效果是一样的，只要都能被spring 扫描到。

## 定时任务 
原本项目中使用了quartz定时任务，使用了大量的xml配置文件，这里就使用schedule替代。 
不过，这一条虽然列在了这里，但是实际上我觉得可以不用列出来，因为这里使用schedule代替quartz，实际上也是spring原本就有的，并不是有了spring boot之后才出现，具体的我也曾写过相应的博客，例如： 
spring schedule定时任务（一）：注解的方式 
只不过，在spring boot中更加的简单，不需要我们再做任何的动作，只需要这个简单的类和注解就ok了：
<pre>
@Component
@EnableScheduling
public class MySchedule {
    @Scheduled(cron = "*/2 * * * * ?")
    public void sTest() {
        System.out.println(Calendar.getInstance().getTime());
    }
}
</pre>

## 返回页面 
返回页面，在spring boot中叫引用模板，需要特定的依赖，例如：
<pre>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
</pre>

当然了，这只是其中一种。 
引入了相应的依赖后，我们在spring boot项目的resources目录下的templates目录下建立具体的html页面，然后具体的controller中只需要返回相应的html的文件名，即能成功返回页面。 
例如templates下有index.html文件，在controller中就可以写成这样：
<pre>
@RequestMapping("/index")
public String index(ModelMap map) {
        map.put("name", "testfsdfdsfdsfsdf");
        return "index";
}
</pre>

不过这里我觉得应该也可以使用modelandview，但并没有尝试，不知是否可行，而且即便可行，在性能上如何也还未曾测试。

## spring boot 集成activemq，即jms 
前文中我有说到我们原本的项目中使用了spring 整合activemq，那么改为spring boot后不可能就把activemq扔掉，还需要找到可行的集成方案，可喜的是，spring boot中有jms专门集成activemq。 
要实现这种集成，我们首先还需要导入jms相应的依赖，创建项目的时候选择jms，会在pom.xml文件中生成如下内容：
<pre>
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
</pre>

使用activemq发送消息需要有消息队列，因此接下来就需要我们生成一个消息队列的配置类，用来指定queue：
<pre>
@Configuration
@EnableJms
public class ActiveMQConfig {
    @Bean // 配置一个消息队列
    public Queue queue() {
        return new ActiveMQQueue("sample.queue");
    }
}
</pre>

有了队列以后，然后就是发送消息：
<pre>
@Service
public class MQProduceService {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    @Autowired
    private Queue queue;
    public void send(String msg) {// 向指定队列中发送消息
        this.jmsMessagingTemplate.convertAndSend(this.queue, msg);
    }
}
</pre>

就这样简单两步，我们的activemq消息服务器就算是写好了，如果再其他地方调用这个service中对应的方法，就可以实现消息的发送。 
有了发送端，还需要有接受端，或者说消费端。也是需要先配置队列，因为我这里都写在一个项目中，所以就公用同一个配置就好，具体的消费端代码如下：
<pre>
@Service
public class MQConsumerService {
    private String text;
    @JmsListener(destination = "sample.queue") // 监听指定消息队列
    public void receiveQueue(String text) {
        this.text = text;
        System.out.println(text);
    }
    public String receive() {
        return text;
    }
}
</pre>

不论是发送端还是消费端，代码都很简单，为了更直观的看到结果，我编写了一个controller来调用相应的服务实现发送和接受，代码如下：
<pre>
@RestController
public class MQController {
    @Autowired
    private MQProduceService produceService;
    @Autowired
    private MQConsumerService consumerService;
    @RequestMapping("/send")
    public String send() {
        produceService.send("this is an activemq message");
        return "send";
    }
    @RequestMapping("/receive")
    public String receive() {
        String str = consumerService.receive();
        return str;
    }
}
</pre>

这个代码非常的常规，也就不做多的解释了。

## spring boot中使用hessian 
mq分为发送端和消费端，而hessian作为一种webservice，也是一样具有服务端和消费端，不过与mq不同的是，hessian需要在web.xml进行一定的配置。 
与之前hessian的实现方式一样的是，服务端需要有相应的接口类和实现类，例如：
<pre>
public interface HelloWorld {
    public String hello();
    public Map<String, String> helloMap();
}
</pre>

<pre>
@Service("helloWorld")
public class HelloWorldImpl implements HelloWorld {
    private static final long serialVersionUID = 1L;
    @Override
    public String hello() {
        return "hello hessian";
    }
    @Override
    public Map<String, String> helloMap() {
        Map<String, String> map = new HashMap<String, String>();
        map.put("message", "hello word!!!!!");
        return map;
    }
}
</pre>

而不同的是，之前使用web.xml时在web.xml中进行了配置，而在这里则是使用了一个类来暴露接口：
<pre>
@Component
public class HService {
    @Autowired
    private HelloWorld helloWorld;
    @Bean(name = "/springBootDemo/HelloService")
    public HessianServiceExporter exportHelloService() {
        HessianServiceExporter exporter = new HessianServiceExporter();
        exporter.setService(helloWorld);
        exporter.setServiceInterface(HelloWorld.class);
        return exporter;
    }
}
</pre>

就这样，在不使用web.xml的情况下我们也一样实现了spring boot和hessian服务端的集成，或者更准确的说，是spring和hessian服务端的集成。 
而至于客户端的集成，目前我还没有整出可行的方案，因此便只能继续先使用之前xml文件配置的方式，引入这个xml文件然后进行后续的操作，具体的不用xml的方式还需要进一步探索，如果有朋友用过，欢迎指点和交流！