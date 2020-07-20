---
title: Linux中redis安装及软件安装相关Linux知识要点
date: 2020-7-6 23:09:42
tags: [linux,redis]
toc: true
categories: [redis,linux]
---

无论是Linux还是windows安装软件，都要先有一个安装包。这个安装包可以是直接本机下载，也可以是从其他机器传过来。
在Linux中软件安装方式也有多种，我目前知道了就有yum安装、直接本机下载安装包安装以及他机传递安装包进行安装，这些方式可能常常会搭配着使用，哪个方便用哪个。
<!--more-->
就我个人以往的使用来说，场景大概如下：
>yum：常用来安装一些系统级命令以及软件；
本机直接下载安装包：建立在本机能连接外网的基础上，需要使用到Linux的wget命令；
他机传递安装包：常用于已经有了安装包或者需要安装软件的机器不能访问外网，通常需要使用到FtP工具；

下边以一个刚安装的纯净版CentOS系统中安装Redis为例，记录wget安装方式的过程及各操作细节分析。（此描述可能不准确，wget实际只是下载安装包）
使用wget方式的原因，是因为能联网的情况下，这种方式比他机传递更方便，而yum方式不是太熟，也不清楚是否能用yum安装Redis。

## yum安装wget
wget下载安装包，需要系统有这个命令，而这个刚安装的系统中还没有wget命令，就需要先安装一下，这里采用的就是yum方式：
```
yum install wget
```

注：yum是什么？yum是RedHat、CentOS等Linux系统中Shell前端软件包管理器，可以管理一些基础软件的安装、查找和删除，这是一个系统自带的功能。更详细的只是可参考如下博客：
https://blog.csdn.net/shuaigexiaobo/article/details/79875730

## wget下载Redis安装包
wget下载安装包，需要先知道安装包的下载地址，对于redis来说，可以在官网中找到。
Redis官网地址是：https://redis.io/，中文版的官网地址是：http://www.redis.cn/，不管哪个，打开首页都能看到Download或者下载选项，在下边就能看到下载链接，右键选择复制链接即可，我复制到的链接如下：
http://download.redis.io/releases/redis-6.0.5.tar.gz

有了链接之后，就可以使用wget下载了：
```
wget http://download.redis.io/releases/redis-6.0.5.tar.gz
```
## tar解压安装包
tar本身功能是用来文件归档的，所以其实并不具备压缩和解压功能，之所以说使用tar解压安装包，是因为它能够调用系统的解压和压缩软件，实际上就是后边参数的作用。
tar部分参数解释如下;
>-v --verbose 显示详细的tar处理的文件信息
-f --file 要操作的文件名
-x  --extract, --get 解压文件
-z --gzip, --gunzip, --ungzip      通过 gzip 来进行归档压缩

上边只列出了四个参数，其实也是我一直习惯用的，只是之前一直只是用，也没有想各个参数的含义。
那么其实从上边的解释来看，z和v实际都不是必要的，而且z是指特定的压缩格式，或许加了反而会导致某些格式的压缩包解压失败。
所以，这里的解压命令如下：
```
tar xf redis-5.0.5.tar.gz
```

注：若想了解更多tar知识，可参考后边的博客：https://blog.csdn.net/u014642915/article/details/86579575

## 帮助文档readme
大部分软件包解压后都会一个Readme文件，实际上不论是安装还是使用，都可以从这里得到很多帮助和指引。
拿我安装Redis和mongodb来说，Redis里很详细的说明了如何安装和基本使用，mongo虽然没有那么详细，但是里边也指明了安装需要去看哪里的文档。
所以这个readme文件应该是最权威的新手指引文档之一了。
对Redis来说，从readme文件里，就能看到源码安装实际就是一个`make`命令。

## make
make实际可以看做是一个构建工具，常用语c语言编写的程序源码构建安装某个软件，然是它不限于c语言，而这里redis源码是用c语言编写的，所以使用make。
make执行的根源，其实是执行目录下的一个Makefile文件，make会根据这个文件里的内容做一些事情。
而在redis源码目录下，Makefile文件直接是又执行了另一个Makefile文件，打开这个Makefile文件会发现指向了src目录下的Makefile，而这个Makefile里的内容才是make真正要做的事。

