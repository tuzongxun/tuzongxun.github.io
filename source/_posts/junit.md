---
title: 我以为我会junit，原来我还不会
date: 2022-2-28 19:43:58
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [java,junit]
---
`junit`从某种程度上来说应该是很简单的一项技术，但是正所谓会者不难，难者不会，如果没有好好地用过，总会有些你以为是对的地方，其实他是错的。
对于有7年java开发经验的我来说，不完全会写junit，实在是汗颜。
以前的项目基本都没怎么要求写junit，所以我一直误以为junit简单到就是在Test中调一下相关方法，只要跑出绿色结果就好了，直到这一次需要相对正式的junit时，才发现这样是不对的，以下是从错到对的一个记录。
<!--more-->
## 业务功能代码
例如我这里有一个很简单的`Service`，代码如下：
```
/**
 * @Author tuzongxun
 * @Date 2021/9/1
 */
@Service
public class TestService {

   public String hello(String name){
      if("张三".equals(name)){
         System.out.println("名字："+name);
         return "你好，张三";
      }
      System.out.println("name:"+name);
      return "hello,welcome:"+name;
   }
}
```

## 错误的junit
按照以往的认知，我写的junit是这样的：
```
/**
 * @Author tuzongxun
 * @Date 2021/9/7
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class ServiceTest {

   @Autowired
   private TestService testService;

   @Test
   public void helloZhangsanTest() {
      String res = testService.hello("张三");
      System.out.println(res);
   }
}
```

## 错误的认知
上边的junit是可以运行的，运行之后的结果状态是绿色，显示`Tests passed`，如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/903f8b0068e443f8be0e3c5e81f02b5f.png#pic_center)

因为没有报错，输出也是passed，所以我理所当然的以为这就是正确的junit了，这样就可以完美的上线了。
然而，并不是。

## 错在哪儿，怎么解决
上边的junit错在哪儿呢？起码有这样两个：
1. 只是调用了相关代码，而没有预期的结果；
2. 功能代码覆盖不足；

### 断言assert
怎么理解呢，首先第一个，我这里调用了hello方法并打印了返回值，我心里相信返回的结果肯定是我要的。
但是这里有一个误区，那就是junit的作用是做什么的。junit在我理解，某种程度上就是不相信你相信的，你心里的相信无效，所以才要代码进行验证。
因此这个问题的解决方式实际就是加入断言`assert`，例如：
```
@Test
public void helloZhangsanTest() {
   String res = testService.hello("张三");
   System.out.println(res);
   Assert.assertEquals("预期结果不一样","你好，张三",res);
}
```
那么当我们再执行test时，其实会发现结果和开始是一样的。
但是，如果稍微改一下代码，假如在service中我手误把`return "你好，张三";`打成了`return "你好，张三三";`，那么当再执行上边的test方法时就会报错，像这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/5d4af9ac0a5945b19419f88118c2dee5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


那么，当我去掉断言那行代码，重新执行test方法，会发现这时候执行结果还是`Tests passed`，但是很显然那个service中的代码是有问题的，而我的junit根本没能测试出来，也就更加证明没有加断言预期结果的junit其实是无效的junit。

### 功能覆盖率
对于功能覆盖率问题，可以看到在hello方法中，有效代码是6行：
```
if("张三".equals(name)){
   System.out.println("名字："+name);
   return "你好，张三三";
}
System.out.println("name:"+name);
return "hello,welcome:"+name;
```

而我的junit中因为还如的参数是`张三`,所以实际上只会走前边的几行，也就是：
```
if("张三".equals(name)){
   System.out.println("名字："+name);
   return "你好，张三三";
}
```

