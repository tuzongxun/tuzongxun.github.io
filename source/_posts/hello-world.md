---
title: 使用github和hexo写博客要点记录
date: 2017-9-7 08:55:29
categories: 博客搭建
toc: true  //在此处设定是否开启目录，需要主题支持。
tags: 博客搭建
---
有很多爱分享的技术人，都会在网上或多或少的写博客，一开始可能都是在那些有名气的网站上发表，例如CSDN、开源中国等，一方面是分享，一方面是作为自己的笔记。
但是时间久了就会发现在这种网站上写博客会受到各种制约和限制，如果网站本身技术不是很好或者其他原因，还会经常出现无法访问的情况。
<!--more-->
于是很多人就想拥有属于自己的博客，这样只要技术够好，想要什么样就可以要什么样。
而搭建自己的博客通常又分为花钱的和不花钱的，花钱通常指的是自己租服务器、买域名，而我这里用的就是不花钱的，使用github和hexo搭建属于自己的博客。
主要搭建过程参考如下两篇文章：
<http://blog.csdn.net/gdutxiaoxu/article/details/53576018>
<http://www.jianshu.com/p/e99ed60390a8>

因为上边两篇文章讲的已经非常详细，所以我这里就不需要赘述了，以下的一些记录仅作上述两篇的小小补充，同时方便后边写博客时查阅。
如果有朋友也想有一个属于自己的风格的个人免费博客，可以参考。

## 常用操作

### 常用操作
``` bash
hexo new "第一篇github博客" #新建文章
hexo new page "pageName" #新建页面,比如
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
```

More info: [Writing](https://hexo.io/docs/writing.html)

### 文章模板
#### 标题
    ---
    title: 使用github和hexo写博客要点记录 #标题
    date: 2017-9-7 08:55:29 #时间
    categories: 博客搭建 #分类
    toc: true  #在此处设定是否开启目录，需要主题支持。
    ---
	<!--more--> #使这一段之前的内容成为摘要

#### 生成about页面
    hexo new page "about"
上述命令会在source目录生成about目录，并在about中生成index.md文件，更改这个文件的type为"about"，如果没有type，自己加上。
书签books页面创建方法一样。
注意执行部署hexo d之前，先把浏览器访问的页面关掉，否则有时候提交不了。

### markdown常用语法

#### 区块引用
>\>这是一个区块引用
>这是一个区块引用
>这是一个区块引用

#### 无序列表
星号、加号、减号开头
>\* 列表一
>\+ 列表四
>\- 列表三

* 列表一
+ 列表四
- 列表三

#### 有序列表
数字开头，数字顺序无所谓
>1. 列表一
>1. 列表二

1. 列表一
1. 列表二

#### 分割线
在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西
>\*\*\*
>\-\-\-

* * *
- - -

#### 链接
>\[链接描述\]\(实际url\)
>\[涂宗勋的个人博客\]\(http://tuzongxun.github.io)

[涂宗勋的个人博客](http://tuzongxun.github.io)

还可以链接到本机，例如链接到关于我页面：/about/url

#### 强调
两个*对或者_包起来
>\*\*强调\*\*
>\_\_强调\_\_

**强调**
__强调__
>\*强调\*
>\_强调\_

*强调*
_强调_

#### 图片
>\!\[描述]\(url)



## 博客增加目录导航
## 一级目录
目录定位，格式："## 一级目录 "（注意“#”之后的空格）

### 二级目录
目录定位，格式："### 二级目录"

#### 三级目录
目录定位，格式："#### 三级目录"

其实就是对应的h1到Hn的html标签，这个功能我个人觉得非常好，可以做一些教程之类的，直接在后边会出来层级目录。
![目录结构](/images/mulu.png)

## 部署

### 本地部署

``` bash
$ hexo server  #可以简写成hexo s
```

More info: [Server](https://hexo.io/docs/server.html)

### 常用部署三部曲

#### clean
``` bash
$ hexo clean
```
大概就是清缓存的意思

#### generate
``` bash
$ hexo generate
```
可以简写为 hexo g,即把hexo new生成的.md结尾的文件生成为html

More info: [Generating](https://hexo.io/docs/generating.html)

#### deploy

``` bash
$ hexo deploy
```
部署上传到github服务器,可简写为hexo d

More info: [Deployment](https://hexo.io/docs/deployment.html)

