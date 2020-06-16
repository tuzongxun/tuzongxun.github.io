---
title: vue前端项目搭建和要点记录
date: 2020-3-1 14:09:42
tags: [业余项目,tzxblog]
toc: true
categories: 业余项目
---
vue-cli2和vue-cli3（vue-cli4）的一些区别

据我目前的了解，创建vue项目，vue-cli不是必须的，但是在实际开发时，几乎就是必须的。vue-cli既可以看做是一种创建vue项目的快捷工具，也可能理解成是vue的一种规范。
由于本机的vue-cli安装较早，还是vue-cli2的版本，而如今早已经是vue-cli4，所以也是时候进行一定的升级了，升级过程以及升级之后的部分区别记录如下：
<!--more-->

vue-cli安装
```
v2：`npm i -g vue-cli`
v3/v4：`npm i -g @vue/cli`
```

创建项目
```
v2：`vue init projectName`
v3/v4：`vue create projectName`
```

启动项目
```
v2：`npm run dev`
v3/v4：`npm ren serve`
```

端口修改
vue-cli创建的项目，默认端口号是8080，一般正式开发都不会直接使用这个默认端口号，了解java后端开发的就会知道，tomcat的默认端口号也是8080，通常也一样都需要修改一下。
在vue-cli2中，端口号修改起来可能相对更简单，直接在项目的config目录下能找到index.js文件，然后修改里边的port属性即可。
但是vue-cli3创建的项目整体结构不同，也没有了config目录，他的端口号修改显得比较隐蔽，需要一层层找到如下文件，然后修改里边的端口号：
node_modules\@vue\cli-service\lib\commands\serve.js

```
const defaults = {
  host: '0.0.0.0',
  port: 8080,
  https: false
}
```

vue的一些基础用法
组件安装
在我目前的需求中，除开创建项目时选择的router之外，已知需要的就还有axios和vant的组件依赖。对于技术选型，axios用来在vue中进行ajax请求，vant用作ui设计。
作为后端为主的前端菜鸟，现在用过的ui技术只有element-ui、bootstrap、vant，有的还只是用了一两下，早就忘到了千里之外，所以这次也是希望可以选择一个主修，最终就锁定在了vant上。
组件安装命令如下：
```
cnpm i vant -S
cnpm i axios  -S
cnpm i babel-plugin-import -D
```

上边最后一个的作用是为了vant的组件按需引入，查看vant官网会有说明。
额外提一点的就是，安装了cnpm之后，安装起依赖确实比之前的npm快了非常非常多，尤其是如今身在农村之能手机热点联网的情况下。

组件使用
组件的安装我理解为就是java里边maven下载jar包，那么和java一样，在使用的地方也需要引入。
axiox的引入，是在main.js中：
```
import axios from 'axios'
Vue.prototype.$http=axios
```

就目前的理解，上边的代码，第一行是必要的，第二行不是必须，写法也不一定非要这样，应该是一种习惯。我跟着教程学，也就按照这样写了。

vant的引入，和axios不太一样，不是main.js，而是router目录中index.js下：
```
import { Tab, Tabs } from 'vant';
import { Sidebar, SidebarItem } from 'vant';
Vue.use(Sidebar);
Vue.use(SidebarItem);
Vue.use(Tab);
Vue.use(Tabs);
```

这个地方我其实还不是太明白，因为一开始ui组件的引用也放在了main.js中，但是页面却不显示，挪到了index.js中之后就可以正常使用，希望在后续学习和敲代码应用的过程中能进一步弄清楚。

组件（component）和视图（view）的区别
在vue-cli3或者vue-cli4中，项目目录有components和views这两个目录，就目前的理解来说，里边的代码用法基本一样。
view可能更侧重于整体的页面，而compenent则是零散的局部的一些，从一开始的学习中来说，可以尽量规范的划分，但是也可以都放在一个目录下，并不影响实际的功能。

关于this和_this
this是一个关键字，代表的是当前对象，在java中也是有的。
而vue中的this，有一个用法可能需要记录，那就是在某些地方使用之前可能需要把this先赋值给另一个变量，比如_this。
_this应该是一个普遍的写法，却不是硬性的规定，也可以叫其他的名字。
它的出现，是因为在一个视图或者组件中，可能会有非常多的对象，比如点击事件等，这时候很容易分不清当前的this代表的是哪个，从而加大错误率和调试难度。

静态资源
不论是真是项目，还是学习时的demo，总是少不了一些静态资源，在vue-cli4中，静态资源需要一般放在public目录下，例如可以在public下创建一个css文件夹存在css样式文件，可以在public下创建一个img文件夹存在图片等资源，也可以创建一个json目录存放模拟数据的json文件。
当然了，这些命名都不是必须的，也都可以自定义，只要使用的时候保持一致即可。

使用axios进行http请求
http请求有get、post、put等，最长用的是get和post，这里就先以此记录axios的语法：
```
this.$http.get(url,{params:{paramName:paramValue}}).then(function(res){
	console.log(res);
})
this.$http.post(url,{data:{paramName:paramValue}}).then(function(res){
	console.log(res);
})
```

以上是基础用法，也是单请求的简单示例，但实际上很多时候一个功能的最终结果是需要多个请求处理后才能实现，这就还需要借助spread：
```
this.$http.all([this.$http.post(url1),this.$http.get(url2)]).then(
	this.$http.spread((res1,res2)=>{
			console.log(res1);
			console.log(res2);
		}
	)
);
```

以上不论是get、post单请求，还是合在一起的复合请求，都用到了`$http`，可能会有人有疑问。这里实际上是因为上边所说的引入了axios后，加了`Vue.prototype.$http=axios`这一行，所以这里$http代表的就是axios，也算是一种习惯性写法。

# 项目地址
项目代码和文档均以github托管，地址如下：
<https://github.com/tuzongxun/tzxblog>