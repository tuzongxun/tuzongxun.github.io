---
title: 《maven实战》学习笔记6——maven聚合和继承
date: 2017-11-14 15:09:42
tags: [maven,读书笔记]
toc: true
categories: [读书笔记,maven]
---
### maven聚合
#### 为什么要用maven聚合
随着互联网的发展，一个项目的业务复杂度越来越高，整个项目的业务代码也会越来越庞大，因此便有了把一个项目拆分成若干个子项目的需求。
根据之前的知识，clean、test、package、install等都是针对单独的项目，那么对于上边若干个子项目可能就需要执行若干次的clean、test、package、install等操作，而这些操作具有很高的重复性。
介于这样的前提，便需要使用到maven聚合，经过一定的配置后，可以让我们每一次clean或者test或者package等都同时对所有子模块生效，大大减少了重复工作。
<!--more-->
#### maven聚合怎么用
根据上边的前提可以知道，属于同一个项目的子项目，或者说子模块，规范性的配置应该是他们groupId都一样，version也一样，只有artifactId不同。
例如有这样两个子项目mytest-project1、mytest-project2，他们的pom.xml的必要配置分别如下：
myporject1的pom.xml：
<pre>
  &lt;modelVersion>4.0.0&lt;/modelVersion>
  &lt;groupId>my.maven.test&lt;/groupId>
  &lt;artifactId>mytest-project1&lt;/artifactId>
  &lt;packaging>war&lt;/packaging>
  &lt;version>0.0.1-SNAPSHOT&lt;/version>
  &lt;name>mytest-project1 Maven Webapp&lt;/name>
</pre>

myporject2的pom.xml：
<pre>
  &lt;modelVersion>4.0.0&lt;/modelVersion>
  &lt;groupId>my.maven.test&lt;/groupId>
  &lt;artifactId>mytest-project2&lt;/artifactId>
  &lt;packaging>war&lt;/packaging>
  &lt;version>0.0.1-SNAPSHOT&lt;/version>
  &lt;name>mytest-project2 Maven Webapp&lt;/name>
</pre>

那么就可以再创建一个只有pom.xml的项目作为聚合这两个子项目的父项目，这个父项目的pom.xml的必要配置如下：
<pre>
  &lt;modelVersion>4.0.0&lt;/modelVersion>
  &lt;groupId>my.maven.test&lt;/groupId>
  &lt;artifactId>mytest-project0&lt;/artifactId>
  &lt;packaging>pom&lt;/packaging>
  &lt;version>0.0.1-SNAPSHOT&lt;/version>
  &lt;name>mytest-project0 Maven Webapp&lt;/name>
  &lt;modules>
     &lt;module>mytest-project1&lt;/module>
     &lt;module>mytest-project2&lt;/module>
  &lt;/modules>
