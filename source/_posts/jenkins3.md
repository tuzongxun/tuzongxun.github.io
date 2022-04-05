---
title: jenkins pipeline部署实践及重点问题分析
date: 2022-3-18 18:43:58
categories: 运维 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [运维,jenkins]
---
## 前言
根据网上的说法，以及暂时使用过程中的感受，使用自由风格或者maven风格来创建jenkins item，虽然也能实现自动化部署，但是面对相对复杂的构建需求时可能就不太好实现。
一般正式的项目，除了基本的拉取代码、编译代码、运行junit、打包、启动或者重启外，可能还会涉及到sonar代码检查、集成测试、关联例如jira或者conflunce等系统。
因此，我目前所知道的很多正式项目在使用jenkins时可能都会使用pipeline流水线，如果要使用pipeline，需要先安装`pipeline`插件。
pipeline看起来好像也不是太难，但是真正自己操作的时候可能会发现很多地方会有小问题。例如可能涉及到ssh问题，可能涉及到容器操作问题，也可能涉及到shell脚本问题。
以下是我实际操作过程中的一些记录，主要分为两个部分：一部分是我只使用了一台linux虚拟机，jenkins、git、maven、springboot服务都在上边跑；另一部分是我把spirngboot服务单独放到了另一个虚拟机上。
同一台机的时候，不涉及ssh，但是却会涉及到jenkins杀死java进程的问题，不同机器就涉及到ssh远程调用shell脚本的问题。
<!--more-->
## 构建步骤
jenkins pipeline进行构建的时候，根据需求，可以有各种不同的步骤，我这里模拟的只有这样几个基础的：
>拉取git代码；
maven执行junit并打包；
推送jar包到服务目录；
启动或者重启springboot服务。

那么这里，前两步其实是一样的，后两步略有区别。推送jar包，在jenkins本机实际就是用的cp操作，而其他机器则是使用的scp操作。
启动和重启spingboot服务，本机直接启动脚本，远程就需要额外的配置。

## 部署到jenkins本机
jenkins的pipeline配置，分为脚本式和声明式两种，语法上不同，但是主要内容是差不多的。
这里需要注意的是部署到jenkins本机，最后启动服务的操作。
我一开始写的是`sh "/opt/sh/test.sh restart base-springboot.jar"`，然后如果是把这些内容直接贴到jenkins 的管理界面配置pipeline那里，build的时候都是成功的，服务也都是正常启动。
`但是当我把这些内容写到jenkinsfile文件中，然后和springboot项目代码放在一起，上传到github上，然后jenkins build的时候也都是成功的，可是却发现每次构建结束后springboot服务都没有启动起来，也查不到相关的java进程`。
这个地方其实花了好长时间，后来查了各种资料才大概知道了jenkins会杀掉这里启动的java进程，至于为什么，到现在都还没能弄清楚。
所以需要把上边的内容改成`sh "JENKINS_NODE_COOKIE=dontKillMe /opt/sh/test.sh restart base-springboot.jar"`。
当然了，在使用其他远程机器部署的时候，就不会有这个问题。
### jenkins本机部署服务-脚本式
```
node {
   stage('Print Message') {
      echo "【start build】 workspace: ${WORKSPACE}"
   }

   stage('git pull code') {
      echo "【git pull】"
      git branch: 'jenkin-test', credentialsId: '3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2', url: 'git@github.com:tuzongxun/base-springboot.git'
   }

   //mvn打包
   stage('mvn build project') {
      echo "【mvn build】"
      sh 'mvn clean package'
   }

   //上传jar
   stage('Push jar') {
      echo "【push jar】"
      sh 'cp /root/.jenkins/workspace/pip-test/target/base-springboot-0.0.1-SNAPSHOT.jar /opt/code/base-springboot.jar'
      sh '''rm -rf  `ls | egrep -v '(Jenkinsfile|start.sh)'` '''
   }

   stage('start/restart service') {
        echo "【start/restart service】"
        sh "JENKINS_NODE_COOKIE=dontKillMe /opt/sh/test.sh restart base-springboot.jar"
    }

}
```

### jenkins本机部署服务-声明式
```
pipeline {
    agent any

    stages {
        //打印信息
       stage('Print Message') {
          steps {
             echo "【start build】 workspace: ${WORKSPACE}"
          }
       }

       stage('git pull code') {
          steps {
             echo "【git pull】"
             git branch: 'jenkin-test', credentialsId: '3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2', url: 'git@github.com:tuzongxun/base-springboot.git'
          }
       }

       //mvn打包
       stage('mvn build project') {
          steps {
             echo "【mvn build】"
             script {
                try {
                //刚开始使用mvn clean package 提示找不到mvn,所以就这样指定地址，指定配置文件
                    sh 'mvn clean package'
                } catch (err) {
                   echo 'mvn打包失败'
                }
             }
          }
       }

       //上传jar
       stage('Push jar') {
          steps {
             echo "【push jar】"
             dir('/root/.jenkins/workspace/pip-test') { //指定工作目录
                script {
                   try {
                      sh 'cp ./target/base-springboot-0.0.1-SNAPSHOT.jar /opt/code/base-springboot.jar'
                   } catch (err) {
                      echo "上传jar构建失败"
                   }
                }
             }
             //推送镜像后，删除工作空叫初Jenkinsfile & start.sh 以外所有文件
             sh '''rm -rf  `ls | egrep -v '(Jenkinsfile|start.sh)'` '''
          }
       }

       stage('Deploy to the Target server') {
          steps {
                sh "JENKINS_NODE_COOKIE=dontKillMe /opt/sh/test.sh restart base-springboot.jar"
          }
       }

    }
}
```

