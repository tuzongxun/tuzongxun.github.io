---
title: VuePress搭建静态网站记录
date: 2022-4-5 11:43:58
categories: 前端 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [前端,vuepress]
---
## vuepress介绍

最近在看鱼皮直播时，了解到一个新的静态网站生成器，看起来似乎蛮好用，于是就自己也尝试了一下，它就是VuePress。

VuePress是基于vue的静态网站生成器，类似于hexo那种可以直接把markdown等格式的文件解析成静态网页，用来搭建个人网站比较方便。

可能很多人了解过hexo，它生成的静态网站，是支持本地离线运行访问的，这一点尤其是对于笔记管理类的博客就非常方便。

<!--more-->

## vuepress基本安装

### 依赖

node.js v12+

yarn v1

如果没有yarn，可以使用npm安装：

```
npm install --global yarn
```



### 初始步骤

相关步骤主要是参考官网文档[VuePress](https://v2.vuepress.vuejs.org/zh/guide/getting-started.html)

1. 创建并进入目录

```
mkdir vuepress-blog
cd vuepress-blog
```

2. 初始化项目

```
git init
yarn init
```

这一步在执行yarn的时候报错，错误信息如下：

```
Can't answer a question unless a user TTY
```

错误原因是我用的`git bash`进行操作，后来改为其他的工具就可以了，至于为什么`git bash`不行，暂时没有深究。

3. 将VuePress安装为本地依赖

```
yarn add -D vuepress@next
```

这个过程中，第一次执行时抛出了如下异常：

```
error postcss-csso@6.0.0: The engine "node" is incompatible with this module. Expected version "^12.20.0 || ^14.13.0 || >=15.0.0". Got "14.4.0"
```

从异常信息来看应该是版本有问题，最终百度也确定确实是版本问题，可以通过修改版本，也可以通过另一种配置的方式，我暂时选择了配置的方式，配置命令如下：

```
yarn config set ignore-engines true
```

做了上述配置之后，再次执行则成功。

4. 在package.json中添加脚本

```
{
  "scripts": {
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
}
```

5. 将默认临时目录添加到`.gitignore`文件

```
echo 'node_modules' >> .gitignore
echo '.temp' >> .gitignore
echo '.cache' >> .gitignore
```

这一步其实是用git管理，提交代码的时候起作用的，如果只是为了用VuePress应该是不需要的，由于我是准备用github管理的，所以也就照做了。

6. 创建markdown文件

这一步VuePress原文档中也是用的命令操作，如下：

```
mkdir docs
echo '# Hello VuePress' > docs/README.md
```

不过我原本就有一些markdown文档，包括这个笔记也是直接markdown写的，所以就直接拿来用了。

至于上边的docs目录，不一定要叫这名字，但是多数文档生成器都这样叫了，相当于一个约定俗成，所以也就不改了。

7. 启动

准备好文档后，就相当于已经在本地搭建好了最基本的一个VuePress网站了，启动命令如下：

```
yarn docs:dev
```

这个实际就是上边配置的那个脚本。

8. 预览

启动完成后，控制台会出现内外网以及本地访问的Url，浏览器访问效果如下：

![](/images/vuepress/vuepress1.png)

### 基本配置

上边的操作已经可以正常使用了，但是还缺少一些东西，例如一般网站应该都有网站标题之类的，上边就没有，这是因为还没有做对应的配置。

VuePress基础配置也很简单，[官网配置说明](https://v2.vuepress.vuejs.org/zh/guide/configuration.html)也说的很清楚，需要现在文档目录下创建`.vuepress`目录，里边再创建`config.js`或者`config.ts`文件。

不过经过实际使用发现，在前边运行过启动命令后，`.vuepress`目录实际已经自动创建了，所以只需要手动再创建`config.js`文件即可，之后我配置的内容如下：

```
module.exports = {
    // 站点配置
    lang: 'zh-CN',
    title: '程序员涂小哥',
    description: 'java全栈，现居广州',

    // 主题和它的配置
    theme: '@vuepress/theme-default',
    themeConfig: {
    	logo: 'https://vuejs.org/images/logo.png',
    },
}
```

配置之后重新启动，然后浏览器访问效果如下：

![](/images/vuepress/vuepress2.png)

可以看到在最上边出现了两个内容，一个是站点标题，另一个是可以设置背景颜色的，现在背景是白色，点击一下就可以变成黑色：

![](/images/vuepress/vuepress3.png)

VuePress配置分为站点配置和主题配置，其实可以理解成就是全局配置和只针对主题的局部配置，写在`themeConfig`里边的就是主题配置。

### 静态资源

VuePress中的静态资源，官网建议放在使用的地方比较近的位置，或者默认的`.vuepress/public`目录中，我就使用默认目录修改了logo。

把对应的log图片logo.jpg放入上述目录中，然后在`.vuepress/config.js`中修改logo为`logo: '/logo.jpg'`，刷新页面看到logo已经修改：

![](/images/vuepress/vuepress4.png)

### 页面和路由

在上边的操作中，细心的人可能会发现这只是单一的一个文档，如果是多个文档怎么办呢？

这个在[官网说明](https://v2.vuepress.vuejs.org/zh/guide/page.html)其实也讲的比较清楚，这里抄过来用一下：

> 将 `docs` 目录作为你的 [sourceDir](https://v2.vuepress.vuejs.org/zh/reference/cli.html) ，例如你在运行 `vuepress dev docs` 命令。此时，你的 Markdown 文件对应的路由路径为：

| 相对路径           | 路由路径             |
| ------------------ | -------------------- |
| `/README.md`       | `/`                  |
| `/contributing.md` | `/contributing.html` |
| `/guide/README.md` | `/guide/`            |
| `/guide/page.md`   | `/guide/page.html`   |

参考上边的规则，我创建了自己的文件结构，如下：

| 文件路径                   |
| -------------------------- |
| `docs/README.md`           |
| `docs/java/javathread1.md` |
| `docs/java/javathread2.md` |
| `git/gitbase.md`           |

### 导航配置

有了上边的准备，就可以进一步完善网站了，例如必要的导航，根据官网说明，这个应该配置在主题配置中，我的配置如下：

```
themeConfig: {
      logo: '/logo.jpg',
      navbar: [
        // 嵌套 Group - 最大深度为 2
        {
          text: '开发技术',
          children: [
            {
                text: 'java',
                children: ['/java/javathread1.md', '/java/javathread2.md'],
            },
            {
                text: 'git',
                children: ['/git/gitbase.md'],
            },
          ],
        },
        // 控制元素何时被激活
        {
          text: '资源分享',
          children: [
            {
              text: '技术类',
              link: '/',
              // 该元素将一直处于激活状态
              activeMatch: '/',
            },
            {
              text: '感悟类',
              link: '/not-foo/',
              // 该元素在当前路由路径是 /foo/ 开头时激活
              // 支持正则表达式
              activeMatch: '^/foo/',
            },
          ],
        },
      ]
    }
```

有了上述配置后，再访问页面，效果如下：

![](/images/vuepress/vuepress5.png)



## 更换主题

好了，到这里，基本上一个简单可用的本地笔记或者说博客网站就好了。

不过每个人需求不一样，有的人可能还想要更酷的效果，可能还想要页脚配置、评论等等，这些在VuePress的其他主题中会有这些功能，都可以根据文档进行尝试。

