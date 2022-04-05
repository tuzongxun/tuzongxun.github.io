---
title: 若依管理系统RuoYi-Cloud版搭建记录
date: 2020-10-30 15:43:58
categories: 问题整理 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 问题整理
---
现在快速构建web应用程序的系统有很多，若依RuoYi是其中一个，根据官网说明，`使用最流行的技术SpringBoot、Shiro、Thymeleaf、Vue、Bootstrap`，这个系统分为一体化版本和前后端分离版本。不管是学习某些技术，学习整体架构设计思想，还是拿来进一步二次开发，都是不错的选择。

最近抽空搭了下这个环境，基本按照官网说明，但也有一些细节略有差异，以下为踩坑记录：

<!--more-->

## RuoYi-Cloud代码下载

若依系统官网是[http://ruoyi.vip/](http://ruoyi.vip/)，springcloud微服务版本的代码下载地址是[https://gitee.com/y_project/RuoYi-Cloud](https://gitee.com/y_project/RuoYi-Cloud)，可以直接使用`git clone`把代码下载到本地，下载好之后会看到出现`RuoYi-Cloud`目录，并且在里边有很多内容，例如`ruoyi-auth`、`ruoyi-gateway`、`ruoyi-modules`、`ruoyi-ui`、`sql`等目录。

## Mysql准备

根据若依官网说明，mysql是本系统依赖的基础设施之一，而且必须是`5.5`版本以上，我本机是`5.6`，也就不用重新安装，只需要按照官网说明执行上边下载下来的sql脚本即可，脚本在`sql`目录下，不同版本的命令可能和官网说明略有差异，我这里的是`quartz.sql`、`ry_20200924.sql`、`ry_config_20200924.sql`。

官网相应说明如下：

```
2、创建数据库ry-cloud并导入数据脚本ry_2020520.sql（必须），quartz.sql（可选）
3、创建数据库ry-config并导入数据脚本ry_config.sql（必须）
```



## Nacos和Redis准备

根据说明，这个系统整体架构是采用的`Spring Cloud & Alibaba`，而微服务注册中心和配置中心是`Nacos`，权限认证使用的是Redis，因此这两个应该算是这个系统的基础设施之二。

Redis我之前已经安装，windows中安装也很简单，`Nacos`是第一次使用，还需要去下载，下载地址是[https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases)。

我选择的是`nacos-server-1.4.0-BETA.zip`，然后解压为`nacos-server-1.4.0-BETA`目录。

根据若依官网文档[https://doc.ruoyi.vip/ruoyi-cloud/document/hjbs.html#%E8%BF%90%E8%A1%8C%E7%B3%BB%E7%BB%9F](https://doc.ruoyi.vip/ruoyi-cloud/document/hjbs.html#%E8%BF%90%E8%A1%8C%E7%B3%BB%E7%BB%9F)的说明，需要再Nacos的`application.properties`文件中加入如下内容：

```
# db mysql
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://localhost:3306/ry-config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=password
```

这里需要注意的是，其他都不用改动，`user`和`password`需要根据自己数据库实际用户名和密码修改一下，例如我本地的`password`实际是`123456`。

修改好上边内容后，在`nacos`的`bin`目录执行启动命令即可，例如我在windows系统中执行命令如下：

```
startup.cmd -m standalone
```

执行后会看到nacos启动成功并开启了`8848`端口。

## idea导入服务

官网中示例是用`eclipse`，直接导入`project`，`idea`中我是一个个模块选择性导入，根据官网启动说明，后台只需要启动`RuoYiGatewayApplication`、`RuoYiAuthApplication `、`RuoYiSystemApplication `，因此我也就先只导入了这三个模块，前两个对应的是`ruoyi-auth`、`ruoyi-gateway`，后一个则是在`ruoyi-modules`下的`ruoyi-system`。

导入三个模块，会发现提示有依赖包引入不了，是因为本地maven库中没有这些jar，同时也没有直接把相应源码导入进来，这就需要先把相应的依赖安装到本地maven仓库中。

这些依赖都是本项目中其他模块，因此最简单的方式就是，在`ruoyi-system`目录下执行`mvn clean install`，执行完毕后，在idea中重新加载几个模块就好了。

## sql脚本修改

通过上边步骤，编译没有问题，但是启动却会发现提示mysql和redis都连不上，是因为redis没有启动，windows中使用`redis-server.exe redis.windows.conf`启动`redis`就好了。

而mysql连接不上，是因为sql脚本中的`password`全是`password`，而我的实际是`123456`，因此需要修改密码为实际的mysql密码，然后再启动几个服务，便可以成功启动并正常运行。

## RuoYi-ui运行

这里的前台项目是`ruoyi-ui`，使用的是`vue2.6`，`vue`项目运行前需要安装相应的依赖，在若依官网也有说明。

`cmd`进入到`ruoyi-ui`目录下执行如下命令：

```
npm install --registry=https://registry.npm.taobao.org
```

上边操作需要一点时间，完成之后即可启动前台服务，启动命令`npm run dev`，然后会自动在浏览器打开页面，例如`http://localhost/index`，会看到一个登录界面，默认已经填好了用户名和密码，只需要输入验证码即可登录。

至此，`RuoYi-Cloud`基础服务环境搭建完毕。