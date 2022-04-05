---
title: hadoop安装环境准备和关联知识解析
date: 2020-8-5 11:59:42
tags: hadoop
toc: true
categories: hadoop
---
>本想一口气把redis多学一点，奈何还有常见的如穿透、雪崩、击穿、分布式锁、redis并发原理、linux多路复用、redis集群等都还没梳理清楚，而项目就需要先学习一下hadoop等大数据相关技术，于是不得不暂停redis，转而进入hadoop系列的摸石头过河。

据我了解，一般正式环境的hadoop使用都是需要zookeeper的，但是使用hadoop是否一定要zookeeper这个事，对于刚开始学习hadoop的我来说，还是一个未知数。尤其是网上有的教程写了zookeeper，有的又没写，也就更加的茫然。
茫然不可怕，怕的是一直茫然，而解决茫然最好的办法就是行动起来，只要动起来就会有结果，然后便能自然而然的触发下一步，进而一点一点的从茫然中走出来。
那么第一步自然是先想办法把环境搭起来，根据已知的内容一步步摸索，如果抛开zookeeper不谈，能确定的是jdk和hadoop本身是一定不可或缺的。

<!--more-->
## linux中jdk安装
我现在的虚拟机安装的是迷你纯净版的centos系统，一开始连wget、nc等常用命令都没有，所以就更不用说jdk了，这里其实想说的是，建议大家装虚拟机的时候尽量装纯净版，这样其实能体验很多非纯净版体会不了的乐趣。

jdk的安装实际也很简单，windows中老的版本还需要手动配置环境变量，现在已经精简到就鼠标点点点，然后自动配置环境变量的地步了。
而linux中，虽然还没有自动配置环境变量那么便捷，但也不复杂，配置环境变量本身就是一个通用的技术，操作多了也就像windows中点点点一样简单。
linux安装jdk大体分三步，分别是下载、解压、配环境变量，如果一定要说的话，也可以再加一步，那就是验证。

