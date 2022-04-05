---
title: idea常用功能笔记
date: 2022-2-28 16:43:58
categories: 开发工具 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [开发工具,idea]
---
工欲善其事，必先利其器，在idea中有很多功能都可以提高做事的效率。
这些功能太多，很难全部都记住，以下是自己使用过程中的一些记录，方便换电脑时进行参考。
<!--more-->
## 一、常用设置
### 1、进入设置界面
`File-》Settings`或者在显示工具栏的情况下点击工具栏的小扳手或者`Ctrl+Alt+s`快捷键调出。

### 2、调出工具栏
可以通过`View-》Appearance-》ToolBar`显示和关闭工具栏。

### 3、鼠标滚轮调整编辑区字体大小
在`Settings`界面通过`Editor-》General中`勾选`Change font size with Ctrl+Mouse Wheel`启用或关闭。

### 4、鼠标悬浮提示
如果要提示或者取消提示，不同版本的设置略有区别。
有的版本需要在`Settings`的`Editor-》general`中勾选`Show quick documentation on mouse move`；而有的版本这个`Show quick documentation on mouse move`在`Settings`的`Code Editing`中。

### 5、自动导包和删包
java写新代码后经常需要导包，可以使用快捷键，例如默认的`Alt+Enter`，但是也可以设置成更方便的自动导包。
自动导包可以通过`Settings`中`General-》Auto Import`中勾选`Add unambiguous imports on the fly`来设置。
除了自动导包，在删除一些代码的时候，之前导入的包也可以设置成自动删除，通过`Settings`中`General-》Auto Import`中勾选`Optimize imports on the fly（for current project）`来设置。

### 6、行号和分隔符
显示和关闭行号，通过`Settings`中`General-》Appearance`中勾选`Show line numbers`设置，显示和关闭方法分隔符通过`Settings`中`General-》Appearance`中勾选`Show method separators`设置。

### 7、代码提示大小写敏感
通过`Settings`中`General-》Code Completion`中勾选`Match case`启动或者停用。

### 8、省电模式
开启了省电模式时，代码提示等功能会失效，通过`File-》Power Save Mode`可以停止。

### 9、自动生成序列化versionId
通过`Settings`中`Editor-》Inspections`中勾选`Serializable class without ‘serialVersionUID’`设置，然后`implements Serializable`的类使用`Alt+Enter`时根据提示选择就可以自动成成versionID。

## 二、默认常用快捷键
### 1、向下复制一行
`Ctrl+D`复制光标所在的行。

### 2、删除一行
`Ctrl+Y`，两个键距离有点远，但是功能常用，一般都会修改掉。

### 3、生成toString等代码
`Alt+insert`会弹出选择生成构造方法、toString、equeal等进而快速生成。

### 4、导包或生成变量
`Alt+Enter`可以手动导包或者生成变量，

### 5、代码块包围
`Ctrl+Alt+t`可以弹出选择界面，选择将选中的代码放到if或者for或者try catch里。

### 6、代码格式化
`Ctrl+Alt+L`

### 7、生成UML
`Ctrl+Alt+Shift+u`

### 8、字母大小写转换
`Ctrl+Shift+u`

### 9、快速查找java class
`Ctrl+n`

### 10、快速查询方法
`Ctrl+Alt+Shift+n`

### 11、快速查找所有类型文件
`Ctrl+Shift+n`

### 12、从整个project中查找
连按两次`Shift`

### 13、新建文件
`Ctrl+Alt+insert`

### 14、查询方法哪里被调用
`Ctrl+Alt+h`

### 15、查看类的大纲
`Ctrl+F12`

## 三、代码模板
### 1、快速写main方法
`psvm`可以快速写出man方法，也可以用`main`回车。

### 2、控制台打印（标准输出）
`sout`能快速生成`System.out.println()`，如果是要打印已知的内容，可以在内容或者变量后边直接调用，例如`"aaa".sout`回车后会生成`System.out.println("aaa")`，如果是有个变量叫a，则`a.sout`回车会生成`System.out.println(a)`。

### 3、生成变量
上边快捷键那里说`Alt+Enter`之后可以生成变量，但是生成变量也可以调用var，例如`new Date().var`回车后会自动成为`Date date=new Date()`。

### 7、for循环快捷生成
`fori`回车可以快速生成`for (int i = 0; i < ; i++) {}`这种代码，如果是已知的集合循环，可以直接在变量后边调用，例如有一个集合变量名是list，则list.for回车会生成如下代码：
```
for (User user : list) { 
}
```

list.fori会生成如下代码：
```
for (int i = 0; i < list.size(); i++) {
}
```

### 8、其他代码模板
上边根据代码模板快速生成代码，实际分两种，一种是使用"."来调用，一种是直接写，都是可以在Settings里查看的。
通过"."调用的，在`Settings`里`Editor-》General-》Postfix Completion`中可以查看详情，这里的是系统定义的，没法修改。
直接写的这种，在`Settings`里`Editor-》Live Templates`中可以查看详情，还可以增加和修改。