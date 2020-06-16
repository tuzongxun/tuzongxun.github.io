---
title: 《maven实战》学习笔记3——maven使用入门
date: 2017-11-07 15:09:42
tags: [maven,开发工具]
toc: true
categories: 开发工具
---
### 说明
由于我目前所做的项目皆是java web项目，因此整个系统学习的过程也就以java web为基础。
### maven项目结构
根据maven约定，一个比较标准的maven java web项目，结合eclipse通常可以看到结构如下：
<!--more-->
<pre>
src/main/java            java主要代码存放目录
src/main/resources       java配置文件存放目录
src/test/java            java测试代码存放目录
JRE System Library       jre中的一些jar包映射
Maven Dependencies       根据pom.xml配置的依赖而导入的jar包映射
src/main/webapp          web相关的文件，如html等
src/test                 这个目录暂时还没用过
target                   maven项目输出目录，例如生成多的war包
pom.xml                  maven项目最重要的一个文件
</pre>

### pom.xml文件说明
#### pom.xml文件格式
pom.xml文件是maven项目的灵魂所在，一个标准的pom.xml文件格式大概如下：
<pre>
&lt;?xml version="1.0" encoding="UTF-8"?\>
&lt;project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"\>
	&lt;modelVersion>4.0.0&lt;/modelVersion>
	&lt;groupId>com.cmupay&lt;/groupId>
	&lt;artifactId>cmupay-coresys&lt;/artifactId>
	&lt;version>0.0.1-SNAPSHOT&lt;/version>
	&lt;packaging>war&lt;/packaging>
	&lt;name>cmupay-coresys&lt;/name>
	&lt;description>Demo project for Spring Boot&lt;/description>
	&lt;properties>
		&lt;project.build.sourceEncoding>UTF-8&lt;/project.build.sourceEncoding>
		&lt;project.reporting.outputEncoding>UTF-8&lt;/project.reporting.outputEncoding>
		&lt;java.version>1.8&lt;/java.version>
	&lt;/properties>
	&lt;dependencies>
		&lt;dependency>
			&lt;groupId>commons-lang&lt;/groupId>
			&lt;artifactId>commons-lang&lt;/artifactId>
			&lt;version>2.6&lt;/version>
			&lt;scope>test&lt;/scope>
		&lt;/dependency>
	&lt;/dependencies>
	&lt;build>
		&lt;plugins>
			&lt;plugin>
				&lt;artifactId>maven-compiler-plugin&lt;/artifactId>
				&lt;configuration>
					&lt;source>1.8&lt;/source>
					&lt;target>1.8&lt;/target>
				&lt;/configuration>
			&lt;/plugin>
		&lt;/plugins>
	&lt;/build>
&lt;/project>
</pre>
#### pom.xml节点说明
这个文件中，首先是xml格式说明，接下来是pom.xml文件中的父节点`<project>`
##### project
project是pom.xml文件中的父节点(或者说根元素)，节点中有一些属性配置，这些属性可以说不是必须的，但是一般都会写上，有了这些配置可以使一些工具更容易解析和编辑pom.xml文件，例如使用eclipse编辑。
##### modelVersion
`<modelVersion>`指定了pom文件模板的版本，根据我的理解，和`<project>`中的属性涉及到的版本应该一致。需要注意的是，对于maven2和maven3来说，只能是4.0.0。
##### groupId、artifactId及version
这三个元素可以说是pom.xml中最重要的元素，他们三个决定了当前maven项目的坐标唯一性，貌似可以理解成三维坐标的x、y、z三个坐标，然后决定了一个确定的点。</br>
其中`<groupId>`指定当前项目所属的组，一般可能就填写公司的网址,由于网址的唯一性，这个就不会重复，相当于三维坐标中从x轴定位；</br>
`<artifactId>`指定了当前项目在当前组中的唯一ID，通常都是填写的项目名，同一个公司可能有多个项目，但不同项目有不同的编号和名称，相当于在三维坐标确定的x坐标位置再确定y坐标；</br>
实际开发过程中，经常是以迭代的形式进行，因此不同的项目就会有很多个不同的版本，单独用项目名也不能确定项目的唯一性，通过`<version>`指定项目版本，再结合前边两项，就能保证项目坐标的唯一性，这样在maven不同项目间的依赖上就能通过配置准确的获取配置的依赖项。
##### packaging
这里主要是指定项目打包时是生成什么形式的压缩包，例如通常web项目会填写“war”。
##### name和description
因为项目名称有时候可能并不能很好的表达该项目做什么，因此name元素可以用来声明一个更加友好的名称，同样的description是更加详细的项目描述。
##### properties
maven项目基础属性配置，通常会配置项目构建的字符集编码，输出，也就是打包文件的字符集以及jdk版本等。
##### dependencies
这里主要是项目依赖的配置，也就是通常所说的jar包配置，一个`<dependencies>`里边可以有很多个`<dependency>`元素，而`<dependency>`里边就是实际依赖包的配置，配置的原理实际上就是上边描述过的项目坐标，通过`<groupId>`、`<artifactId>`和`<version>`确定唯一的jar包。</br>
当有了上述三个配置后，maven就会自动下载相应的jar包，至于具体是去哪里下载，就要涉及到一些其他的配置了，后续熟悉过程中再进一步说明。</br>
`<dependency>`里边还有一个属性是`<scope>`,指的是作用范围，如果是“test”，说明只在测试的时候生效，如果没有配置这个值，默认是"compile",表示对主代码和测试代码都生效。
##### build
build里边通常是对maven构建项目过程中需要用到的maven插件进行配置，例如上述例子中指定编译插件版本。</br>
在实际使用过程中，我发现当配置了`<properties>`中的jdk版本后，似乎不需要再在这里配置maven编译插件版本也可以，具体细节可能还需要在后续熟悉的过程中进一步了解。
### maven常用命令
在maven构建项目的过程中，有几个比较常用的命令或者说操作，那就是编译、测试、打包、安装。这些命令可以再cmd窗口执行，但我通常结合eclipse使用，因此就记录在eclipse中使用的方法：</br>
在项目名或者pom.xml右键——》run As——》maven build，然后在出现的界面的Goals中输入具体的命令（如果是在cmd命令行窗口，需要加入在前边加上mvn）</br>
#### clean complie 编译
这个命令实际上会先执行clean，清除输出目录target中的所有内容，然后进行编译，把编译的文件放入到target目录中来。
#### clean test 测试
这个命令就是执行测试类里的测试代码，一般使用junit测试。需要注意的是，在执行test之前会下进行compile。
#### clean package 打包
这个操作一般是在项目开发完成后，生成可执行的jar包或者war包或者供其他项目调用的jar包。需要注意的是，在执行package之前会先依次执行compile和test。
#### clean install 安装
项目打包完成后，jar包输出在target目录下，而不是maven仓库，如果其他项目要依赖引用，就没有那么方便。使用install之后，maven就会把jar包安装到maven仓库中。</br>
需要注意的是，执行install之前也会依次执行compile、test、package，所以实际上如果是要把项目压缩包放入到maven仓库，那么只需要执行install就够了。