</pre>
对比之下可以知道看到，这个父项目的配置中modelVersion、groupId、version都和子项目的一模一样。其他除开artifactId、name本身每个maven项目都应该不一样之外，还有packaging的值不一样，并且还多了一个`modules`属性,而`packaging`和`modules`这两个属性就是maven聚合最重要的属性了。
首先，用来作为聚合其他子项目的父级项目的packaging的值必须是`pom`，也只能是`pom`。
然后`modules`中配置了需要聚合的子项目的列表。
如上边例子中配置的子项目列表是什么意思呢？比如`<module>mytest-project1</module>`,这个其实指像的是一个pom.xml文件，也就是`工作空间/mytest-project0/mytest-project1/pom.xml`，说的更简单点，就是需要把子项目mytest-project1放到mytest-project0里边。
那么如果mytest-project1和mytest-project0位于同级目录怎么办呢？这个聚合的`modules`里的配置就要该成这样：
<pre>
&lt;module>../mytest-project1&lt;/module>
</pre>
由此其实可以看到，这个module里边的配置实际上就是一个相对路径，是相对于当前这个pom.xml（聚合项目的pom.xml）的相对路径。
例如`<module>mytest-project1</module>`，其实就是mytest-project0/pom.xml所在目录下的mytest-project1，而`<module>../mytest-project1</module>`，其实就是mytest-project0/pom.xml所在目录的上一级目录中的mytest-project1，`../`代表上一级目录，这个应该能来学maven的人都知道。
经过上边这样配置之后，那么我们只需要对这个聚合父项目进行clean等操作，就会同时对配置的各子模块生效，例如执行clean，那么mytest-project1和mytest-project2下边的target就都会被清理掉，执行package，那么mytest-project1和mytest-project2就都会执行package操作，然后在各自的target目录下生成各自的输出文件，这些结果在执行操作的时候就可以再控制台看到信息输出，例如执行clean，就会如下：
<pre>
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] mytest-project1 Maven Webapp
[INFO] mytest-project2 Maven Webapp
[INFO] mytest-project0 Maven Webapp
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building mytest-project1 Maven Webapp 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ mytest-project1 ---
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building mytest-project2 Maven Webapp 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ mytest-project2 ---
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building mytest-project0 Maven Webapp 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ mytest-project0 ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] mytest-project1 Maven Webapp ....................... SUCCESS [  0.164 s]
[INFO] mytest-project2 Maven Webapp ....................... SUCCESS [  0.004 s]
[INFO] mytest-project0 Maven Webapp ....................... SUCCESS [  0.003 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.272 s
[INFO] Finished at: 2017-11-14T15:07:59+08:00
[INFO] Final Memory: 6M/123M
[INFO] ------------------------------------------------------------------------
</pre>

### maven继承
maven继承的作用其实和java中继承的作用是类似的，基本上都是主要为了实现重用，java中类的继承，减少了重复的代码，而maven中的继承则减少了pom.xml的配置。
因此显而易见的是，这里所说的maven继承更确切的说，应该指的是pom.xml配置的继承。
说到继承，自然是需要有父类和子类，那么引申到这里，就需要有父的pom.xml和子pom.xml,例如我这里有一个父项目pom.xml的必要内容如下：
<pre>
  &lt;modelVersion>4.0.0&lt;/modelVersion>
  &lt;groupId>my.maven.test1&lt;/groupId>
  &lt;artifactId>mytest-project00&lt;/artifactId>
  &lt;packaging>pom&lt;/packaging>
  &lt;version>0.0.1-SNAPSHOT&lt;/version>
  &lt;name>mytest-project00 Maven Webapp&lt;/name>
  &lt;url>http://maven.apache.org&lt;/url>
  &lt;dependencies>
     &lt;dependency>
    	 &lt;groupId>org.springframework&lt;/groupId>
    	 &lt;artifactId>spring-webmvc&lt;/artifactId>
    	 &lt;version>4.3.9.RELEASE&lt;/version>
     &lt;/dependency>
  &lt;/dependencies>
  &lt;build>
    <finalName>mytest-project00&lt;/finalName>
  &lt;/build>
</pre>
这个配置只显示的引用了spring-webmvc的依赖，所以会导入spring-webmvc及它所依赖的所有jar包，这个项目看起来和一个普通的maven项目几乎没有区别，唯一不同的就是，他的`packaging`和聚合一样也是pom才行。
那么如果我们有一个子项目也需要使用spring-webmvc，就可以继承这个项目的pom.xml，而不需要再另行配置spring-webmvc的依赖，例如子项目的pom.xml如下：
<pre>
  &lt;modelVersion>4.0.0&lt;/modelVersion>
  &lt;groupId>my.maven.test1&lt;/groupId>
  &lt;artifactId>mytest-project01&lt;/artifactId>
  &lt;packaging>war&lt;/packaging>
  &lt;version>0.0.1-SNAPSHOT&lt;</version>
  &lt;name>mytest-project01 Maven Webapp&lt;/name>
  &lt;parent>
     &lt;groupId>my.maven.test1&lt;/groupId>
     &lt;artifactId>mytest-project00&lt;/artifactId>
     &lt;version>0.0.1-SNAPSHOT&lt;/version>
     &lt;relativePath>../mytest-project00/pom.xml&lt;/relativePath>
  &lt;/parent>
  &lt;url>http://maven.apache.org</url>
  &lt;dependencies>
  &lt;/dependencies>
  &lt;build>
    &lt;finalName>mytest-project01&lt;/finalName>
  &lt;/build>
</pre>
很明显的可以看到，上边的配置中并没有显示的配置任何依赖，但是多了一个`parent`的配置，而这个配置就是继承父项目的关键配置。
在parent中，groupId、artifactId、version这几个坐标的配置相信不用多说，关键在于`relativePath`，这个属性指定了maven父项目的pom.xml的路径，当然了，这里也是配置的相对于当前pom.xml的相对路径。
有了这个配置之后，mytest-project01不需要显示的配置对spring-webmvc的依赖，就会导入相应的依赖，也就是继承了mytest-project00的pom.xml的配置。

pom.xml中有很多的属性，这些属性并非都能继承，根据我目前的工作状况，我觉得需要知道有这么回事就好了，因此暂时便不做更深入的了解。
maven聚合和maven继承可以让我们的maven开发过程更加简洁，两个可以结合在一起使用，既然知道了怎么聚合，也知道了怎么继承，那么结合在一起使用应该就不用多说了。
