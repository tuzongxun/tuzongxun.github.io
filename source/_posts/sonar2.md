---
title: 阿里云centos安装sonar问题记录
date: 2022-3-26 13:43:58
categories: 运维 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [运维,sonar]
---
## 前言
最近在阿里云服务器上重新安装了jenkins，同时打算集成更多的常用的插件，例如sonar。
很多年前，我在自己的windows电脑上安装过sonar，不过那个电脑我已经没再用了，这次在阿里云上准备按之前的笔记重新安装sonar，但是依旧是不太顺利。
<!--more-->
## github下载问题
首先就是参考之前的博客[使用sonar进行java代码质量管理](https://blog.csdn.net/tuzongxun/article/details/78280322?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164826399216780366534601%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164826399216780366534601&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-78280322.nonecase&utm_term=sonar&spm=1018.2226.3001.4450)去github下载的时候就卡住了，我先执行如下操作克隆代码：
```
git clone git://github.com/SonarSource/sonar.git
```

上边操作并没有成功，而是抛出了如下的error:
```
fatal: remote error:
  The unauthenticated git protocol on port 9418 is no longer supported.
Please see https://github.blog/2021-09-01-improving-git-protocol-security-github/ for more information.
```

上边的问题，按网上的说法是由于github的安全设置，导致git协议的问题，需要执行如下操作进行一定的配置转换：
```
git config --global url."https://".insteadOf git://
```

做了上边的操作之后，我再次执行clone操作，但是依旧是失败，但是出现了新的提示，如下：
```
fatal: unable to access 'https://github.com/SonarSource/sonar.git/': OpenSSL SSL_read: Connection was reset, errno 10054
```

这个问题就相对比较常见了，我知道是因为ssl验证的问题，之前也操作过，就是再进行一个配置，如下：
```
git config --global http.sslVerify "false"
```

配置了ssl验证的问题后，再次执行clone操作，结果还是失败，新的提示如下：
```
fatal: unable to access 'https://github.com/SonarSource/sonar.git/': Failed to connect to github.com port 443 after 21058 ms: Timed out
```

根据网上的说法，这是因为git代理的问题，需要取消代码，于是我又做了如下操作：
```
git config --global --unset http.proxy
```

之后再执行clone操作，终于松了一口气，下载的进度条开始了。
然而，有些郁闷的是，这个操作在我windows机上成功了，然后我直接去阿里云的centos机上操作时，一模一样的步骤，最终依旧还是失败，始终都是这样一个错误：
```
Cloning into 'sonar'...
fatal: unable to access 'https://github.com/SonarSource/sonar.git/': Encountered end of file
```

正常来说，只要进度条在走，最终应该是可以下载成功的，但是由于下载的太慢，最终我终止了git的clone操作，选择直接去sonar官网下载：
https://www.sonarqube.org/

## 启动问题
一开始我下载的是官网最新版sonarqube-9.3.0，但是在启动的时候却没有成功，查看日志发现提示如下：
```
Running SonarQube...
wrapper  | --> Wrapper Started as Console
wrapper  | Launching a JVM...
wrapper  | JVM exited while loading the application.
jvm 1    | Unrecognized option: --add-exports=java.base/jdk.internal.ref=ALL-UNNAMED
jvm 1    | Error: Could not create the Java Virtual Machine.
jvm 1    | Error: A fatal exception has occurred. Program will exit.
wrapper  | JVM Restarts disabled.  Shutting down.
wrapper  | <-- Wrapper Stopped
```

初步猜测可能是jdk版本问题，于是就是官网查了下，然后果然看到了这样一句"You must be able to install Java (Oracle JRE 11 or OpenJDK 11) on the machine where you plan to run SonarQube."。
由于我现在服务器上还是安装的jdk8，暂时也不想升jdk11，所以就去找了下可以用jdk8的版本，最终下载了sonarqube-6.6。
不过在启动后又出现了新的问题，信息如下：
```
--> Wrapper Started as Daemon
Launching a JVM...
Wrapper (Version 3.2.3) http://wrapper.tanukisoftware.org
  Copyright 1999-2006 Tanuki Software, Inc.  All Rights Reserved.

2022.03.26 12:25:13 INFO  app[][o.s.a.AppFileSystem] Cleaning or creating temp directory /opt/soft/sonarqube-6.6/temp
2022.03.26 12:25:13 INFO  app[][o.s.a.es.EsSettings] Elasticsearch listening on /127.0.0.1:9001
2022.03.26 12:25:13 INFO  app[][o.s.a.p.ProcessLauncherImpl] Launch process[[key='es', ipcIndex=1, logFilenamePrefix=es]] from [/opt/soft/sonarqube-6.6/e/elasticsearch/bin/elasticsearch -Epath.conf=/opt/soft/sonarqube-6.6/temp/conf/es
2022.03.26 12:25:13 INFO  app[][o.s.a.SchedulerImpl] Waiting for Elasticsearch to be up and running
2022.03.26 12:25:14 INFO  app[][o.e.p.PluginsService] no modules loaded
2022.03.26 12:25:14 INFO  app[][o.e.p.PluginsService] loaded plugin [org.elasticsearch.transport.Netty4Plugin]
2022.03.26 12:25:17 WARN  app[][o.s.a.p.AbstractProcessMonitor] Process exited with exit value [es]: 1
2022.03.26 12:25:17 INFO  app[][o.s.a.SchedulerImpl] Process [es] is stopped
2022.03.26 12:25:17 INFO  app[][o.s.a.SchedulerImpl] SonarQube is stopped
<-- Wrapper Stopped
```

通过日志，大概知道是es启动有问题，但是不知道为啥，然后百度得知是因为es不能用root用户启动，于是做了用户相关的一些操作，如：
```
useradd sonar
chown -R sonar:sonar sonarqube-6.6
```

之后切换到sonar用户重新启动，依旧是没有成功，错误信息如下：
```
]: /opt/soft/jdk1.8.0_261/jre/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/opt/soft/sonarqube-6.6/temp -Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -cp ./lib/common/*:./lib/server/*:/opt/soft/sonarqube-6.6/lib/jdbc/h2/h2-1.3.176.jar org.sonar.server.app.WebServer /opt/soft/sonarqube-6.6/temp/sq-process8738477899699194234properties
Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000e0000000, 44695552, 0) failed; error='Cannot allocate memory' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 44695552 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /opt/soft/sonarqube-6.6/hs_err_pid19910.log
2022.03.26 12:36:31 WARN  app[][o.s.a.p.AbstractProcessMonitor] Process exited with exit value [web]: 1
2022.03.26 12:36:31 INFO  app[][o.s.a.SchedulerImpl] Process [web] is stopped
```

这个错误是比较容易看出来的，就是内存不够，不过还是不太清楚具体是什么内存不够，于是查百度得到了解决方案，参考文档如下：
https://www.cnblogs.com/brady-wang/p/11307877.html
按照上边文档设置了m.max_map_count=262144后，再次启动sonar，终于启动成功，并且也成功通过阿里云外网http://139.224.23.230:9000/进行访问。