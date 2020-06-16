---
title: vue项目搭建问题记录
date: 2020-1-28 09:09:42
tags: [前端]
toc: true
categories: 前端
---
## 前言
新型肺炎，湖北在水深火热之中，到处封路，不敢出门。
武汉回来还不满14天的我，又不敢带小孩儿，干着急也没有什么作用，索性还是学点东西吧。
本想重构一下之前的小项目，把前台的实现由thymeleaf模板改为更流行的vue，但是创建vue项目时却遇到一些问题，以下为记录备忘。

<!--more-->

## 记录一:安装cnpm卡住
按原来的步骤，使用`vue init webpack projectName`可以创建一个初始的vue项目，但是这次大概是由于家里网络不好，在`npm install`的时候一直卡着不动。
网上说是由于npm的镜像在国外，可以换成阿里的cnpm镜像，安装命令如下：
<pre>
npm install -g cnpm --registry=https://registry.npm.taobao.org
</pre>

但是在执行这个命令之后，却在`fetchMetadata`这里一直卡着不动，足足一个小时。
于是各种尝试之后，根据网上的方法成功安装，步骤如下：
<pre>
npm set registry https://registry.npm.taobao.org
npm set disturl https://npm.taobao.org/dist
npm cache clean --force
npm install -g cnpm --registry=https://registry.npm.taobao.org
</pre>

可能还是网络不好，执行了上边几步之后，依然用了差不多十五分钟才安装成功。
参考：[cnpm执行卡住，卡顿解决](https://www.jianshu.com/p/f735414b1bc4)

## 记录二：cnpm初始化vue项目
在创建vue项目的时候，最后一步有三个选项：
<pre>
Yes, use NPM
Yes, use Yarn
No, I will handle that myself
</pre>
这一步的意思就是下载安装各种依赖包，默认是npm下载，我上边创建项目卡住的原因也就是这里直接回车了。而网络不佳。
因此，安装cnpm之后，可以选择第三项。
完成之后，cd到项目目录下再执行`cnpm install`即可用cnpm代替npm下载安装依赖包，大大提升项目创建的速度以及成功率。

## 后记
湖北依旧在水深火热之中，武汉更是在水深火热的中心。
昨天一天湖北增加确诊1291人，希望是因为试剂盒、设备、医务人员跟上之后出现的正常现象。
祖国加油，湖北加油，武汉加油，希望一切都快点好起来，希望一起都快点过去！
