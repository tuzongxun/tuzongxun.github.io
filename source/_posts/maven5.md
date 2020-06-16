---
title: 《maven实战》学习笔记5——maven仓库和镜像
date: 2017-11-13 15:19:42
tags: [maven,开发工具]
toc: true
categories: 开发工具
---
### 什么是maven仓库
要理解什么是maven仓库，需要先理解什么是maven构件。
什么是mavne构件呢？在本书中解释是：
>在maven的世界中，任何一个maven依赖、插件和maven项目构建的输出都是maven的构件。
<!--more-->
那么我个人的理解就是我们创建项目时需要的jar包、插件包以及项目打包后的文件，等等这些。
这本书中有一个对于maven仓库的比喻，我觉得很是贴切，所以对于maven仓库的解释我觉得完全可以引用。
书中以图书来比喻maven构件，而maven仓库则是个人书房、实体书店以及网上书店，书房和书店是用来放书的房间，而maven仓库就是用来放maven构件的目录。

### maven仓库的分类
根据上边所述，maven仓库就是用来放maven构建的目录，那么对于我们和计算机打交道的人就知道，一种是本地目录，还有一种是非本地目录。
本地目录自然就是我们当前操作的这台计算机的某个目录，而非本地目录就是通过网络连接的服务器的某个目录。
**本地的那个目录叫本地仓库，远程的目录称作远程目录。**
那么构建一个maven项目使用maven仓库的过程，也大概就像一个找书看的过程。
我们构建maven项目时需要根据pom.xml的配置来引用相应的maven构件，就相当于我们需要看什么书的时候去找书。
如果自己有书房，想看书的时候自然会先去书房找一下，如果没有，则可能去实体书店或者网上书店买书，买回来以后放入书房。
同样的，在maven项目需要maven构件的时候，也会先到本地仓库（书房）找一下有没有这个构件，如果没有，则会根据配置（相当于你所知道的实体店地址或网店的网址）去相应的远程仓库下载，下载完以后放入本地仓库。
既然把远程仓库类比为实体书店和网店，那么就自然而然的需要知道远程仓库也是有多种的。
**远程仓库大体上可分为中央仓库、私服和其他公共库。**
中央仓库，个人理解就是maven提供的那个最原始的仓库；私服就是一个小范围的、可以供一部分人用，但又不是完全公开的那种仓库；而其他公共库就是除开maven中央库之外的、其他公司或机构提供的可以被所有人访问的仓库。
一般来说，本地库类似于书房，可以最快的拿到需要的书；私服类似于实体书店，如果不需要很久就能到书店的话，也能很快拿到书；而远程库就相当于网店，下单后需要一定的快递时间才能到货。

### maven仓库的配置
理解了什么是maven仓库以及分类后，接下来就需要知道具体怎么用了。在这里，我就下意识的以为能看到这里的都是知道maven基础使用的，也都应该知道maven的setting.xml文件。

#### 本地仓库配置
仓库分为了本地仓库和远程仓库，对于仓库的配置也是分为了本地仓库和远程仓库，本地仓库的配置比较简单，在setting.xml中如下：
<pre>
  &lt;localRepository>F:\repo_new1&lt;/localRepository>
</pre>
也就是在localRepository中填上本地maven仓库目录的路径。

#### 远程仓库配置
而maven仓库的配置就是在setting.xml文件中，以下是我的某个setting.xml中对maven远程仓库的的配置，这里配的就是一个私服：
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
对于上边的配置，profiles、profile以及开始的那个id就不多说了，除此之外，从外到内，首先是`repositories`和`pluginRepositories`,其中repositories标签里的是主要项目依赖构建的仓库配置,pluginRepositories里的是插件依赖构建的仓库配置。
每个repositories里可以包含几个repository，即具体仓库的配置。在repository里有id和url，id需保证唯一性，url就是具体的仓库的地址。
如我上边例子中，除开id和url外还有两个不一样的属性，一个是releases，一个是snapshots，releases指的是发布版的仓库，snapshots指的是快照版的仓库。
在第一个releases中又配置了三个属性，一个是enable，值为true代表开启，false代表关闭。
还有一个属性是updatePolicy，代表从中央仓库更新构建的频率，如daily，代表每天。updatePolicy的值有多个，具体的这里不说。
另外一个就是checksumPolicy，代表下载构建的时候是否要校验文件的正确性，上述例子中ignore指的是忽略。这个属性也是有多个值，具体的这里略过。
而与repositories类似，pluginRepositories里可以包含几个pluginRepository，具体的其他属性也和上边说的一样。

