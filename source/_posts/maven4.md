---
title: 《maven实战》学习笔记4——maven坐标和依赖
date: 2017-11-13 15:09:42
tags: maven
toc: true
categories: maven
---
### maven坐标
#### maven坐标是什么
坐标一词最让人熟悉的就是读书时学的x、y轴的横竖坐标，平面中由x、y决定一个唯一的点，x、y就是坐标。三维中，x、y、z决定一个唯一的点，x、y、z就是坐标。而在maven中，就是groupId、artifactId、version、packaging等属性决定一个唯一的项目模块，这些属性就是maven项目的坐标。
<!--more-->
#### maven坐标详解
对于groupId、artifactId、version、packaging，上一篇实际上我已经根据自己的理解说过，但是这本书学到这里后，却发现有些地方略有些偏差，因此还需要进行补充和修正。
对于groupId，我说是项目所属的组，一般可能是公司网址，当时我的理解其实就是公司的网址。
但是读到这一章，却发现groupId实际上应该是针对开源项目发布之后的项目网址，而并非是公司网址。只不过在我之前所接触的项目中，很多人写的全是公司网址而已。
究其原因，可能是因为写的人对maven规范并不是很了解，也或者是公司项目不多，所以也就无所谓怎么写。而根据书中所说，maven的规范是不应该都写公司网址的。
至于artifactId，我之前理解的就是项目名，书学到这里，个人觉得也还是项目名。不过这个项目名应该更具体的理解为项目里边的模块名，一个模块其实也就是一个单独的子项目。
至于version和packaging比较好理解，和之前的没区别，但了解了一个新的、在我看来不怎么常用的属性classifier，这个属性定义maven项目一些附属构件，由maven插件决定，不能像前几个属性一个样直接定义值。由于这个属性我目前不常用，因此只做了解。

### maven依赖
#### 依赖配置
在学习maven一开始就了解到了maven是一个优秀的依赖管理工具，这也是在我看来maven最重要的特性，通常一个比较完整的maven依赖配置是这样的（下边的例子只是为了演示用法，一般不太可能真的从webmvc排除web）：
<pre>
&lt;dependency>
    &lt;groupId>org.springframework&lt;/groupId>
    &lt;artifactId>spring-webmvc&lt;/artifactId>
    &lt;version>4.0.9.RELEASE&lt;/version>
    &lt;scope>test&lt;/scope>
    &lt;type>jar&lt;/type>
    &lt;optional>true&lt;/optional>
    &lt;exclusions>
       &lt;exclusion>
          &lt;groupId>org.springframework&lt;/groupId>
    	  &lt;artifactId>spring-web&lt;/artifactId>
    	  &lt;version>4.0.9.RELEASE&lt;/version>
       &lt;/exclusion>
    &lt;/exclusions>
&lt;/dependency>
</pre>
以上配置中前三个属性就是上边说的最常用的maven坐标，scope是生效范围，例如示例中的test，指的是只有在测试阶段有效，也就是运行测试类的测试方法的时候有用。
scope默认值时compile，表示对编译、测试、运行阶段都有效。除开test和compile，还有runtime、system等，我目前基本不怎么用，也就暂时略过。
type指的是依赖类型，默认是jar，可以理解成jar包，type可以是jar、war和pom等。
optional代表的是可选，值为true代表选择，值为false为不选择，这个属性实际用的也不多。
exclusions、exclusion是排除依赖，例如这里需要带入spring-webmvc的jar包，spring-webmvc依赖于spring-web的jar包，也就是说使用maven显示导入spring-webmvc的jar包时，maven也会把spring-web的jar包也导入进来，如果我们不需要spring-web，就可以在这个地方配置spring-web的坐标排除掉，然后就不会再导入这个依赖包。

#### 依赖传递
还是如上边所示的例子，我们要导入spring-webmvc的jar包，spring-webmvc依赖于spring-web，所以在我们的项目中也会把spring-web的jar包导入进来。
根据上边对依赖配置的解释，我们知道配置中有scope，即依赖范围，如果spring-webmvc中配置的对spring-web的scope是compile，那么在我们的项目中默认导入进来的spring-web的依赖范围就也是compile。
但是上边的结论的前提是我们我们项目中对spring-webmvc的依赖范围和spring-webmvc中对spring-web的依赖范围一致。
但这两个依赖范围不一致的时候，就需要理解第一直接依赖和第二直接依赖这样的概念。就比如上述例子中，我们的项目队spring-webmvc就是第一直接依赖，我们的项目对spring-web就是第二直接依赖。
因此这里需要特备注意的是，有时候如果有第二直接依赖的包，但没有生效，可以从这个角度去考虑一下。
还有一个问题就是，针对配置了可选依赖选项的jar，将不会被传递，如果需要，必须另外手动导入才行。

#### 依赖调解
maven之所以能根据一定的配置就准确的导入我们需要的依赖，那是因为我们的配置，也就是所说的坐标保证了依赖的唯一性，而一旦出现不唯一的情况，那么就需要进行选择，这就涉及到依赖调解，或者就理解成依赖选择。
之所以会出现这种不唯一的情况，一般都是因为依赖传递。
例如我一个项目导入了A.jar和B.jar两个jar包，而A.jar和B.jar都依赖于C.jar，但是这两个C.jar的版本version又不一样，这时候我们的项目就需要选择究竟是使用哪一个C.jar，这就是所谓的依赖调解。
那么首先，根据最短路径原则选择。
联系上边的知识，可以理解成第几直接依赖的问题。比如刚才A、B、C的例子，由于两个C.jar都是第二直接依赖，所以他们路径就是一样长。假如这里是A.jar依赖于C-1.jar，而B.jar依赖于D.jar，而D.jar再依赖于C-2.jar，那么这时候C-1是第二直接依赖，c-2就是第三直接依赖，也就是c-1的路径比较短，那么这时候就会选择c-1而不是c-2。
当然了，上边是假设B.jar依赖于D.jar，那么如果就是一开始那样B.jar就是直接依赖C-2.jar呢，这时候两个的路径是一样的，那就涉及到第一声明优先原则，也就是说在pom.xml中A和B谁在前，谁就会被选择，需要注意的是这个原则是maven2.0.9之后才有的。

#### 依赖优化
根据上边所有的知识，可是知道一个项目的第一、第二乃至第n个直接依赖可能会有若干个重复的依赖声明，但是根据一些原则，最终被实际用到的确实唯一的，这些被最终确认的依赖称为解析依赖，可以通过`mvn dependency:list`来进行查看，然后可以看到各依赖是第几依赖，是根据哪个依赖进来的，然后就可以进行一定的优化。