## shell脚本
在上边的pipeline中调用了shell脚本来重启springboot服务，或者说java进程，对应的shell脚本如下：
```
#!/bin/bash
APP_NAME=$2
source /etc/profile
BUILD_ID=dontKillMe
usage() {
    echo "Usage: sh test.sh [start|stop|restart|status]"
    exit 1
}

is_exist(){
    pid=0
    javaps=`jps -l |grep base-springboot.jar`
    if [ -n "$javaps" ]; then
      echo ${javaps}
      pid=`echo ${javaps} | awk '{print $1}'`
      echo "bb":${pid}
      return 0
    else
      echo "cc":${pid}
      return 1
    fi
}

start(){
    is_exist
    if [ $? -eq "0" ]; then
        echo "${APP_NAME} is already running. pid=${pid} ."
    else
        nohup java -jar /opt/code/${APP_NAME} >> /opt/code/test.log 2>&1 &
        echo "${APP_NAME} start success"
    fi
}

stop(){
    is_exist
    if [ $? -eq "0" ]; then
        echo "aa":${pid}
        kill -9 ${pid}
    else
        echo "${APP_NAME} is not running"
    fi
}

status(){
    is_exist
    if [ $? -eq "0" ]; then
        echo "${APP_NAME} is running. Pid is ${pid}"
    else
        echo "${APP_NAME} is NOT running."
    fi
}

restart(){
    stop
    sleep 5
    start
}

case "$1" in
    "start")
        start
        ;;
    "stop")
        stop
        ;;
    "status")
        status
        ;;
    "restart")
        restart
        ;;
    *)
        usage
        ;;
esac

exit 0
```

这里其实也有一个问题，由于对linux和shell脚本不太熟，所以很多东西都是网上找的，一开始根据网上找到的资料，查看相关java进程，使用的是这段代码：
```
is_exist(){
    pid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v $SCRIPT|awk '{print $2}'`
    echo $pid
    if [ -z "${pid}" ]; then
        return 1
    else
        return 0
    fi
}
```
但是后来发现总在报错，于是又改成了这样：
```
is_exist(){
    pid=`ps -ef|grep $APP_NAME|grep -v grep|awk '{print $2}'`
    echo $pid
    if [ -z "${pid}" ]; then
        return 1
    else
        return 0
    fi
}
```

但是这样之后又发现每次拿到的pid都不止1个，于是在后边stop的时候一样有问题。所以最终改成了上边脚本示例的这样：
```
is_exist(){
    pid=0
    javaps=`jps -l |grep base-springboot.jar`
    if [ -n "$javaps" ]; then
      echo ${javaps}
      pid=`echo ${javaps} | awk '{print $1}'`
      echo "bb":${pid}
      return 0
    else
      echo "cc":${pid}
      return 1
    fi
}
```

## 部署到非jenkins所在机器的pipeline配置
虽然jenkins本机部署可用了，但是一般真实项目，基本都不会是业务服务和jenkins在同一台机上，所以我又克隆了一台虚拟机作为业务服务器。
不过，这里暂时只跑通了脚本式的pipeline配置，声明式的没有成功。以下是脚本式配置的代码：
```
node {
    def remote = [:]
    remote.name = 'test'
    remote.host = '192.168.19.200'
    remote.user = 'root'
    remote.allowAnyHosts = true

   stage('Print Message') {
      echo "【start build】 workspace: ${WORKSPACE}"
   }

   stage('git pull code') {
      echo "【git pull】"
      git branch: 'jenkin-test', credentialsId: '3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2', url: 'git@github.com:tuzongxun/base-springboot.git'
   }

   //mvn打包
   stage('mvn build project') {
      echo "【mvn build】"
      sh 'mvn clean package'
   }

   //上传jar
   stage('Push jar') {
      echo "【push jar】"
      sh 'scp /root/.jenkins/workspace/pip-test/target/base-springboot-0.0.1-SNAPSHOT.jar root@192.168.19.200:/opt/code/base-springboot.jar'
      sh '''rm -rf  `ls | egrep -v '(Jenkinsfile|start.sh)'` '''
   }

   stage('start/restart service') {
        withCredentials([sshUserPrivateKey(credentialsId: '3bdb20f7-1c4f-4a34-98c0-ef1b9202cbe2', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'root')]) {
            echo "【start/restart service】"
            remote.user = 'root'
            remote.identityFile = identity
            sshCommand remote: remote, command: "/opt/sh/test.sh restart base-springboot.jar"
        }
    }

}
```

这里主要就是上传jar时使用了scp，这里没有出现密码，是因为在另一台机上配置了jenkins所在机的sshkey的公钥。
然后就是远程调用时配置的`credentialsId`，我两台机`192.168.19.199`和`192.168.19.200`，我是需要用jenkins所在的199去调用200机器上的shell脚本。
一开始以为和scp一样，已经在200配置了199的公钥，那么应该可以直接调用了。但是结果发现并不是想的那样，还需要配置jenkins中配置的credentials的Id。

## 总结
pipeline本身或许不难，但是每一步涉及到的可能都是一个面的知识，例如shell、ssh、jenkins作用域或者工作空间之类的，需要有比较好的综合知识来支持。
pipeline可以支持的需求和功能很多，基于它相关的插件也有很多，待后续再逐步了解。