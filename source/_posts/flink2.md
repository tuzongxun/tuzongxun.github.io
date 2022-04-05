---
title: flink-HA集群搭建和问题记录
date: 2020-10-14 22:59:42
tags: flink
toc: true
categories: flink
---
上一篇中，搭建了一个简单的flink集群，在这个集群中，我使用了一个`jobManager`节点，三个`taskManager`节点，之后根据官网和其他资料写了一个简单的java验证程序验证集群的可用:

[flink初识及集群搭建和简单验证](https://blog.csdn.net/tuzongxun/article/details/109039580)

<!--more-->

虽然这个集群搭建是成功的，但是这种方式的集群却存在问题。

flink集群中jobManager和taskManager这种，是典型的`master/slave`主从模式的设计，jobManager具有`资源管理`和`任务调度`的功能，管理taskManager的资源和调度，也就是启动以及外部的交互实际都是先经过jobManager。

这种情况下，虽然有三个实际的taskManager处理任务，但是jobManager是单机的，一旦jobManger故障，则整个集群依然不可用。

这个其实和hdfs中`nameNode`与`dataNode`的关系很像，可能就是一样的（带后续都深入之后再比较）。

**hdfs的ha中，实际就是增加了nameNode的节点，交给zookeeper管理，而flink的ha也类似，即增加jobManager的节点数，也要依赖zookeeper来管理jobManager。**

之前的简单flink集群，我使用的节点情况如下：

| 类别        | node001 | node002 | node003 | node004 |
| ----------- | ------- | ------- | ------- | ------- |
| jobManager  | ●       |         |         |         |
| taskManager |         | ●       | ●       | ●       |

## flink-HA机器规划

虽说简单的flink集群升级为HA，本质上是增加jobManager节点，但是实际并不是单纯的加一个jobManager节点就够了。

上边提到了需要zookeeper管理jobManager的集群，所以还需要zookeeper。

同时，由于jobManager的任务较重，相应的源数据也很多，因此官方建议使用可被各个节点访问的持久化文件系统存储源数据，鉴于之前搭建了hdfs，自然就直接选用了。

zookeeper和hdfs之前都已经搭建好了，就可以直接使用，需要参考的可以查看之前这篇：

[HDFS-HA模式搭建（基于完全分布式模式升级）](https://blog.csdn.net/tuzongxun/article/details/108347899)

因此，最终的flink-HA集群机器规划如下：

| 类别               | node001 | node002 | node003 | node004 |
| ------------------ | ------- | ------- | ------- | ------- |
| nameNode(HDFS)     | ●       |         |         | ●       |
| dataNode(HDFS)     |         | ●       | ●       |         |
| journalnode(HDFS)  | ●       | ●       | ●       |         |
| zkfc(HDFS)         | ●       |         |         | ●       |
| nodeManager(HDFS)  |         | ●       | ●       |         |
| zookeeper          |         | ●       | ●       | ●       |
| jobManager(FLINK)  | ●       | ●       |         |         |
| taskManager(FLINK) |         |         | ●       | ●       |

## flink-conf.yaml修改

规划好了机器，然后就是修改配置，实际也很简单，首先还是修改flink安装目录的conf目录下flink-conf.yaml文件，找到如下的三个配置，把原本的注释放开，然后配置自己的hdfs地址和zookeeper地址。

需要注意的是，我这里的hdfs是之前的ha集群，`mycluster`是我的hdfs的集群名，至于后边的内容会在hdfs中创建路径，可以自定义，不需要提前创建。

```
high-availability: zookeeper
high-availability.storageDir: hdfs://mycluster/flink/ha/
high-availability.zookeeper.quorum: node002:2181,node003:2181,node004:2181
```

## workers修改

上一篇有提到过，旧版本的flink中有个文件叫`slaves`，新版的就叫这个`workers`，代表的是taskManger节点，之前我配置了三个，现在其中一个换成jobManager，所以删掉一个之后内容如下：

```
node003
node004
```

## masters修改

之前的监看flink集群搭建时，是没有管这个文件的，因为jobManager就只有一个，现在有了两个jobManager，就需要修改这个文件制定jobManager集群的节点。

实际上从这里，尤其是之前的`masters`和`slaves`这两个文件的命令，也很容易看出来他们的主从关系。

修改后的masters文件内容如下：

```
node001:8081
node002:8081
```

## 配置文件同步分发

和hdfs一样，和flink简单集群一样，这些修改的配置文件也都需要同步分发到所有的节点中，`scp`就不多说了。

## hadoop依赖jar下载

上边操作完成后，我就使用`start-cluster.sh`启动的集群，然后看到打印出了如下的信息：

```
Starting HA cluster with 2 masters.
Starting standalonesession daemon on host node001.
Starting standalonesession daemon on host node002.
Starting taskexecutor daemon on host node003.
Starting taskexecutor daemon on host node004.
```

也没有报错，我以为就成功了，但是当我访问web页面时，无论是`http://node001:8081`还是`http://node002:8081`都无法访问，于是查看了flink的日志文件，结果发现日志中打印了如下的异常信息：

```
Caused by: org.apache.flink.core.fs.UnsupportedFileSystemSchemeException: Could not find a file system implementation for scheme 'hdfs'. The scheme is not directly supported by Flink and no Hadoop file system to support this scheme could be loaded. For a full list of supported file systems, please see https://ci.apache.org/projects/flink/flink-docs-stable/ops/filesystems/.
	at org.apache.flink.core.fs.FileSystem.getUnguardedFileSystem(FileSystem.java:491) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.core.fs.FileSystem.get(FileSystem.java:389) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
```

看起来就是无法识别和连接hdfs，实际上是因为没有相关的依赖，因此需要下载flink依赖的hadoop的jar到flink安装目录下的`lib`目录下。

这个插件在flink官网可以找到，`https://flink.apache.org/downloads.html`，这个连接中`Additional Components`下就是flink依赖的hadoop插件。

按网上说的，需要根据相应的hadoop版本下载对应的插件版本，但是我的hadoop是`3.1.3`，而这个页面中最高才是`2.8.3`，因此最终就下载了这个版本。

之后重新执行`start-cluster.sh`后日志没有再打印上边的异常，同时web页面也都可以成功打开了，并能看到两个taskManger。

**在web页面提交上一次做好的flink程序的jar之后，也能看到`running`状态，似乎ha模式搭建成功了，但是实际上并不是。**

## classpath配置

当在上述jar生成的task运行的机器节点使用`nc -lk 8888`启动监听并发送数据后，web页面的`Stdout`上并没有如愿输出单词的统计次数，反而是在对应机器节点的日志中出现了异常。

通过查看，我的这个task运行在`node003`节点，找到日志使用`tail -500 flink-root-taskexecutor-1-node003.log`后发现了如下的异常：

```
2020-10-14 16:00:56,527 ERROR org.apache.flink.runtime.taskexecutor.TaskManagerRunner      [] - TaskManager initialization failed.
java.io.IOException: Could not create FileSystem for highly available storage path (hdfs://mycluster/flink/ha/default)
	at org.apache.flink.runtime.blob.BlobUtils.createFileSystemBlobStore(BlobUtils.java:103) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.blob.BlobUtils.createBlobStoreFromConfig(BlobUtils.java:89) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.highavailability.HighAvailabilityServicesUtils.createHighAvailabilityServices(HighAvailabilityServicesUtils.java:117) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.taskexecutor.TaskManagerRunner.<init>(TaskManagerRunner.java:133) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.taskexecutor.TaskManagerRunner.runTaskManager(TaskManagerRunner.java:306) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.taskexecutor.TaskManagerRunner.lambda$runTaskManagerSecurely$2(TaskManagerRunner.java:330) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.security.contexts.NoOpSecurityContext.runSecured(NoOpSecurityContext.java:30) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.taskexecutor.TaskManagerRunner.runTaskManagerSecurely(TaskManagerRunner.java:329) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.taskexecutor.TaskManagerRunner.runTaskManagerSecurely(TaskManagerRunner.java:314) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.taskexecutor.TaskManagerRunner.main(TaskManagerRunner.java:298) [flink-dist_2.11-1.11.2.jar:1.11.2]
Caused by: org.apache.flink.core.fs.UnsupportedFileSystemSchemeException: Could not find a file system implementation for scheme 'hdfs'. The scheme is not directly supported by Flink and no Hadoop file system to support this scheme could be loaded. For a full list of supported file systems, please see https://ci.apache.org/projects/flink/flink-docs-stable/ops/filesystems/.
	at org.apache.flink.core.fs.FileSystem.getUnguardedFileSystem(FileSystem.java:491) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.core.fs.FileSystem.get(FileSystem.java:389) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.core.fs.Path.getFileSystem(Path.java:292) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.runtime.blob.BlobUtils.createFileSystemBlobStore(BlobUtils.java:100) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	... 9 more
Caused by: org.apache.flink.core.fs.UnsupportedFileSystemSchemeException: Hadoop is not in the classpath/dependencies.
	at org.apache.flink.core.fs.UnsupportedSchemeFactory.create(UnsupportedSchemeFactory.java:58) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
```

经过一番查询和尝试后找到了解决办法，即配置两个环境变量，环境变量的配置方式较多，可以配系统变量，可以配用户变量，我就直接配置的系统变量，执行`vi /etc/profile`，然后加入如下两行：

```
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_CLASSPATH=`hadoop classpath`
```

配置完成后使用`source /etc/profile`重新加载刚修改的内容，然后重新提交flink程序jar后日志不在报错，同时再次在`nc`中输入单词后，在web界面的`Stdout`中便能成功的刷新出预想的结果，至此，flink的ha模式搭建成功，搭建过程也算是对flink的设计思想和架构有了更进一步的认识。