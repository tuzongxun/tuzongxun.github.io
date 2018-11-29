---
title: 《maven实战》学习笔记7——maven项目版本管理和灵活构建
date: 2017-11-15 09:09:42
tags: [maven,读书笔记]
toc: true
categories: [读书笔记,maven]
---
### 说明
《maven实战》一书内容很多，整个maven要学的东西也很多，不过，结合个人实际情况，我打算把这一篇作为这次对maven学习的一个阶段性收尾，待其他更急需补充的知识有一定眉目了，再回过头来继续深入。
<!--more-->
### maven版本管理
对于maven版本管理，最重要的是需要区分出快照版本SNAPSHOT和发布版本release，据目前的了解，快照版本格式是固定的，而发布版本有几种，例如带release单词的和不带release单词的。
快照版本由于每次发布都带时间戳，所以适用于开发阶段团队协作，但同时也是不稳定的。
发布版本就需要是比较稳定的版本，定了就不能变，如果要变就要发布新的发布版本。
不过，要更好的了解maven版本管理，必须先比较熟悉版本控制，结合实际情况，就是需要先比较熟悉svn版本控制。
由于目前我虽然天天在用svn，却还没有系统的学一下，所以对svn也还不是太熟，再加上目前实际工作对maven版本管理似乎也没有什么需求，因此maven版本管理这一块就先只做了解。

### 灵活构建
根据我的理解，maven灵活构建指的是经过一定的配置后，可以使maven在不同的场景很容易的改变一些构建细节，例如某些属性实际值的使用。
maven提供了属性properties、profiles、资源过滤三个特性来实现。

#### 属性properties
这个说白了就相当于配置的一个变量，里边的标签名就是变量名，只不过有一些标签是maven默认的，或者说约定了的，同时也能自定义并使用，例如我在某个项目的pom.xml中这样配置：
<pre>
&lt;properties>
    &lt;project.build.sourceEncoding>UTF-8&lt;/project.build.sourceEncoding>
    &lt;project.reporting.outputEncoding>UTF-8&lt;/project.reporting.outputEncoding>
    &lt;java.version>1.8&lt;/java.version>
    &lt;spring.version>4.0.9&lt;/spring.version>
    &lt;auth.name>tzx&lt;/auth.name>
&lt;/properties>
</pre>
在上边的例子中，前三个是maven默认的一些属性名，而spring.version、auth.name是我自己加的，有了这些配置，在这个pom.xml文件中就能用`${auth.name}`、`${spring.version}`这样的方式获取到里边配置的值，这样大大增加可维护性以及减少重复配置。
就比如这里的spring.version，如果这个项目导入了很多个spring的jar包，那么每个依赖配置都要配置version，如果有了spring.version，那么在依赖配置的version那里就可以使用`${spring.version}`，以后如果要升级spring版本，就可以只改动properties中的spring.version的值就可以了。
需要注意的是，我的这个配置是在项目的pom.xml中，但是实际上不只是能配置在这里，还能配置在maven的settings.xml中，例如我之前学习代码管理工具sonar的时候就在setting.xml中配置过mysql数据库相关的属性：
<pre>
&lt;properties>
       &lt;sonar.jdbc.url>
        jdbc:mysql://localhost:3306/sonar
       &lt;/sonar.jdbc.url>
       &lt;sonar.jdbc.driver>com.mysql.jdbc.Driver&lt;/sonar.jdbc.driver>
       &lt;sonar.jdbc.username>sonar&lt;/sonar.jdbc.username>
       &lt;sonar.jdbc.password>sonar&lt;/sonar.jdbc.password>
       &lt;sonar.host.url>http://localhost:9000&lt;/sonar.host.url>
&lt;/properties>
</pre>
除开上边两个之外，maven还能用maven内置属性、环境变量、系统变量等等，可以用`mvn help:system`命令来查看有哪些环境变量和系统变量，至于具体的内置属性有哪些，可以需要时网上查询一下。

#### maven的profiles
在说这个之前，先来看一个在前几篇记录中，讲maven仓库时有在settings.xml中出现过的一个配置：
<pre>
&lt;profiles>
   &lt;profile>
     &lt;id>nexus&lt;/id>
     &lt;repositories>
	  &lt;repository>
	     &lt;id>repo1&lt;/id>
	     &lt;url>http://192.168.0.224:8081/nexus/content/groups/public&lt;/url>
	     &lt;releases>
                &lt;enabled>true&lt;/enabled>
                &lt;updatePolicy>daily&lt;/updatePolicy>
                &lt;checksumPolicy>ignore&lt;/checksumPolicy>
	     &lt;/releases>
	  &lt;/repository>
	  &lt;repository>
	     &lt;id>repo2&lt;/id>
	     &lt;url>http://192.168.0.224:8081/nexus/content/groups/public-snapshots&lt;/url>	     
	     &lt;snapshots>
                &lt;enabled>true&lt;/enabled>
	     &lt;/snapshots>
	  &lt;/repository>
     &lt;/repositories>
     &lt;pluginRepositories>
	  &lt;pluginRepository>
	     &lt;id>repo3&lt;/id>
	     &lt;url>http://192.168.0.65:8082/nexus/content/groups/public&lt;/url>
	     &lt;releases>
                &lt;enabled>true&lt;/enabled>
	     &lt;/releases>
	  &lt;/pluginRepository>
	  &lt;pluginRepository>
	     &lt;id>repo4&lt;/id>
	     &lt;url>http://192.168.0.65:8082/nexus/content/groups/public-snapshots&lt;/url>
	     &lt;snapshots>
                &lt;enabled>true&lt;/enabled>
	     &lt;/snapshots>
	  &lt;/pluginRepository>
     &lt;/pluginRepositories>
   &lt;/profile>
&lt;/profiles>
</pre>
很显然，这就是一个这里要说的profiles的配置，这种配置的基本形式是，在一个`profiles`中可以配置多个`profile`，`profile`里就是具体的配置信息，据我理解，几乎所有能在profile之外配置的信息都可以放到这里边配置。
在这个配置中，最重要的是`id`属性，这是每个`profile`的唯一标示。
之后可以使用很多种方法来激活这个配置，例如可以在settings.xml文件的`profiles`标签之外增加这样的配置：
<pre>
&lt;activeProfiles>
	&lt;activeProfile>nexus&lt;/activeProfile>
&lt;/activeProfiles>
</pre>
然后，上边所配置的profile就被激活，可以被使用了。
除了上边这种激活方式，还可以通过‘命令行激活’、‘环境变量激活’、‘文件是否存在激活’等各种激活方式，不过目前我还完全没有用过这些方式，就暂时认为不常用，稍作了解就好。
那么除开上述的所有激活方式外，还有一种就是在profile内部配置激活：
<pre>
&lt;activation>
      &lt;activeByDefault>true&lt;/activeByDefault>
&lt;/activation>
</pre>
这种配置的意思，从字面意思其实就可以理解，那就是默认激活。不过需要注意的是，假如有用上述任何一种方式激活过某个profile，那么所有这种默认激活的方式都会失效。

#### web资源过滤
这个目前感觉也似乎不怎么常用，因此暂时只需要知道有这么回事就好。