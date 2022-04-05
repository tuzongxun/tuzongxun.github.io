---
title: jenkins初步理解及参数化构建
date: 2022-3-18 10:33:58
categories: 运维 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [运维,jenkins]
---
## jenkins构建和插件的初步理解
就我目前了解到的，一个jenkins自动化部署，至少包含拉代码、编译和打包及运行单元测试、部署这几个步骤。
拉代码指的是从代码托管服务器下载代码，编译和打包及运行单元测试，实际上是项目构建工具的功能，例如maven、gradle。
根据个人理解，jenkins不安装插件，也能够完成基本的自动化部署，因为它本身就支持运行shell脚本，支持参数化构建，支持多种触发器等。只要能运行shell脚本，那么不论是拉代码，还是构建和部署，都是可以通过linux的命令完成的。（这一段没有经过验证，也基本不会这样使用）
但是，jenkins本身就是为了自动化，为了简化我们的工作，所以如果有能够进一步提升效率、简化操作的插件，当然是能用就用了，例如常见的git插件、maven插件等。
git基础插件，在系统管理的插件管理中搜索`git plugin`。
maven基础插件，插件名称是`Maven Integration plugin`。
当然了，刚安装的jenkins一般都是英文版，可能大部分人都还会安装汉化插件，也就是`Localization: Chinese (Simplified)`。
<!--more-->
## 参数化构建和git增强参数化构建
jenkins一开始就带有`构建自由风格项目`这个功能，当安装了maven插件后，还能构建`maven风格项目`。
不论是构建自由风格的项目，还是构建maven风格的项目，在配置那里都能使用基础的参数化构建功能，如图：
![请添加图片描述](https://img-blog.csdnimg.cn/e4792b1eb4f3436fb8ec438ac9be3ea3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16)
不使用参数化构建功能时，只有`立即构建`的选项，当使用了参数化构建之后，原本的`立即构建`就会变`Build with Parameters`。
例如我使用参数化构建，创建两个参数，一个字符参数，一个文本参数，使用`Build with Parameters`之后就不会像之前一样直接就开始运行，而是跳转到填写参数的界面，如图：
![请添加图片描述](https://img-blog.csdnimg.cn/49d1f9fcc7bd4402960b4d10df0bc92c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16)
填写参数后，点击`开始构建`才会正式构建。
但是基础的参数化构建基本都是要手动填写或者设置，例如常见的git的branch选择，可能需要手动设置branch的名称，为了进一步简化，就有git参数化构建的增强插件，例如`Git Parameter`，装了这个插件后，会自动拉取 git的branch，然后选择对应的就好，少了填写和手动修改的动作，安装了`git parameter`后，我把上边的branch改为git参数，如图：
![请添加图片描述](https://img-blog.csdnimg.cn/b2fd03031ee245eba01e6c1164cdd5ee.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16)
然后`Build with Parameters`后，界面如下：
![请添加图片描述](https://img-blog.csdnimg.cn/f0f8cb3bf9054c77880b0fe6c5e658ef.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16)
会看到提示git残酷配置错误，原因是没有指定git的路径，那么增加git仓库和用户凭证配置：
![请添加图片描述](https://img-blog.csdnimg.cn/82b68aa877e94dfe8ea3ad902764d77d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16)
注意这里的分支使用了变量`${branch}`这样的写法，`branch`就是在上边参数化构建那里指定的参数名。
有这个配置后，再`Build with Parameters`就可以看到自动刷新了git分支供选择：
![请添加图片描述](https://img-blog.csdnimg.cn/bc346c5320b845938b253397b6eded77.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16)
选择对应的分之后点击立即构建，也会正确识别到branch然后执行任务：
![请添加图片描述](https://img-blog.csdnimg.cn/f8a1d40ca8f8416790f9c1f1ed2fb627.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16)
git和maven等插件，让我们的jenkins构建变的更加简单方便，而参数化构建能让构建变得更加灵活。
这些常用插件和基础功能远不止这些，甚至有的功能还会有多个插件可选，但是基本网上都能搜到，可以再实际需要的时候一一了解。