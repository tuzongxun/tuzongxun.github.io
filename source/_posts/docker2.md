---
title: 使用docker部署springboot服务
date: 2020-9-4 19:59:42
tags: docker
toc: true
categories: docker
---
上一篇，在centos7中搭建好了docker，并尝试了基础的image镜像和container容器的命令操作。
这一篇就尝试自己在docker容器中启动并使用springboot服务，基本步骤就是准备jar、配置Docker、构建image、启动container、测试服务功能。

<!--more-->
## 准备springboot的jar
为了简单演示，我创建了一个新的springboot项目，只写了一个controller接口以供查看效果，controller接口代码如下：
```
@RestController
@RequestMapping
public class TestController {
    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

确定上述代码可以正常运行并在浏览器正常访问后，生成springboot可运行的jar，我这里命名为`demo.jar`，然后把这个文件上传到linux虚拟机中准备构建docker镜像的目录，例如`/root/docker/demo`。

## 配置Dockerfile
有了jar之后，接下来就可以配置Dockerfile文件了，我这里是按这个顺序来的，因为Dockerfile里需要指定jar文件。
而实际上Dockerfile文件只是一个静态文件，在jar之前配置也没有问题，只需要后续生成的jar名字及路径和Dockerfile里的保持一致即可。
在上边jar所在目录，使用vi命令创建并编辑Dockerfile文件：
```
vi Dockerfile
```
编辑文件内容：
```
FROM java:8
VOLUME /tmp
COPY demo.jar app.jar
RUN bash -c “touch /app.jar”
ENTRYPOINT ["java","-jar","/app.jar"]
```

以上内容的含义如下：
>第一行，本Dockerfile文件生成的image镜像，基于java8的镜像；
第三行，把当前目录下的`demo.jar`文件拷贝到docker容器中，同时重命名为`app.jar`；
第四行和第五行，运行`app.jar`,即启动spingboot服务。

细心之人会发现，上边的解释没有提到第二行，这是因为这一行并不是必须的，只有需要使用到文件系统时才会用，即docker中的程序需要直接操作文件时就要配置这个。
并且，这种写法也只是docker和宿主机文件关联的方式之一，还有其他写法，后续再深入了解。

以上配置和网上很多教程写的都类似，某种程度来说，可能会给不清楚的人造成一定误解，会以为很多就是固定写法，而实际上上边很多内容都是可以有其他写法的。
首先，上边每行开头的单词都是大写，但是和使用mysql查询时的sql关键字一样，这里的关键字并不是必须大写，也可以使用小写。
其次，如果需要第二行的话，后边指向的目录也不是必须使用tmp，也可以用其他。
然后就是本地文件添加到容器的操作，可以使用`copy`,也可以使用`add`，都是需要两个参数，一个是本地文件路径和文件名，一个是docker容器中的文件名。
所以，后边的app.jar自然也是可以叫其他名字的，只需要后边使用时保持一致就可以了。
因此，上边配置的写法，也可以改成下边这样：
```
from java:8
volume /test
add demo.jar demo.jar
run bash -c "touch /demo.jar"
entrypoint ["java","-jar","/demo.jar"]
```

## image构建
有了需要运行的jar，也有了配置好的Dockerfile文件，准备工作就做好了，接下来就可以构建一个image镜像，使用如下命令：
```
docker build -t docker-spring .
```
上述命令尤其要注意末尾的点符号`.`，很容易被忽略，执行上述命令后会看到陆续输出如下内容：
```
Sending build context to Docker daemon  16.52MB
Step 1/6 : from java:8
 ---> d23bdf5b1b1b
Step 2/6 : volume /test
 ---> Running in 5533951f04ca
Removing intermediate container 5533951f04ca
 ---> 0965f253bac5
Step 3/6 : add demo.jar demo.jar
 ---> bff720047ccb
Step 4/6 : run bash -c "touch /demo.jar"
 ---> Running in 900c847c14a4
Removing intermediate container 900c847c14a4
 ---> edb32f0ef654
Step 5/6 : expose 8080
 ---> Running in c25a898a3e37
Removing intermediate container c25a898a3e37
 ---> 55bbf5e81907
Step 6/6 : entrypoint ["java","-jar","/demo.jar"]
 ---> Running in b891f3caeeb3
Removing intermediate container b891f3caeeb3
 ---> 8caaec5d617b
Successfully built 8caaec5d617b
Successfully tagged docker-spring:latest
```
上述输出代表这个image已经构建成功了，如果是在这台机器第一次使用docker，没有java8的镜像，执行上述命令时就会慢一些，会从远程镜像库拉取java8镜像，如果不是第一次就会快很多，因为本地已经有了这个镜像，会直接使用。

## 启动container
镜像好了之后，就可以指定镜像启动容器，由于容器本身运行服务后会启动一个端口，但并不是直接使用物理机的端口，所以需要启动时把docker内服务端口和外部端口做映射。同时，为了方便操作，还可以使用后台运行的模式，使用`-d`，则最终启动命令如下：
```
docker run -d -p 8000:8080 docker-spring
```
以上操作，会启动docker-spring镜像，在docker容器中使用8080端口，并把8080端口映射到虚拟机的8000端口上。
需要注意的是，实际使用发现上边参数指定必须位于镜像名之前，如果先写镜像名后指定参数，则参数不生效。

## 验证
容器和服务启动完成，就可以在浏览器使用虚拟机ip和映射后的端口来访问docker内的服务，例如我的虚拟机是`192.168.139.91`，则根据开头的接口代码，就可以浏览器输入如下url：
```
http://192.168.139.91:8000/hello
```
会看到浏览器正常返回`hello`字符串，证明docker的springboot服务部署成功。