---
title: centos7中jenkins安装和验证
date: 2022-3-18 10:13:58
categories: 运维 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [运维,jenkins]
---
# 前言
以前了解过`jenkins`，也大概知道做什么的，但是由于有专门的运维，因此并没有太多实际使用。
但是作为`DevOps`中比较重要的一个工具，还是有必要加强这方面的能力，因此就在本地虚拟机安装了一个进行相关的学习。
<!--more-->
# 安装
## 环境
虚拟机系统是`centos7.5`，安装的jdk是`java1.8.0_261`，虚拟机ip是`192.168.19.199`。


## 安装包下载
jenkins安装包有很多种，这里选中比较简单的war包。
```
wget https://get.jenkins.io/war-stable/2.303.1/jenkins.war
```

## 启动
```
java -jar jenkins.war
```

需要注意的是，第一次启动时有一串密码，在稍后登陆的时候需要用到。

## 访问
第一次访问时，需要输入启动时的那串密码，同时要设置用户名密码，还会提示安装初始推荐的插件，可选择不安装，我这里就没有安装。访问地址`http://192.168.19.199:8080/`

## 插件安装源更换
插件安装可以在线安装，也可以离线安装，为了简单，我这里就选择在线安装。
同时，为了加快安装速度，我修改了插件源，修改成了`https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`

## 服务器安装maven和git
据我目前的了解，jenkins在DevOps中的一个重要作用就是自动化部署，涉及到的基础依赖就是git和maven，所以我打算先安装maven和git插件，如果实际用的时候不需要git也不需要maven，那么也就不需要安装。
再由于插件实际只是一个软件功能的集成，而不是软件本身的功能，所以还需要在jenkins服务器上安装maven和git。

### maven安装
#### 下载
```
wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
```

#### 解压并配置settings.xml
这里我直接复制了windows中之前使用的maven的`settings.xml`

#### 配置MAVEN_HOME
修改`/etc/profile`文件，加入`MAVEN_HOME`的环境变量，并执行`source /etc/profile`加载环境变量。

### git安装
git安装使用了更简单的方式，直接使用yum安装，命令如下：
```
yum install -y git
```

## jenkins安装git和maven插件
有了上述基础后，就可以在jenkins的界面上在线安装maven和git的插件了，搜索git和maven就可以，会把相关依赖的其他插件都一起安装，安装完后重启jenkins。

# 验证
为了验证jenkins和git插件都确实安装成功，创建一个简单的item进行测试。

## 新建项目
我没有装中文版插件，英文版就是`New Item`，然后输入一个名称，选择自由项目，英文版`Freestyle project`。

## 配置github地址
我在github上有一个简单的springboot项目，只有一个helloworld的接口，这里就指向这个project。在`Source Code Management`这里配置git的`Repository URL`为`git@github.com:tuzongxun/base-springboot.git`。
需要注意的是，这里似乎只能使用git协议的，如果使用https的，即使用户名密码正确，后边从github拉代码时也会失败。

## 配置ssh key
使用git协议的时候，需要配置ssh key。在jenkins服务器执行`ssh-keygen`就会生成公私钥对，例如`id_rsa` 和`id_rsa.pub`。
pub结尾的是公钥，需要配置到github中。
没有pub的是私钥，需要在上边jenkins中配置github的url那里配置，要注意类型选择`SSH Username with private key`，然后复制id_rsa中的内容粘贴进去。

## build
配置好之后点击`Build Now`进行构建，稍等一会儿后执行成功，在console中看到如下日志：
```
Console Output
Started by user 涂宗勋
Running as SYSTEM
Building in workspace /root/.jenkins/workspace/base-springboot
The recommended git tool is: NONE
using credential 3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2
 > git rev-parse --resolve-git-dir /root/.jenkins/workspace/base-springboot/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url git@github.com:tuzongxun/base-springboot.git # timeout=10
Fetching upstream changes from git@github.com:tuzongxun/base-springboot.git
 > git --version # timeout=10
 > git --version # 'git version 1.8.3.1'
using GIT_SSH to set credentials 
[INFO] Currently running in a labeled security context
[INFO] Currently SELinux is 'enforcing' on the host
 > /usr/bin/chcon --type=ssh_home_t /root/.jenkins/workspace/base-springboot@tmp/jenkins-gitclient-ssh8748129526411728188.key
 > git fetch --tags --progress git@github.com:tuzongxun/base-springboot.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
Checking out Revision 19065451d6cd9890de4b751c0e7af06de6628ec5 (refs/remotes/origin/master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 19065451d6cd9890de4b751c0e7af06de6628ec5 # timeout=10
Commit message: "init"
First time build. Skipping changelog.
Finished: SUCCESS
```

可以看到成功从git中拉取到了代码，证明jenkins和git插件安装没有问题，至于其他功能，待后续再进一步学习和验证。