那么还有剩下的两行是不会走的，也就是说这个junit中并没有覆盖到足够的代码，没有覆盖到，就不能保证没有问题，就跟上边没有加断言一样。
所以这里解决办法呢，就是要在junit中加入另一种情况，很显然，在我这里需要加入`不是张三`的，例如我再加一个方法，然后整个`ServiceTest`类就成了这样:
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class ServiceTest {

   @Autowired
   private TestService testService;

   @Test
   public void helloZhangsanTest() {
      String res = testService.hello("张三");
      System.out.println(res);
      Assert.assertEquals("预期结果不一样","你好，张三",res);
   }

   @Test
   public void helloNotZhangsanTest() {
      String res = testService.hello("lisi");
      System.out.println(res);
      Assert.assertEquals("预期结果不一样","hello,welcome:lisi",res);
   }
}
```

## junit常用注解
之所以说junit从某种程度来说很简单，就是因为基本都是借助一些注解和方法，都还是比较好理解的，除了上边出现的`@Test`外，还有例如`@Before`、`@After`、`@BeforeClass`、`@AfterClass`以及`@Ignore`、`@Parameters`等等，以下是其中一些示例：
```
@BeforeClass
public void beforeClass(){
   System.out.println("执行这个类的test前会执行一次");
}

@Before
public void before(){
   System.out.println("每个test执行前都要执行我");
}

@After
public void after(){
   System.out.println("每个test执行完都要执行我");
}

@AfterClass
public void afterClass(){
   System.out.println("执行这个类的所有需要执行的test之后会执行一次");
}
```

## idea junit代码覆盖率工具
上边说了junit代码覆盖率问题，如果是用idea的话，实际上有工具可以帮助我们分析识别。
在相应的测试类上右键点击，例如我这里的TestService，会出现`Run “ServiceTest” with Coverage`这样一个选项，左键点击执行，运行完以后就会出现这样一个界面：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ebc9ba42a0db4f79934ed8a08536de29.png#pic_center)

很显然，这里是我整个项目的内容，可以从`Element`下边点击进到需要看的类，例如我点击`Service`后就会出现下边这样的界面：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4ac36cb601b846a59ff2869108d448e5.png#pic_center)

上边的结果是Class覆盖率100%，Method的覆盖率100%，代码行的覆盖率也是100%。
那么如果像最开始一样，只有一个参数是`张三`的测试方法，例如：
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class ServiceTest {

   @Autowired
   private TestService testService;

   @Test
   public void helloZhangsanTest() {
      String res = testService.hello("张三");
      System.out.println(res);
   }
}
```
再运行`Run “ServiceTest” with Coverage`，就会看到结果成了这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1797b56bd35e4f7aa1653c40e1025bd8.png#pic_center)

因为我这里就是一个类，一个方法，所以Class和Method都还是100%，但是Line，也就是代码行的覆盖率就只有66%了，而且后边也明确的说了，6行只覆盖了四行。
那么这里有人可能就有疑问，到底是哪四行覆盖了，那两行没覆盖呢？
像这里代码比较简单，其实上边也说了是拿四行覆盖了，哪两行没覆盖。但是这里只是人工分析的，如果代码很多，就没有那么容易分析了。
那么实际上是可以直接看的，运行了上边的`Run “ServiceTest” with Coverage`之后，可以进入到实际业务功能代码中，例如我这里就是`TestService`的`hello`方法，进去之后会看到类似这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/428d765ddd3347d5ad160b1745ea6de1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

这里可以看到，13-15行后边有一条绿色的矩形，而17、18行则是红色的矩形，实际上这里的绿色就代表被覆盖了，而红色的代表没有被覆盖。
那么还有点小问题就是，这里绿色的3行，红色的2行，一共才5行，而实际代码有6行。
这是因为16行的括号和if方法是一体的，所以跑了if的内容后这个括号就也会被算作覆盖了。

## 后记
junit只是一个工具，那些注解以及各种辅助方法在需要的时候试一下，大概就会知道怎么用了。
再借助覆盖率分析工具，也能更好的进行junit的编写，可能很多公司不严格要求这个，但是一般有sonar扫描的公司多半都是需要这个的。
作为一个7年经验的开发，不会正确的写junit，确实是有点说不过去。
但是话说回来，对于一个7年经验还不会正确的写junit的人，团队领导和队友们都还能包容，不得不说我确实是很幸运的。
有团队如此，夫复何求。
感谢我的领导，感谢我的队友！