### maven镜像
maven镜像我在实际使用的时候其实一直都有配置，网上大多的解释也都是说镜像相当于一个代理，或者说网络加速器，用来加速对于maven仓库的访问。
之所以这样说，是因为配置了某个镜像指向我们配置的某个maven仓库后，我们再去访问这个maven仓库，就会被这个镜像拦截，转而去访问这个镜像配置的地址，这样做的目的一般就是为了提高速度。
根据我个人理解，这个镜像应该是采用了网络加速机制再指向实际的maven仓库地址，而这个仓库地址是否就是我们自己配置的和这个镜像关联的maven仓库，其实并没有太大的关系。
上边这句听起来似乎有些绕口，那么不妨通过一个例子来进行理解，就如我的setting.xml中镜像的配置：
<pre>
&lt;mirrors>
   &lt;mirror>
      &lt;id>releases&lt;/id>
      &lt;mirrorOf>*&lt;/mirrorOf>
      &lt;url>http://maven.aliyun.com/nexus/content/groups/public/&lt;/url>
   &lt;/mirror>
   &lt;mirror>
      &lt;id>snapshots&lt;/id>
      &lt;mirrorOf>repo1&lt;/mirrorOf>
      &lt;url>http://maven.aliyun.com/nexus/content/groups/public/&lt;/url>
   &lt;/mirror>
   &lt;mirror>
      &lt;id>maven.oschina.net1&lt;/id>
      &lt;mirrorOf>repo2&lt;/mirrorOf>
      &lt;url>http://maven.aliyun.com/nexus/content/groups/public/&lt;/url>
   &lt;/mirror>
   &lt;mirror>  
      &lt;id>alimaven&lt;/id>  
      &lt;mirrorOf>central&lt;/mirrorOf>
      &lt;url>http://maven.aliyun.com/nexus/content/groups/public/&lt;/url>            
   &lt;/mirror>  
&lt;/mirrors>
</pre>
以上配置基本是我setting.xml文件中的镜像配置了，但是只是为了说明镜像的配置才配置了这么多。
镜像配置的基本格式就是上边这样，由一个`mirrors`标签包含多个`mirror`标签，每一个mirror就是一个具体的镜像。
每个mirror都有一个id，这是镜像的唯一标示，也需要保证唯一性。
url就是具体的镜像的地址，就像上网配置代理一样。
这里需要着重说明的是`mirrorOf`，正是有了这个属性，才使我们配置的镜像和我们配置的仓库地址关联起来，这个`mirrorOf`实际上对应的就是上边所配置的仓库的id，例如这里的`<mirrorOf>repo1</mirrorOf>`就是对应上边仓库配置那里的`<id>repo1</id>`，当我们访问id为repo1的仓库时，就会被这个镜像拦截，继而访问这个镜像实际要访问的maven仓库的地址。例如我这里，不管repo1配置的仓库是什么，最终实际上访问的都是aliyun的maven仓库。
这里需要注意的是，我上边配置了四个镜像，而实际上真正生效的只有第一个，因为第一个的mirrorOf值是通配符`*`，代表所有仓库都走这里，所以下边几个就不会走了。
还需要注意的是最后一个mirrorOf是`central`，这个也有特殊意义，代表的是maven默认的中央仓库的id。
在镜像的实际配置中，其实不止我配置的那几个属性，例如还有name等，这些有时候不是必要的，所以我就略过了。