---
title: flink on yarn集群搭建及验证要点记录
date: 2020-10-16 12:59:42
tags: flink
toc: true
categories: flink
---
## standalone模式的弊端

前面搭建了flink简单集群，并在此基础上又搭建了HA集群，记录地址如下：

[flink初识及集群搭建和简单验证](https://tuzongxun.blog.csdn.net/article/details/109039580)

[flink-HA集群搭建和问题记录](https://tuzongxun.blog.csdn.net/article/details/109081839)

虽然上述两种都能使用，在学习flink-api阶段应该是够用了，但是如果真要上生产使用，就还是有一定的弊端。

<!--more-->

根据之前的学习可知，flink集群主要分为`jobManager`和`taskManager`，而jobManger的任务主要有两个，一个是`资源管理`，另一个是`任务调度`。

这样一来，jobManager的任务其实就显得有点多，而又由于mapReduce、spark、flink的集群都能基于yarn管理资源，所以有一个更好的方式就是让flink集群运行在yarn上。

yarn是hadoop项目自带的模块，启动hadoop集群会一起启动yarn，所以也不用单纯的再维护yarn，同时，yarn的资源管理也能分担jobManager的资源管理，使得这种模式下的jobManager基本只需要专注于任务调度，也能进一步提高可用性。

之所以说是基本专注，是因为它不直接管理资源，但是还是要为taskManager向yarn申请资源，而这里说的yarn管理资源，实际上就是之前hdfs搭建时里边所说的`resourceManager`，可以参考之前的hdfs搭建记录：

[HDFS-HA模式搭建（基于完全分布式模式升级）](https://tuzongxun.blog.csdn.net/article/details/108347899)及[hadoop分布式安装及配置初步解析（坑坑不息）](https://tuzongxun.blog.csdn.net/article/details/107843326)



## flink on yarn运行

单纯的使flink集群运行在yarn上，不需要额外的配置，只需要使用yarn命令启动flink集群即可，但是使用方式有两种，一种是提交任务的同时启动集群，另一种是先启动集群再提交运行任务。

前一种方式每次提供运行任务时启动集群，关闭任务也会关了集群，后一种则是一直保持`jobManager`开启，`taskManager`按需启动，因此后一种相对更加常用，这种方式分为两步：

步骤一，在yarn上启动flink集群，例如：

```
yarn-session.sh -n 3 -s 3 -nm flink-yarn-test -d
```

上边参数的意思是：`-n`代表最多启动的`taskManager`数量，`-s`代表每个`taskManager`中最多分配的`slot`数量，`-nm`代表自定义的flink集群名称，`-d`代表后台运行。

步骤二，提交flink任务，这里其实又可以分为两种方式，一种是在web界面提交，一种是命令行，命令行提交示例如下：

```
flink run -c com.tzx.study.demo.flink.FlinkTest -yid application_1586794520478_0007 tzx-study-demo.jar
```

这里`-c`指定flink程序启动类的路径，`-yid`指定任务要提交到的flink（yarn）集群的id，末尾是jar包路径，我是在jar包所在目录执行的命令，因此就没有其他前缀。

这里需要注意的是，`-yid`可以不指定，默认会提交到最新启动的flink(yarn)集群中。

**注：上述操作若有问题，请继续往下看。上述操作是理论操作，因为HA方式也是一样，因此我直接在HA中操作，因此有些问题会直接记录在下边。**

## flink on yarn HA配置

上边的方式把flink运行在yarn上，有一个问题在于`jobManager`一样是单机的，也会有单点故障，因此正常的生产环境也应该是要使用HA方式。

yarn中的flink的HA，实际是增加`jobManager`的故障重试次数，进而使得原本运行的`jobManager`出现问题后，yarn能够再启动一个新的`jobManager`，从而提高整个flink集群的可用性，这个配置在`hadoop`中的`yarn-site.xml`文件中，增加如下配置：

```
<property>
        <name>yarn.resourcemanager.am.max-attempts</name>
        <value>10</value>
</property>
```

有了上述配置之后，一次启动zookeeper集群、hdfs集群、flink集群，但是执行`yarn-session`启动flink集群时却出错了，错误如下：

```
2020-10-16 09:52:05,326 ERROR org.apache.flink.yarn.cli.FlinkYarnSessionCli                [] - Error while running the Flink session.
org.apache.flink.client.deployment.ClusterDeploymentException: Couldn't deploy Yarn session cluster
	at org.apache.flink.yarn.YarnClusterDescriptor.deploySessionCluster(YarnClusterDescriptor.java:382) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.yarn.cli.FlinkYarnSessionCli.run(FlinkYarnSessionCli.java:514) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.yarn.cli.FlinkYarnSessionCli.lambda$main$4(FlinkYarnSessionCli.java:751) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at java.security.AccessController.doPrivileged(Native Method) ~[?:1.8.0_261]
	at javax.security.auth.Subject.doAs(Subject.java:422) ~[?:1.8.0_261]
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1836) ~[flink-shaded-hadoop-2-uber-2.8.3-10.0.jar:2.8.3-10.0]
	at org.apache.flink.runtime.security.contexts.HadoopSecurityContext.runSecured(HadoopSecurityContext.java:41) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.yarn.cli.FlinkYarnSessionCli.main(FlinkYarnSessionCli.java:751) [flink-dist_2.11-1.11.2.jar:1.11.2]
Caused by: org.apache.flink.yarn.YarnClusterDescriptor$YarnDeploymentException: The YARN application unexpectedly switched to state FAILED during deployment. 
Diagnostics from YARN: Application application_1602812938780_0001 failed 2 times in previous 10000 milliseconds (global limit =10; local limit is =2) due to AM Container for appattempt_1602812938780_0001_000003 exited with  exitCode: 127
Failing this attempt.Diagnostics: [2020-10-16 09:52:04.913]Exception from container-launch.
Container id: container_1602812938780_0001_03_000001
Exit code: 127

[2020-10-16 09:52:04.920]Container exited with a non-zero exit code 127. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :

[2020-10-16 09:52:04.931]Container exited with a non-zero exit code 127. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :

For more detailed output, check the application tracking page: http://node001:18088/cluster/app/application_1602812938780_0001 Then click on links to logs of each attempt.
. Failing the application.
If log aggregation is enabled on your cluster, use this command to further investigate the issue:
yarn logs -applicationId application_1602812938780_0001
	at org.apache.flink.yarn.YarnClusterDescriptor.startAppMaster(YarnClusterDescriptor.java:1021) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.yarn.YarnClusterDescriptor.deployInternal(YarnClusterDescriptor.java:524) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.yarn.YarnClusterDescriptor.deploySessionCluster(YarnClusterDescriptor.java:375) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	... 7 more
```

上述提示内容没有看出实质性的问题，但是却发现了如下两行：

```
If log aggregation is enabled on your cluster, use this command to further investigate the issue:
yarn logs -applicationId application_1602812938780_0001
```

不用完全看懂也能大概知道要进一步查问题，需要执行`yarn logs -applicationId application_1602812938780_0001`命令，但是直接执行会发现也看不到什么内容，反而有如下错误提示：

```
2020-10-16 10:07:48,478 INFO client.RMProxy: Connecting to ResourceManager at node001/192.168.139.91:18040
File /tmp/logs/root/logs-tfile/application_1602812938780_0001 does not exist.

Can not find any log file matching the pattern: [ALL] for the application: application_1602813970991_0001
Can not find the logs for the application: application_1602813970991_0001 with the appOwner: root
```

这个问题其实上边有说明了，使用上述命令的前提是集群中开启聚合日志，因此需要在`yarn-site.xml`中再增加一个配置开启这个日志：

```
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
```

重启hdfs集群后，再执行`yarn-session`来启动flink集群，还是会出现启动失败的提示，但是再次执行`yarn logs`就有了更多的日志信息，其中有一段如下：

```
Container: container_1602813970991_0001_02_000001 on node002_40809
LogAggregationType: AGGREGATED
==================================================================
LogType:jobmanager.err
LogLastModifiedTime:Fri Oct 16 09:52:05 +0800 2020
LogLength:48
LogContents:
/bin/bash: /bin/java: No such file or directory

End of LogType:jobmanager.err
```

有了这段提示，原因就显而易见了，这是很多地方的常见问题，找不到`JAVA_HOME`。

但是为啥找不到呢，联想一下之前的hdfs集群搭建，想起来hdfs集群启动的时候，会从一台机`ssh`到另一台机启动shell进而启动该节点。

而这个shell不会加载`/etc/profile`文件，也就导致会找不到`JAVA_HOME`。

根据之前在`hadoop-env.sh`中增加`JAVA_HOME`配置的经验，猜想应该是yarn需要一样的处理，于是在`yarn-env.sh`中加入了`JAVA_HOME`的配置：

```
export JAVA_HOME=/opt/java/jdk1.8.0_261
```

**需要注意，以上所有配置的修改，均需要同步分发到所有节点。**

## web ui

之前在`standalone`模式中，可以直接访问`8081`端口查看`jobManager`的`web ui`，在这里界面能看到很多信息，也能直接提交flink的任务，而flink运行在yarn上，一样有`web ui`，只不过有两个。

其中一个可以通过`yarn`的`web ui`跳转，另一个则是直接访问，但是这两个有所区别。

通过`yarn`跳转的，默认只能看一些信息，不能提交任务，而直接访问的，和原来`8081`访问的效果一样。

上边配置好`JAVA_HOME`后，再重启hadoop和yarn下的flink集群，会看到不再报错，并且最后带出了`jobManager`的ui地址：

```
2020-10-16 10:33:15,488 INFO  org.apache.flink.shaded.curator4.org.apache.curator.framework.state.ConnectionStateManager [] - State change: CONNECTED
JobManager Web Interface: http://node003:37917
```

上边这个`http://node003:37917`就是我现在可直接访问的，等同之前`8081`访问的`jobManager`的`web ui`。

而使用`yarn`跳转方式，需要先访问yarn的`web ui`，默认端口是`8088`，而我搭建的时候配置的是`18088`。

## 其他要点

在`standalone`模式的HA集群中，有两个重要文件，`masters`和`workers`，这两个文件决定了`jobManager`和`taskManager`节点。

而在yarn中，资源是yarn管理，这两个文件实际是无效的，我试过清空这两个文件的内容，对运行没有任何影响。

但是和`standalone`模式一样的是，在系统环境变量中依然需要有下边的配置存在：

```
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop export 

HADOOP_CLASSPATH=`hadoop classpath`
```

