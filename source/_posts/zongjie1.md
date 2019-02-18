---
title: 技术问题小总结1
date: 2017-12-01 15:43:58
categories: 问题整理 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 问题整理
---
## 前言
做项目是提高技术最高效的手段，这句话从某种程度上而言真是太对了。
因为不论是大项目还是小项目，不论是正式项目还是个人业务项目，只要在做，就总能遇到各种各样的问题，从而能直接着重于某个点去学习。
以下便是最近做`业余小项目-tzxblog`<https://github.com/tuzongxun/tzxblog>时遇到的一些问题和解决办法记录。
<!--more-->

## springboot日志的问题
### 问题描述
springboot默认的是logback日志框架，不做任何配置的情况下，日志输出在控制台。
我想要通过配置把日志内容输出到指定的文件中，于是做了如下的配置：
<pre>
logging.level.root=warn
logging.level.com.tzx.blog=warn
logging.file=mylog.log
logging.path=C:/Users/tzx/Desktop/mylog/
</pre>

我下意识的认为`logging.file`是日志文件名，`logging.path`是日志文件输出的目录。
但是实际运行时发现我想要的`C:/Users/tzx/Desktop/mylog/mylog.log`文件没有出现。

### 问题分析
经查阅springboot官方文档，发现了如下的一些内容：
![logg](/images/logging.png)

很显然，官方文档中并没有我这种配置。
根据上边的描述可知这里的日志有三种配置方式：
1. `logging.file`和`logging.path`都不做任何配置，则默认日志输出到控制台
2. 只配置`logging.file`，如果这里只是一个文件名，则会在当前目录下输出一个日志文件；如果是一个绝对路径下的文件名，则会在生成一个该绝对路径的文件
3. 只配置`logging.path`，在配置的目录下生成一个`spring.log`日志文件

### 解决办法
根据上边分析，我选择的解决方式如下：
去掉`logging.path`,直接在`logging.file`中配置成`C:/Users/tzx/Desktop/mylog/mylog.log`。

## markdown语法解析
### 问题描述
由于之前写博客时，那些博客网站基本都支持markdown语法，经过使用后，发现确实很方便，于是决定自己写的这个博客系统也使用markdown。

### 问题分析
markdown语法解析需要引入markdown相关的js，并使用该js中定义的方法格式化博客内容。

### 解决办法
引入`showdown.min.js`文件，同时使用如下自定义代码格式化博客内容：
<pre>
var text = data.blogContent;
var converter = new showdown.Converter();
var html = converter.makeHtml(text);
document.getElementById("curcontent").innerHTML = html;
</pre>

## jpa关联查询的问题
### 问题描述
博客中实体类的初始设计是单独的`userModel`和`blogModel`,如果要在显示某篇博客时显示出作者名称等信息，则需要根据blog表中的userId字段再查user表。

### 问题分析
jpa里边也可以自定义sql，但是总感觉这样会很麻烦。
而且从某种程度而言，jpa应该不至于连这么简单的需求都无法处理，于是经过查找，得到了关联关系的解决办法。

### 解决办法
blog表和user表都保持不变，在blog表中有一个字段userid用来保存作者的id。
userModel保持不变，在blogModel中userId的属性使用如下定义：
<pre>
/**
*多个博客对应一个人，关联字段bloginfo表中user_id字段
*/
@JoinColumn(name = "user_id")
@ManyToOne
private Userinfo userinfo;
</pre>

`@JoinColumn`注解指定了关联字段，`@ManyToOne`声明关联关系是多对一。
之后使用jpa的查询方法，例如`userDao.findAll()`,进行查询时，每一条blog记录中都会带上相应的user对象，也就只需要一次查询数据库就够了。

## 使用thymeleaf模板后，不能返回对象给前台
### 问题描述
使用thymeleaf模板渲染页面的时候，spring的controller的方法只需要return一个html的页面文件名就行了，例如`index.html`，就写成`return "index";`。
使用过程发现这里不是只需要写成`return "index"`，而是必须写成这样，还必须有这个页面存在，否则报错。
例如我本来想在某个方法中返回一个blog对象给前台，但是向这个方法发起请求时后台报错如下：
<pre>
org.thymeleaf.exceptions.TemplateInputException: Error resolving template 
"tzxblog/openBlog", template might not exist or might not be accessible by any of
the configured Template Resolvers
</pre>

### 问题分析
看到上边的问题，我猜测可能是用了模板以后，会对所有return的内容进行解析渲染，但是对象是不能直接渲染成html的，因此就解析出错。
这个时候我有两种解决办法，一种是把数据放到modelMap中，然后使用类似th:text这样的方式取得数据，之后还是return回一个html页面。
但是这个地方我不想这样，因为我用了ajax请求，就想要在ajax中获取到返回的数据后进行一定的前台处理，因此就必须要找到能让我返回对象的办法，好在后来在查询的过程中，从其他地方获取到了对的解决办法。

### 解决办法
在需要返回对象或者普通数据的方法上加上`@ResponseBody`注解，有了这个注解以后，thymeleaf模板就不会再做其他的处理，也就相当于暂时和thymeleaf没了关系。