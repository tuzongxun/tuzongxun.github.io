---
title: 前后台打通之springboot后台要点记录
date: 2020-3-5 18:09:42
tags: [业余项目,tzxblog]
toc: true
categories: 业余项目
---
接着之前的vue前端项目搭建成功，在实现了一些基本的页面功能之后，现在再搭建一下基础的后台，从而实现前后台打通，以下是整个打通过程中的一些关键点及感悟记录：

springboot的后台项目搭建比较简单，如果单纯的实现接口的话，实际上没什么好说的，但是为了看起来不那么单调，就先暂时集成了一些非常基础的组件，例如logback、mybatis、lombok等。

<!--more-->

## mybatis关联查询和特定标签使用
springboot操作mysql实际有更简单的方案，例如jpa，但是考虑到这个项目是为了巩固工作中的技术，并期望在实现的过程中发现更多知识盲点，所以还是选用mybatis-plus。
mybatis-plus基本和mybatis是一样的，常规操作也都差不多，并且目前涉及的业务也都不是太复杂，所以这里也仅是对关联查询进行一个记录。
例如系统需要查询博客列表，而博客列表除了博客本身信息外，还需要基本的用户信息以及访问量等信息，所以这里其实就涉及到了另外两个信息的关联查询，除非选择在后台代码中循环遍历查询，这肯定是不可取的。
对于关联查询，主要是返回结果的关联映射，时间长了不写可能就忘记细节，关键在于association 标签的使用，如下：

```
<resultMap id="blogResultMap"
		type="com.tzx.blog.entity.BlogInfo">
		<result column="id" jdbcType="VARCHAR" property="id" />
		<result column="title" jdbcType="VARCHAR" property="title" />
		<result column="desc" jdbcType="VARCHAR" property="desc" />
		<result column="content" jdbcType="VARCHAR" property="content" />
		<result column="type" jdbcType="VARCHAR" property="type" />
		<result column="cate_id" jdbcType="VARCHAR" property="cateId" />
		<result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
		<result column="update_time" jdbcType="TIMESTAMP" property="updateTime" />
		<association property="userInfo" javaType="com.tzx.blog.entity.UserInfo">
            		<result property="id" jdbcType="VARCHAR" column="id"/>
            		<result property="name" jdbcType="VARCHAR" column="name"/>
            		<result property="img" jdbcType="VARCHAR" column="img"/>
        	</association>
       		<association property="blogDetailInfo" javaType="com.tzx.blog.entity.BlogDetailInfo">
            		<result property="id" jdbcType="VARCHAR" column="id"/>
            		<result property="blogId" jdbcType="VARCHAR" column="blog_id"/>
            		<result property="fabulousCount" jdbcType="VARCHAR" column="fabulous_count"/>
           		 <result property="readCount" jdbcType="VARCHAR" column="read_count"/>
            		<result property="commentCount" jdbcType="VARCHAR" column="comment_count"/>
           		<result property="forwardCount" jdbcType="VARCHAR" column="forward_count"/>
            		<result property="cateType" jdbcType="VARCHAR" column="cate_type"/>
            		<result property="power" jdbcType="VARCHAR" column="power"/>
            		<result property="blogType" jdbcType="VARCHAR" column="blog_type"/>
        	</association>
</resultMap>
```

## 跨域问题
使用vue和springboot前后端分离模式，前后台都要独立部署，这也就必然不可能在一台机器上使用一样的端口号，所以我后台使用8089，前台vue使用8088。
这样一来，当我前台访问后台接口时会抛出异常：
```
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

实际上就是跨域访问的问题，对于springboot项目，目前我了解到的解决方式有两种，一种是注解方式，一种是配置方式。
注解方式又分为类注解和方法注解，类注解的意思就是在controller类上边加注解，例如：
```
@CrossOrigin(value = "http://localhost:8088")
@RequestMapping("/tzxblog/blog")
public class BlogController {
```

方法注解就是在要访问的接口方法上加注解，例如：
```
@CrossOrigin(value = "http://localhost:8088")
@GetMapping("/category-list")
public TzxResVO<List<CategoryInfo>> findCategories
```

使用方法就如上边所示，使用`@CrossOrigin`注解，然后填上允许跨域的域名、端口。
那么上边用法的弊端就是哪里需要跨域，那里就要加注解，可能会需要加很多。
所以也可以使用下边配置的方式，在一个地方进行统一的管理，代码示例如下：

```
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
	@Override
	public void addCorsMappings(CorsRegistry registry) {
	registry.addMapping("/**").allowedOrigins("http://localhost:8088").allowedMethods("*").allowedHeaders("*");
	}
}
```

上边这个配置类，能解决跨域的问题，同时还能解决静态资源访问，例如图片访问的问题。

## 图片资源访问
常规来说，博客有作者，作者有头像，加载博客列表可能就同时加载了作者昵称和头像。
对于头像获取的实现方式可能是是多样化的，有的直接以流的方式提供，有的可能以ftp的方式提供，有的可能就像我这里一样直接使用静态资源，只是需要进行一定的配置进行映射，从而实现网络路径和本地文件路径的对接。
这个功能实现起来也不难，只需在上边配置类中增加一个重写方法即可：
```
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/static/**").addResourceLocations("file:///" + imgConfig.getFilepath());
	}
```

上边代码的意思，就是当网络访问/static/下的资源时，系统访问本地某个文件路径的内容，就是一层虚拟服务的映射。
上边文件路径使用配置的方式，没有在代码中写死，所以就又用到了spring属性配置的内容。

## 自定义属性配置
一般系统开发可能是在windows系统中，这个文件系统是以各种盘符开头，而后期的使用则很可能是部署在linux环境，盘符则又是一种规则，所以代码中写死肯定是不明智的。
因此，我直接把路径配在了springboog的配置文件`application-dev.yml`中，例如：
```
imgs:
  filepath: E:/img/
```

然后在代码中使用`@Value`获取时，会发现启动都报错，实际上是还需要引入一定的依赖：
```
<dependency>
	<groupId> org.springframework.boot </groupId>
	<artifactId> spring-boot-configuration-processor </artifactId>
	<optional> true </optional>
</dependency>
```

然后注入配置属性的代码指定配置内容，就可以在代码中读取自定义配置了：
```
@Component
@ConfigurationProperties(prefix = "imgs")
@Data
public class ImgConfig {
	// 上传路径
	private String filepath;
}
```

## 关于lombok
在上边的自定义配置映射类中，有一个`@Data`注解，这个注解是lombok的功能，lombok可以简化开发工作，替代自己写的get、set以及toString等方法。
除了`@Data`注解，我们目前常用的还有`@Slf4j`，用来代替常规的自定义log变量方式。
但是lombok也有一定的弊端存在，例如有父子继承关系的对象的equals方法就有一定的问题，同时，一旦使用了lombok，那么所有用这个项目的人也都必须安装lombok插件，实际上是很不友好的。

## 项目地址
项目代码和文档均以github托管，地址如下：
<https://github.com/tuzongxun/tzxblog>