**下载jdk**
jdk版本现在升级很快，目前已经到14了，但有项目中实际使用的还是8，所以我自然也还是以8为基础，首先是选择对应的安装包，选择页面如下;
[https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)
选择安装包时有一个需要注意的细节是，一定要选择正确位数的安装包，例如centos系统只支持64位，如果下载选择打的是i586而不是x64,则安装以后执行java相关命令会发现报错：
```
java: /lib/ld-linux.so.2: bad ELF interpreter: No such file or directory
```
在此基础上，我选择的安装包是jdk-8u261-linux-x64.tar.gz，在下载页复制到的下载链接是：
[https://download.oracle.com/otn/java/jdk/8u261-b12/a4634525489241b9a9e1aa73d9e118e6/jdk-8u261-linux-x64.tar.gz](https://download.oracle.com/otn/java/jdk/8u261-b12/a4634525489241b9a9e1aa73d9e118e6/jdk-8u261-linux-x64.tar.gz)

这个地方就又可以引申出一些别的知识了，实际上在安装redis的时候也提到过，就是安装包获取方式，既可以直接用windows下载，又可以直接在虚拟机中使用wget命令下载。
实际工作中我们使用最高效可行的方式就可以了，但是既然是学习，自然是要一定程度上为了应用而应用一下，所以之前redis使用的wget，这里就直接windows下载。

**文件传输**
windows下载以后要传到虚拟机中，又有不同的选择，可以使用专门的ftp工具传输，还可以使用redis管道学习的时候拓展到的nc命令。
ftp上传就是纯粹工具的使用，拖拽就完了，所以这里自然是要以看起来高大上的nc来操作。
首先，在我的虚拟机中开启nc文件下载的监听：
```
nc -l 9999 > jdk-8u261-linux-x64.tar.gz
```

然后，在windows的cmd命令窗口，进入到jdk安装包存放目录使用nc执行文件上传：
```
nc 192.168.139.9 9999 > jdk-8u261-linux-x64.tar.gz
```

如果直接按照这个步骤操作，可能会发现windows执行nc命令找不到，这是因为windows中默认也是没有这个的，所以还需要在windows中先安装一下nc。
nc下载地址：https://eternallybored.org/misc/netcat/
下载下来的zip文件解压，解压后可能会还有一层，不管有几层，需要把最内层的所有文件复制到C:\Windows\System32下，然后重启cmd窗口，就能正常使用nc了。
需要注意的是，linux系统本身也没有nc，我是使用redis的时候装过，如果有人linux中也执行不了，那么也需要先安装一下，非常简单：
```
yum instal nc
```
需要提一点的是，实际操作发现，在使用nc传文件的时候，windows向linux传输，文件传输完成了也不会自动断开连接，而linux向linux传输，文件传输完成就会自动断开连接。
所以windows向linux传输时，如果看到文件大小显示已经传输完成不再变化了，可以ctrl+c主动断开连接。
还要注意的是，上边9999端口是随便写的，不和本机现有端口冲突即可，但是由于防火墙存在，可能会连接失败。因此，要么关闭防火墙，要么端口加白名单，我这里就是临时起意试一下，所以防火墙也就使用临时关闭的方式：
```
service iptables stop
```

**解压**
有了安装包以后，下一步是解压，这个就更简单了：
```
tar -zxvf jdk-8u261-linux-x64.tar.gz
```
tar相关内容可以参考redis安装的那篇文章：https://blog.csdn.net/tuzongxun/article/details/107170447

**环境变量配置**
之所以说环境变量的配置简单，是因为这些软件不同，但基本规律一样，像之前redis配置环境变量，就是执行如下命令：
```
export REDIS_HOME=/opt/soft/redis5
export PATH=$PATH:$REDIS_HOME/bin
```
实际就是在PATH环境变量里指向软件的bin目录，linux中多数软件的操作功能命令都是在bin目录下，所以jdk的环境变量配置其实也是这样的，没有一点特殊：
```
export JAVA_HOME=/root/soft/jdk1.8.0_261
export PATH=$PATH:$JAVA_HOME/bin
```
不同的，也就是安装目录不一样而已。有了上边配置后，不能忘记的是需要让配置生效，即执行`source /etc/profile`命令。

**验证**
我们说安装好了不代表真的好了，所以需要进行验证，任意目录执行java版本查询命令即可：
```
java -version
```
正常的情况下就会看到输出几行java版本相关的数据，如果安装包不正确，比如上边说到的位数不对，就会提示相关的错误，我这里输出如下;
```
java version "1.8.0_261"
Java(TM) SE Runtime Environment (build 1.8.0_261-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.261-b12, mixed mode)
```

## hadoop下载安装
安装好了jdk，下一步是下载hadoop安装包，我一开始参考网上教程选择的安装包地址是
http://archive.apache.org/dist/hadoop/core/hadoop-2.7.5/
同样的，有了下载地址，下载方式就可以多种选择，可以windows下载，也可以wget下载，这里就不在重复了。
需要注意的是，上边这个网址流量有限制，目前看到的是每天10G，所以有可能存在下载不成功的情况，会提示当天流量已经超过了10G。

hadoop一般是集群方式出现，所以我准备了3台虚拟机，这里实际也有偷懒的方式，即先一台机器装好一套环境，然后使用虚拟机快照和克隆功能快速的创建多个。
但是还是上边说的为了应用和练习，我以及采用一个一个手动创建和安装各种环境的方式。
这种方式便又涉及到安装包获取问题，根据之前的操作，已知的就有wget直接下载、windows下载+ftp工具上传、windows下载+nc上传，实际还有更多方式，比如这里我就采用新的方式scp。
在192.168.139.9这台机，我已经有了jdk和hadoop安装包，另两台机分别是192.168.139.19和192.168.139.29，则可以使用scp复制文件到另两台机器：
```
scp jdk-8u261-linux-x64.tar.gz root@192.168.139.19:/root/soft/zip/
scp jdk-8u261-linux-x64.tar.gz root@192.168.139.29:/root/soft/zip/
```
这样一来，安装包获取方式就又增加一个选择，即wget直接下载、windows下载+ftp工具上传、windows下载+nc上传、系统间scp复制。

**补充**
上边的安装包是可以使用的，但是实际现在官网最新版已经到了3.3，所以为了不至于落伍太多，后边又重新下载了较新的3.1.3版本，可以到hadoop官网下载：
[https://hadoop.apache.org/releases.html](https://hadoop.apache.org/releases.html)

而后续部署也发现2.6和3.1.3的版本操作上还是有不少区别的，后续操作将会以3.1.3为准，但是部分遇到的差异也会有所提及。

至此，第一步的环境准备以及知识拓展告一段落。