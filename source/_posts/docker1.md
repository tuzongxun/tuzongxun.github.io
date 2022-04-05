---
title: centos7安装docker及docker基础操作
date: 2020-9-3 19:59:42
tags: docker
toc: true
categories: docker
---
之前使用腾讯TSF时，接触了一点docker相关的内容，但只是用了一点点，由于各种原因而没有好好了解，但是这件事始终都在心里搁着，现在总算可以正式来了解一下了。

<!--more-->

## centos7网络配置
根据docker官网安装文档的说明，使用centos系统安装需要版本是centos7以上，而之前学习redis和hadoop时，我为了方便就使用了很久以前保存的一个centos6.5系统，所以不得不再安装一个centos7的系统。
centos系统安装可以选择在线升级，据说6.5的只能先升到7.2，不过应该也够用了。但是出于对当前网速的不信任，我决定还是下载系统镜像重新安装，于是直接从其他同事那里拿来了centos7.8的安装包。
安装过程和centos6.5的大同小异，只是后边网络配置的时候，发现网络配置文件和centos的有所出入，这里简单记录。
centos6.5中之前是配置`/etc/sysconfig/network-scripts/ifcfg-eth0`这个文件，而centos7.5已经没有了这个文件，于是找到了相似的另一个文件进行配置，即`/etc/sysconfig/network-scripts/ifcfg-ens33`
至于具体配置细节，和centos6.5基本一样，可参考之前的内容：
[VMware虚拟机Linux系统NAT模式网络配置及虚拟机克隆要点](https://blog.csdn.net/tuzongxun/article/details/107031865)

## yum换源及更新
由于是全新的虚拟机，再次出于对网络的不信任，所以第一时间把yum源改成了阿里云的并进行了升级，大概操作如下：
```
yum install -y wget
cd /etc/yum.repos.d
mv CentOS-Base.repo CentOS-Base.repo.old
wget http://mirrors.aliyun.com/repo/Centos-7.repo
mv Centos-7.repo CentOS-Base.repo
yum update -y
```
这里要补充一点的是，据说yum操作实际会就近选择，所以更换yum源和不更换究竟有没有区别还是待定，我没有验证过。
还要补充的是，有些不太熟悉linux操作而又习惯于c/v操作的，可能会把上述几行一起复制粘贴执行，实际上上边每一行都是一个单独的操作。

## docker安装
以上操作算是基本环境准备，但是为了更方便的安装docker，还需要一项准备，需要借助`yum-utils`：
```
yum install -y yum-utils
```

然后，配置docker软件源的地址：
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
上边的url是docker官方的，指向的也是国外地址，但是我实际操作时还是挺快，如果网络更差，则可以选择下边这个阿里云的:
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

指定安装源之后，就可以使用yum安装docker了：
```
yum install docker-ce
```

## docker基础使用
上边操作就算初步装好了docker，非常简单，我之前试着在windows中安装过一次，最终没有成功，相对来说，linux中docker的安装实在是比windows中容易多了。

### 启动
安装好之后，就可以启动docker服务进行相关的验证，作为一个linux服务，docker的启动和其他linux服务启动方式一样，如下两个选择都可以：
```
service docker start
systemctl start docker
```

### 停止
同样的，作为linux服务，docker服务的停止也是linux的通用操作，可以使用如下命令：
```
service docker stop
systemctl stop docker
```
注：可能也有人更喜欢`kill -9`，如果只是自己用的话，也没啥关系。

### 镜像和容器操作
docker服务启动后，就可以使用一些基础的docker命令使用docker，例如image镜像的操作：
```
docker image ls
docker image rm imageName
docker image pull library/hello-world
docker image pull hello-world
```
上边第一个命令和第二个分别是列出镜像和删除镜像，和linux文件系统以及hdfs文件系统的操作几乎都一样。第二个和第三个都是拉取一个远程镜像到本地，作用是一样的，之所以有两个，是因为docker默认的镜像组是library，所以可以省略，如果是其他的镜像组，就必须要显示的指定，不过，运行镜像时可以自动抓取image，所以其实不用先pull也是可以的。

镜像文件是静态的，并不能提供服务，就类似java开发好的系统生成的jar或者war，只有运行之后才会提供服务或者说处理一些逻辑，要运行一个镜像，需要使用如下命令：
```
docker container run imageName
````

以上命令指定镜像启动，每次都会运行一个新的实例，如果只是要启动已经运行过的实例，则可以使用如下命令：
```
docker container start containerId
```

对应的，有启动就会有停止，可以使用如下两个命令：
```
docker container stop containerId
docker container kill containerId
```
上边两个不同的是，前一个相对安全，后一个会强制停止容器，就类似`kill -9`。

image镜像是一个文件，容器运行后也会生成一个文件，查看操作和image类似：
```
docker container ls
docker container ls --all
```
上边第一个只会显示运行中的容器，后一个可以显示所有的容器，当使用上边命令查看到具体容器信息后，如果想要删除不再使用的容器实例，则可以使用如下操作删除：
```
docker container rm containerId
```

除了上边的基本启动、查看和删除外，对于docker运行情况，也可以使用命令查看运行日志：
```
docker container logs containerId
```
docker是容器，本身是虚拟机升级后的linux容器的一种，所以容器里边有很多东西，也支持进入容器操作，进入某个运行中的容器，基础命令如下：
```
docker container exec -it containerId
```

以上内容参考：
https://www.cnblogs.com/nhdlb/p/11569191.html
http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
https://docs.docker.com/app/working-with-app/
https://www.cnblogs.com/wang-yaz/p/10429899.html