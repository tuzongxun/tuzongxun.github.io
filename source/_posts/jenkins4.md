---
title: jenkins pipeline部署补充记录
date: 2022-3-27 12:43:58
categories: 运维 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [运维,jenkins]
---
## 前言
最近在阿里云服务器上重新安装了jenkins，参照我之前的笔记，同时打算集成更多的常用的插件。
除了新插件的使用记录外，还遇到了一些其他的问题，觉得有必要也补充记录。
之前笔记参考：
[centos7中jenkins安装和验证](https://blog.csdn.net/tuzongxun/article/details/120048956?spm=1001.2014.3001.5501)
[jenkins初步理解及参数化构建](https://blog.csdn.net/tuzongxun/article/details/120097550?spm=1001.2014.3001.5501)
[jenkins pipeline部署实践及重点问题分析](https://blog.csdn.net/tuzongxun/article/details/120107854?spm=1001.2014.3001.5501)

<!--more-->
## pipeline中git拉取代码问题
根据之前的笔记装好jenkins和maven及git插件，并配置好pipeline后，发现并没有想象中的那么顺利，因为一些未记录的细节，导致首先在git拉取代码的时候就出现了问题。
代码拉取失败，提示'CredentialId "3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2" could not be found'。
经过一番分析后，发现这个CredentialId出现在jenkinsFile的配置中，是在我之前创建的springboot项目中，这段配置如下：
```
stage('git pull code') {
	steps {
		echo "【git pull】"
		git branch: 'jenkin-test', credentialsId: '3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2', url: 'git@github.com:tuzongxun/base-springboot.git'
	}
}
```
之所以现在这里不行了，是因为我重新安装了jenkins，里边的credentialsId实际是改变了的，所以解决办法也就是去jenkins上创建一个credentialsId是'3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2'的credentials或者修改jenkinsFile，把上边的id指向现有的。

## maven构建问题
解决了git问题之后，代码是成功的从github上拉取下来了，但是在接下来的maven构建中又运行失败，从日志中可以看到有这样的信息'package org.springframework.stereotype does not exist'。
一开始我以为是新的阿里云服务器上安装的maven有问题，导致jenkins执行的时候无法下载对应的依赖包，但是通过手动到服务器执行'mvn clean package'命令发现并没有问题。
于是又猜测是不是jenkins中maven运行路径指定有问题或者settings.xml文件路径指定有问题，但是经过查看，发现也没有问题。
最终，通过百度到的一些信息解决了这个问题，原因是，为了图方便，我是直接把windows中的settings.xml上传到了现在的服务器上。
这个文件里边的本地仓库我指向了F盘，配置是这样的：
```
<localRepository>F:\repo_new1</localRepository>
```
而这种路径不是linux标准格式，于是改成了'/opt/repo_new1'之后就成功解决了这个问题。（问题暂时解决了，不过为什么直接服务器上执行mvn命令没有问题，jenkins执行就有问题，这个还没有完全明白）
参考博客：[在 Jenkins 中，使用 maven 打包报 package xxx does not exist 问题的解决方法](https://blog.csdn.net/deniro_li/article/details/73550881)
