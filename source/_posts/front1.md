---
title: vue+element-ui搭建纯前台项目记录
date: 2019-1-29 09:09:42
tags: vue
toc: true
categories: 前端
---
## 架构说明
本次项目用于个人学习，作用于个人聚合支付demo，记录步骤，为以后作参考。
前台项目搭建的架构基础是前后台分离，即：从代码层面来说，前台和后台互不相干，不同的服务，不同的端口，前后台之前使用http协议进行交互。
前台基本技术架构为node.js+vue.js+element-ui+vue-router。
<!--more-->

## 步骤说明
### 安装node.js
本项目基于node.js搭建，需要使用npm命令，由于我之前已经安装过，因此跳过。

### 创建vue项目
vue初始项目搭建过程主要参考如下博客：
[前端架构之路：使用Vue.js开始第一个项目](https://www.cnblogs.com/dreling/p/6892684.html)
稍有不同的是，上边博文中使用的是阿里版node，所以是cnpm命令，而我用的是原版，所以用的是npm。
简单整理一下最终步骤，大致如下：
### 安装webpack
网上说webpack是一个模板打包器，实际使用时我觉得似乎说是一个构建工具更合理，使用如下命令安装：`npm install -g webpack`

### 安装vue-cli 
使用`npm install vue-cli -g`命令安装脚手架，目前理解为就是一个创建vue项目的工具。

### 创建vue项目
使用vue-cli的vue命令创建项目模板，这里实际上踩了个坑。上边博文中创建命令是`vue init webpack-simple first-vue`，我是照着敲的，一开始没有问题，但是到后边使用vue-router路由的时候各种问题出现。原因就是webpack-simple搭建的是极简版项目，如果要使用router这些功能，从依赖到目录都要自己一个个的创建。对于刚接触vue的人来说，这个过程必然频频出错，所以后来得知最佳步骤应该是这样`vue init webpack first-vue`
### 引入element-ui。
使用element-ui，先要在项目中安装element-ui的依赖，在项目根目录下执行如下命令`npm install element-ui -S`。
然后需要在项目main.js中加入这样几行：
<pre>
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
Vue.use(ElementUI);
</pre>

## 注意事项
### webpack和webpack-simple的区别
这个在上边说过，使用webpack会在项目中创建好路由相关的结构，而webpack-simple不会。

### 依赖加载
不管是初始项目后，还是后边加入element-ui之后，都是需要使用类似npm install这样的命令安装加载依赖的，就如java maven项目中import一个类或者包，必须要maven加载了相应的jar包才行。

### vue-router用法
常规的vue文件大概是类似下边的格式：
<pre>
&lt;template>
  &lt;div id="myvue">&lt;/div>
&lt;/template>
&lt;script>
&lt;/script>
&lt;style>
&lt;style>
</pre>

这个模板如果要交给router路由管理，就需要在路由中配置，如router目录下的index.js中配置，例如：
`import myvue from '@/components/myvue'`
其中，import后边的myvue是上边的id的值，components后边的myvue是components目录下的myvue.vue文件名。
然后配置路由组件,在routes中加入类似下边这样的代码：
<pre>
{
  path: '/myvue',
  name: 'myvue',
  component: myvue
}
</pre>

有了上边的操作还不够，还需要在使用上述模板的地方使用`<router-view></router-view>`标签，这样router才能正确的路由跳转。

***
## 代码参考
[涂宗勋的个人聚合支付demo](https://github.com/tuzongxun/tzx-payment)