## cc
上边说redis源码是c语言编写的，因此对于这个源文件的编译，就需要c语言的编译器，从Makefile文件也能看出来有执行cc，所以对于新安装的Linux系统来说，直接执行make一般是会报错`cc: command not found`.
报错是因为没有c语言编译器，所以就需要再先安装c语言编译器，而常用的可能就是gcc，g是GNU的意思，可理解为就是自由和开源，gcc安装命令如下：
```
yum install gcc
```

## make distclean
安装好gcc以后，重新执行make可能会报错，原因是上一次的make会有一些错误残留文件，因此需要执行清理动作，即README.md里也能看到的`make distclean`命令。
清理动作完成后，重新make，一般就能够把源码安装包构建成可运行的软件。像这里的redis，就会在src目录下创建一些redis相关的可执行文件，例如redis-server、redis-client等。直接在此目录执行相应的执行文件，就可以初步使用redis了。

注：如果是最新的安装包，可能还要注意gcc的版本问题，可能因为gcc版本过低，导致make依然报错，那就需要升级gcc。

## make install
上述内容虽然已经可以使用redis了，但是每次启动redis以及使用redis相关可执行程序，都要到这个目录，或者指定绝对路径，就显得很不方便。
因此，很多这种软件都会做成系统的服务，即service。
根据README.md文件说明，redis安装成服务有两步，一步是`make install`，一步`install_server.sh`。
`make install`如果没有其他参数，就会默认安装到`/usr/local/bin`目录，如果想要自定义路径，就可以加上“PREFIX”参数，例如`make install PREFIX=/opt/soft/redis`，这一步其实就是做了一个数据移动，会把src目录下的可执行文件移动到默认或者指定的目录下，相当于把安装后的可执行和原本就有的非可执行程序做了一个分离。

## /etc/profile系统配置
在windows安装过低版本jdk的可能知道，需要自己安装完之后配置一个jdk的环境变量。环境变量配置的目的，一方面是有的软件默认需要加载这个内容，不配置可能导致运行问题，另一方面也是减少手动指定的工作量。
我们这里需要配置redis环境变量，目的就是在智行`install_server.sh`的时候减少一些输入。
根据我上边的步骤，在这个系统配置文件末尾可以加上如下配置：
```
export REDIS_HOME=/opt/soft/redis
export PATH=$PATH:$REDIS_HOME/bin
```

## source重新加载系统配置
上边配置文件是静态的，修改之后不会立即生效，所以需要执行`source /etc/profile`以使配置生效。

## install_server
有了上述环境变量的配置后，再回到redis的utils目录，就可以执行`install_server.sh`了，对于默认配置的redis来说，这时候就会把redis安装到系统的service服务中。
后续在系统任何文件目录下都可以执行`service ** start`、`service ** status`、`service ** restart`等命令操作相关服务。
上边“**”我可以简单理解为服务名，实际上对应的是`/etc/init.d`目录下的一个脚本文件，像这里使用默认配置安装的redis服务，就会在init.d目录下生成一个`redis_6379`脚本。
所有linux系统的服务，都会在`/etc/init.d`目录生成一个相应的脚本文件，这算是一个linux常识。
上边的命令，其实做了很多事，把redis安装成系统服务的同时，还会直接启动这个服务，在本机上我们就可以进行一定的redis操作了。

## bind
默认的redis配置文件，里边有一行`bind 127.0.0.1`，指向本机，所以对于虚拟机来说，如果想要在外部windows系统中使用redis客户端工具连接就连接不上，所以需要修改一下。
可以改成具体的机器ip，也可以改成`bind 0.0.0.0`，这样任意机器就都可以访问了。
至于bind具体用法，可以参考下边博客：
https://blog.csdn.net/cw_hello1/article/details/83444013

## 防火墙
上边说修改了bind后，windows就可以使用redis客户端工具连接了，其实也不对。因为默认开启的Linux系统是开启了防火墙的，那么redis的默认6379端口不在这个防火墙允许的端口内，所以外部依旧无法访问，这就需要进行处理。
处理方式有多重，可以是配置防火墙，把需要的端口加进去；可以是关闭防火墙。而关闭防火墙又可以是永久性关闭和临时关闭。
通常我个人都是使用的临时关闭，命令如下：
```
service iptables stop
```

至此，redis算是正式安装完毕，相应环节中有许多通用内容，也是对基础知识的一个良好补充。相信有了这些知识后，后边安装mongo也好，nginx也好，应该都能节省很多不必要的麻烦。