---
title: 使用sonar进行java代码质量管理
date: 2017-10-19 08:49:06
tags: sonar
categories: 代码管理
toc: true
---
# 前言
应公司要求，这一次的开发需要进行sonar进行静态代码质量检测。
接到这个任务的时候，我还并不知道sonar是什么，但听到静态代码检测几个字的时候，我下意识的以为是类似checkstyle之类的工具，但是真正用过之后我发现我错了。
我发现实际运行的时候，似乎并不纯粹是静态，因为整个检测过程中还会连接数据库，还会发送http请求，还会连接svn等等。
用完之后，深感这个工具的好用，不检测不知道，一检测吓一跳，竟然检查出来了26个bugs，可靠性级别是像毒血一样的黑红E。
那么废话不再多说，进入主题，记录一下整个环境搭建和检测过程，以便备忘。
<!--more-->
# 安装
安装主要是参考了一篇博文[使用 Sonar 进行代码质量管理](https://www.ibm.com/developerworks/cn/java/j-lo-sonar/),不过有一些细节略有区别。

## 下载
执行git命令`git clone git://github.com/SonarSource/sonar.git`
前提是安装了git，我下载下来的sonar版本是6.5，本地jdk环境是1.8。

## 解压
这个就不用多说了

## 启动
在解压后的目录中一层层找到windows-x86-64\StartSonar.bat，当然了，这里需要选择适合自己电脑操作系统的目录。
如果启动更不报错，就可以进行下一步。
我在第一次启动的时候没有启动成功，查看日志发现是h2内存数据库启动失败，然后想起来我电脑安装的有h2并设置了windows服务自动启动，所以端口占用，导致sonar里的h2启动失败，然后关闭了本机的h2之后，成功启动。

## 访问
浏览器访问localhost:9000,前提是启动不报错。

# 插件
在我参考的那篇博客里，安装完sonar之后就是装插件，我当时不知道那插件具体干嘛用的，就抱着试一试的心态，并没有安装，而是直接跳过了，而在保证完成手头工作的情况下，后续也没有再安装任何插件。

# 配置
sonar工作的时候，要使用到数据库，会把要检测的项目的代码导入到数据库中，所以必须进行数据库的配置，我这里是使用的mysql数据库。

## 创建sonar用户并授权
在一开始完全不会的情况下，我跟着上边博客一步步的做，以为必须是sonar用户，但后来看到其他一些博客中并没有用这个用户，想来应该其他用户也可以，不过我没有试。
创建用户命令`CREATE USER sonar IDENTIFIED BY 'sonar';`
用户授权命令`GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'localhost' \ IDENTIFIED BY 'sonar' WITH GRANT OPTION;`

## 建库
创建一个给sonar用的mysql库，不用建表，sonar会自动建表，这一步就不多说了。

## 配置sonar.properties
打开sonar安装目录，也就是之前解压的目录，找到conf下的sonar.properties文件，编辑这个文件。
1. 找到
<pre>
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
</pre>
这一行，原本是注释的，去掉注释。
2. 找到`sonar.jdbc.username`和`sonar.jdbc.password`，都填上sonar，如果上边建立用户那里不是sonar，可能需要改一下。
3. 将 MySQL 的驱动文件（如 mysql-connector-java-5.1.34.jar）拷贝到 安装目录的extensions\jdbc-driver\mysql 目录下，如果没有这些目录就自己创建。我的一开始没有mysql这个目录，就是自己创建的。

## 配置maven的settings.xml
在`<profiles></prifiles>`加入如下配置
<pre>
 &lt;profile>
   &lt;id>sonar&lt;/id>
   &lt;activation>
       &lt;activeByDefault>true&lt;/activeByDefault>
   &lt;/activation>
   &lt;properties>
        &lt;sonar.jdbc.url>
        jdbc:mysql://localhost:3306/sonar
        &lt;/sonar.jdbc.url>
        &lt;sonar.jdbc.driver>com.mysql.jdbc.Driver&lt;/sonar.jdbc.driver>
        &lt;sonar.jdbc.username>sonar&lt;/sonar.jdbc.username>
        &lt;sonar.jdbc.password>sonar&lt;/sonar.jdbc.password>
       &lt;sonar.host.url>http://localhost:9000&lt;/sonar.host.url>
   &lt;/properties>
&lt;/profile>
</pre>
我上边参考的那个文档里 url写的是这样`jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8`,但我这样配之后执行mvn命令报错，所以就去掉了后边一截。

# 进行代码检测
代码检测主要就是在cmd窗口执行了两个mvn的命令，分别是`mvn clean install`和`mvn sonar:sonar`,如果这两个命令执行结果都是build success，基本上就没有问题了，而我在运行的过程中有遇到下边一些问题。

## java_home的问题
被检测的项目使用的是jdk1.8，虽然我在cmd命令行窗口执行`java -version`结果也是1.8，但是环境变量的java_home配置却是1.6的路径，导致执行mvn命令的时候出现如下错误`Unsupported major.minor version 52.0`，把java_home换成1.8之后问题消失。

## maven镜像的问题
由于我的maven配置的仓库镜像是阿里云的镜像，而公司最近封网，导致使用内网的时候执行`mvn clean install`命令，需要的插件无法下载，切换到外部网络之后问题解决。这些插件应该是只有第一次执行的时候才下载，后边继续用内网就没有问题。

## 数据库连接的问题
这里说的数据库不是sonar要用的mysql，而是项目里的数据库。
由于一开始以为静态代码检查跟数据库无关，因此项目里要连接的oracle数据库没有启动，导致执行上边命令的时候，因数据库连接不上而失败，启动oracle数据库后问题解决。

## svn连接的问题
这个问题其实我还没太明白，因为项目里似乎并没有配置svn相关的东西，但是执行`mvn sonar:sonar`的时候，却因svn连接不上而失败，当切换网络连接上svn后，问题解决。

## 代码优化
上述问题都解决之后，使用localhost:9000访问之后，就可以看到我们要检查的项目。看到页面之后的操作，自己点一点鼠标就很容易明白，很容易找到有问题的代码具体的类，具体的行数等，甚至页面上还会给出优化方案，然后就可以根据具体显示出来的代码及优化方案进行优化了。



