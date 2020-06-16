---
title: 前后台打通之vue和vant前端要点及设计问题记录
date: 2020-3-13 18:09:42
tags: [业余项目,tzxblog]
toc: true
categories: 业余项目
---
接着上一次的后台要点记录，这次再总结一下前台的要点，同时对于整体搭建过程中的一些感想也一并记录。
相对于后台来说，前台是弱项，也不是目前的本职，所以前台的记录可能就更基础。

<!--more-->
# 前端要点记录
## vant组件使用
除开vue本身的使用之外，这一次前台的重点应该就是vant了，属于ui层面的这一块技术一直是弱项中的弱项。
使用vant之前，需要先安装vant依赖，网络好的情况下可以使用`npm`，如果网络不好，最好就还是使用`cnpm`，操作如下：
```
cnpm i vant -S
```

依赖安装之后，正常来说就可以在代码中使用相应的组件了，使用之前需要引入。引入的方式有很多种，但是为了性能更好以及避免大量无用组件的引入，我根据官网说明选择自动按需引入的方式，这个方式就需要再安装一个插件，操作如下：
```
cnpm i babel-plugin-import -D
```

依赖安装好之后，就可以敲代码了，拿导航栏来说，我这里选择的是标签页样式作为导航，就可以在js中按如下方式引入：
```
import { Tab, Tabs } from 'vant';
Vue.use(Tab);
Vue.use(Tabs);
```

有了上边的引用，那么在需要设计导航栏的地方就可以使用vant组件了，我的这个功能写在TopNav,vue中，可以前往具体代码中参考使用，代码连接见文章末尾，标签页使用如下：
```
<van-tabs v-model="active">
  <van-tab title="首页" name="a">内容 1</van-tab>
  <van-tab title="下载" name="b">内容 2</van-tab>
  <van-tab title="登录" name="c">内容 3</van-tab>
</van-tabs>
```

上边的内容就引出了一个vue对象绑定的知识点，即`v-model`，这里绑定了一个变量active，所以就需要在js中进行声明：
```
export default{
		data(){
			return{
				active:1
			}
		}
	}
```

以上便是vant组件的基本使用记录，对着官网的说明，使用起来应该说比较简单，这里为vant官网点个赞。[vant官网](https://youzan.github.io/vant/#/zh-CN/)
那么这次的博客截止到目前，已经使用了十几个组件，虽然整体手撸的页面依旧不是那么好看，但是比起之前写的一些来说，还是要好看多了，以下是博客展示页的截图（首页截图可以继续往下翻）：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031321174787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3R1em9uZ3h1bg==,size_16,color_FFFFFF,t_70)

## 前台模块间传值
除了借助vant来设计界面之外，数据交互就还是要回到vue本身了。
像基本的数据绑定、属性绑定、点击事件等应该算是较常规的知识点，这里就不做记录，要特别记录的就是模块间数据传值。
不论是在首页点击分类改变首页博客列表，还是点击首页博客列表进入博客浏览，都需要用到这种模块间的传值。
这种传值，首先需要定义一个js文件，例如Msg.js，名称不一定要这样，js里的内容很简单：
```
import Vue from "vue"
export default new Vue
```

有了这个文件之后，在需要传值的地方就可以引入之后进行使用。
例如我需要在首页分类列表点击某个分类时，传递分类id以改变博客列表，那么在js里首先要引入上边的定义：
```
import Msg from './msg.js';
```

之后在需要传值的地方就可以使用如下代码传值：
```
Msg.$emit("cateId",cateId,1);
```

像我这里，就是在分类列表有一个点击事件，点击事件会调用定义的方法，那么就可以在该方法里使用上边的代码传值。
就目前我的理解是，上边第一个参数是传值标示，起一个绑定作用，用以接收时候的识别，后边就是具体要传的数据，有几个就写几个，以逗号分隔。
有了上边的传值，之后就是接收，接收数据的js同样要先引入msg.js文件，引入之后接收数据的代码如下：
```
mounted(){
	var _this=this;
	Msg.$on("cateId",function(cateId,cuPage){
		_this.cateId=cateId;
	});
}
```

## v-if和v-for的使用
`v-if`和`v-for`的使用，相对比较简单，尤其是之前使用过`augularjs`，有着基本一样的用法，`v-if`后边的表达式或者变量需要是一个boolean类型的结果，`v-for`使用`in`关键字进行集合数组的遍历，例如：
```
<div v-if="type==1 || type==0">
	<div v-if="homeCotentType ==1 || homeCotentType ==0">111</div>
</div>
<div v-else-if="type==2">222</div>
<van-sidebar class="cateList" v-model="activeKey" @change="onChange">
	<van-sidebar-item v-for="cate in list" :title="cate.name" @click="choose(cate.id)">
	<van-icon name="wap-home" /></van-sidebar-item>
</van-sidebar>
```

以上两部分在BlogHome.vue和HomeLeft.vue两个文件中有所体现，后续可能也会在很多地方用到。

前端要记录的知识点其实是很多的，不是本职，不常用，就很容易遗忘，哪怕很简单的地方。
但是单从架构和项目搭建以及前后台打通来说，似乎也没有那么多需要记录的，本项目中前后台交互的关键在于`axios`，这个在最初的前端项目搭建时已经说了，也就没必要赘述。

# 部署调用
前台和后台打通之后，直观的表现形式就是浏览器可访问，并且数据是从后台读取，这就涉及到vue和springboot前后台的部署。
使用eclipse开发，springboot后台项目直接在eclipse中运行即可，根据本项目的配置，后台就会运行在8089端口上。
对于vue前台，就需要使用npm启动，在本项目中，就是在tzxblog目录下的tzxblog目录下执行`npm run serve`，那么前天就会运行在8088端口。
前后台都启动之后，浏览器访问`localhost:8088`,结果会如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200313211821965.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3R1em9uZ3h1bg==,size_16,color_FFFFFF,t_70)
# 设计问题
在先设计后开发的流程中，非常的考验经验和大局观。如果没有相关经验，或者考虑的不是那么周全，就会发现实际写代码时总会有各种地方需要在原本设计的基础上改动。
我不知道是否有那种真的一次设计，后边就完全按设计开发而不再改动的项目。
但是不论是过往的工作项目，还是现在的业余项目，几乎都是会在后续的开发中改动一定的设计，尤其是接口定义。
要改善这个问题，除了堆时间累经验，恐怕就只能是自己想办法做一些完整的项目了，如果永远都是只负责模块，那么必然无法形成整体的大局观。

# 前台和后台
对于前台和后台的概念，可能刚入行的时候总想弄个明白，随着经验多了，慢慢就不再那么执着。
但是在这一次整体项目实施的过程中，却又突然开始想这个问题，虽然可能并没有太大的意义。
何谓前台，何谓后台，可能有时候还要看站在什么角度。
曾经，我觉得java web项目中java是后台，js和html就是前台，但是随着技术的发展会发现，js和html有时候也会在本身分出一个前后台。
例如vue中，使用node.js可以启动一个服务，甚至可以不需要java这种后台语言而直接操作数据库，因此很多纯前端的同学可能就会说页面是前台，js也是后台。
虽然很多时候这种区分并没有什么影响，但是有的时候认知上的不同却也可能带来沟通上的偏差，最终后果就是影响处理问题的效率。

# 项目地址
项目代码和文档均以github托管，地址如下：
<https://github.com/tuzongxun/tzxblog>