---
title: 《maven实战》学习笔记2——maven安装（windows和eclipse插件）
date: 2017-11-03 15:09:42
tags: maven
toc: true
categories: maven
---
# 前言
由于我的工作中开发环境就是windows，IDE是eclipse，因此安装也只涉及和记录这两部分，在看书和动手的过程也就直接跳过其他部分。
<!--more-->
# 笔记

## windows中maven的安装
### 安装条件
maven依赖于java，因此安装和使用maven，要先确保已安装了jdk并配置好jdk的环境变量。
检查jdk是否安装并配好环境变量，可以在windows的cmd窗口执行java -version查看，如果如下所示，则证明jdk安装和配置正确。
<pre>
C:\Users\tzx>java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
</pre>

### 安装包下载
windows安装使用zip格式的压缩包，可从maven官网[http://maven.apache.org/download.cgi](http://maven.apache.org/download.cgi)下载.
进入上边的链接，能看到的maven最新版的安装包，如果不想要最新版，可以再页面上找到` Maven Releases History`，这是一个链接，点进去就可以看到所有的历史版本，然后选择需要的版本点击进去进行下载。

### 解压安装包
这一步相信就不需多说了。

### 配置环境变量（以下内容基于win8系统）
和java一样，maven也需要配置环境变量，找到”我的电脑“，右键然后选择”属性“，之后依次选择”高级系统设置“、”环境变量“。
然后在”系统变量“选择新建，变量名填`M2_HOME`，变量值选择maven解压后的目录，例如我这里的是`D:\maven\apache-maven-3.2.5`。
再然后，在”系统变量“中找到`Path`属性，在末尾加上`%M2_HOME%\bin`，注意在加之前如果原本`Path`变量值的末尾没有分号结尾，需要加上英文格式的分号，然后再添加`%M2_HOME%\bin`。

### 检查maven安装和环境变量的配置
和检查jdk安装和环境变量配置一样，打开windows的cmd窗口，执行`mvn -v`，如果出现如下所示类似的结果，代码环境变量配置正确。
<pre>
C:\Users\tzx>mvn -v
Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-15T01:29:23+08:00)
Maven home: D:\maven\apache-maven-3.2.5
Java version: 1.8.0_131, vendor: Oracle Corporation
Java home: D:\Java\jdk1.8.0_131\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 8.1", version: "6.3", arch: "amd64", family: "dos"
</pre>

## eclipse中安装maven插件
eclipse中安装maven插件，实际上我认为应该是属于eclipse的知识了，而且安装方式也是多种，至少我就用过两种。

### 使用Install New Software安装
打开eclipse后，在最上层菜单栏找到`help`，然后在下拉菜单中找到`Install New Software...`；
接下来，在出现的界面中点击`Work with`后的`add`按钮，在接下来的界面中，`Name`字段中输入`m2eclipse`,`Location`中输入`http://download.eclipse.org/technology/m2e/releases`,之后选择相应的模块，一步一步`next`就好了。

### 使用Eclipse Marketplace安装
Eclipse Marketplace相当于我们通常玩手机看到的各个应用市场，这里可以理解成插件市场，可以下载各种插件。
同样是找到`help`，然后就可以看到`Eclipse Marketplace`，点进去然后搜索`m2eclipse`就可以搜到maven相关的eclipse插件，安装比较简单，就不做过多的说明了。
不过这里需要说明的是，有许多版本的eclipse中实际上有自带的eclipse插件，不过这个插件好像都是最新版本，且是不稳定版本，所以可能会出现各种各样的问题，所以并不建议使用eclipse中默认的maven插件