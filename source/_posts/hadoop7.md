---
title: HDFS-HA模式搭建（基于完全分布式模式升级）
date: 2020-9-1 19:59:42
tags: hadoop
toc: true
categories: hadoop
---
## 说明

本次HDFS-HA模式搭建基于之前的完全分布式，完全分布式搭建可参考之前的内容：

[hadoop安装环境准备和关联知识解析](https://blog.csdn.net/tuzongxun/article/details/107811747)

[hadoop分布式安装及配置初步解析（坑坑不息）](https://blog.csdn.net/tuzongxun/article/details/107843326)
<!--more-->

概括性来说，大概分为如下几个部分：
```
JDK安装和JAVA_HOME配置

HOSTS映射

SSH免密登陆设置

HADOOP配置文件修改

配置HADOOP_HOME

初始化（格式化）

启动验证
```

hadoop项目常被提到的自身模块有yarn、hdfs、mapreduce，hdfs和yarn都可以搭建为高可用（HA），本篇先只介绍hdfs的HA搭建。

## 机器规划

完全分布式和HA模式不仅在配置上有区别，在运行之后启动的进程方面也有很大的区别，之前的完全分布式各机器和主要进程大概如下

### 完全分布式

| host                 | nomenode | datanode | resourceManager | nodeManager | secondaryNameNode |
| -------------------- | -------- | -------- | --------------- | ----------- | ----------------- |
| 192.168.139.9master  | ●        |          | ●               |             | ●                 |
| 192.168.139.19node01 |          | ●        |                 | ●           |                   |
| 192.168.139.29node02 |          | ●        |                 | ●           |                   |

### HA模式

相对于完全分布式模式，HA中加入了ZOOKEEPER、ZKFC、JOURNALNODE，但是没有了secondaryNameNode，搭建好或者说规划后的HA模式各进程及节点分布如下所示：

| host                       | nomenode | datanode | journalnode | zkfc | zk   | nodeManager | resourceManager |
| -------------------------- | -------- | -------- | ----------- | ---- | ---- | ----------- | --------------- |
| 192.168.139.9<br />master  | ●        |          | ●           | ●    |      |             | ●               |
| 192.168.139.19<br />node01 |          | ●        | ●           |      | ●    | ●           |                 |
| 192.168.139.29<br />node02 |          | ●        | ●           |      | ●    | ●           |                 |
| 192.168.139.39<br />node03 | ●        |          |             | ●    | ●    |             | ●               |

需要注意的是，这里的resourceManager是在active节点，并不是两个namenode同时存在。

## HDFS-HA的理解

在完全分布式模式中，namenode节点只有一个，datanode是多个，而namenode用来管理datanode的源数据，一旦namenode出故障，则整个hdfs就会不可用。

所以这里说的HDFS的HA模式，个人理解实际就是namenode的HA，为namenode增加主从模式，增加故障转移和自动主从切换，无论是新加入的zookeeper中间件，还是新加的journalnode进程，其主要作用就是保证namenode的高可用，使得原活跃节点故障后整体依然能提供正常服务。

## 基础环境准备

从上边表格可以看出，相对于完全分布式，我这里又加了一台机，主要是怕三台撑不住。

所以这里的基础环境准备，实际说的是新加的机器，因为基础环境是通用的，即jkd、ssh免密、hosts映射，具体操作可参考开头所列的两篇博客内容。

## 配置

在之前的完全分布式模式中，配置了很多内容，例如hdfs-site.xml、hadoop-env.sh、core-site.xml、workers等，这次的HA模式搭建只需修改hdfs-site.xml、hadoop-env.sh、core-site.xml这三个。

### hdfs-site.xml

据说hadoop3.x最多可以支持5个namenode，但是自己电脑资源有限，就只使用了两个。在下边的配置中，主要分成四段，整体配置如下：

```

<!--1-->

<property>

<name>dfs.nameservices</name>

<value>mycluster</value>

</property>

<property>

<name>dfs.ha.namenodes.mycluster</name>

<value>nn1,nn2</value>

</property>

<property>

<name>dfs.namenode.rpc-address.mycluster.nn1</name>

<value>master:8020</value>

</property>

<property>

<name>dfs.namenode.rpc-address.mycluster.nn2</name>

<value>node03:8020</value>

</property>

<property>

<name>dfs.namenode.http-address.mycluster.nn1</name>

<value>master:9870</value>

</property>

<property>

<name>dfs.namenode.http-address.mycluster.nn2</name>

<value>node03:9870</value>

</property>

<!--2-->

<property>

<name>dfs.namenode.shared.edits.dir</name>

<value>qjournal://master:8485;node01:8485;node02:8485/mycluster<

/value>

</property>

<property>

<name>dfs.journalnode.edits.dir</name>

<value>/root/soft/bigdata/journal/data</value>

</property>

<!--3-->

<property>

<name>dfs.client.failover.proxy.provider.mycluster</name>

<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>

</property>

<property>

<name>dfs.ha.fencing.methods</name>

<value>sshfence</value>

</property>

<property>

<name>dfs.ha.fencing.ssh.private-key-files</name>

<value>/root/.ssh/id_rsa</value>

</property>

<!--4-->

<property>

<name>dfs.ha.automatic-failover.enabled</name>

<value>true</value>

</property>

```

上边第一部分是配置namenode集群，指定集群名字、节点名称以及节点主机，前边两个都是可以自定义的。

第二部分指定journalnode节点所在机器和本地存储目录，后边启动journalnode就在这里配置的节点启动，相应的数据也会存到指定的目录中。

第三部分指定HA角色切换代理类和方法，这里使用ssh免密方式。

第四部分设置开启HA自动化，实际就是ZKFC。

### core-site.xml

core-site的配置相对于HDFS就要简单许多，配置如下：

```

<property>

<name>fs.defaultFS</name>

<value>hdfs://mycluster</value>

</property>

<property>

<name>ha.zookeeper.quorum</name>

<value>node01:2181,node02:2181,node03:2181</value>

</property>

<property>

<name>hadoop.tmp.dir</name>

<value>/opt/hadoop/hadoopdata</value>

</property>

```

在完全分布式模式中，第一项配置的是具体的某台机的hostname或者ip，HA模式下就需要修改为上边定义的namenode集群的名字，虽然上边说具体名字可以自定义，但是这里配置的需要和上边的一致。

第二项即指定zookeeper集群节点，第三项是之前完全分布式模式就有的。

### hadoop-env.sh

网上很多地方似乎没有提这个文件，但是实际操作发现如果这个文件不配置，启动的时候会失败。

在完全分布式模式下，这个文件中除了配置了JAVA_HOME外，还配了如下一些内容：

```

export HDFS_NAMENODE_USER=root

export HDFS_DATANODE_USER=root

export HDFS_SECONDARYNAMENODE=root

```

现在新加了journalnode和zkfc，也需要类似的增加如下配置：

```

HDFS_JOURNALNODE_USER=root

HDFS_ZKFC_USER=root

```

到这里，HA模式HDFS本身的配置就算完了，但是可以看到上边需要使用zookeeper，而完全分布式没有这个，所以还需要先再规划的节点中安装zookeeper。

## zookeeper搭建和启动

首先要获取安装包，可以从官网获取到最新的，地址如下：

https://downloads.apache.org/zookeeper/current/

之后使用tar解压，具体步骤就不在赘述。解压完成之后，需要把安装目录的conf目录下的zoo_sample.cfg文件重命名为zoo.cfg，之后修改这个文件。

### zoo.cfg修改

首先，这个文件中有一个默认的`dataDir`，和hadoop一开始一样配置的是linux系统的临时目录，这个目录需要修改为一个非临时目录，例如我这里修改为`/root/soft/bigdata/zookeeper/data`。

然后，需要配置zookeeper集群各个节点，例如我这里的配置如下：

```

server.1=node01:2888:3888

server.2=node02:2888:3888

server.3=node03:2888:3888

```

上边server后边的数字并不一定要和这里一样，可以理解为就是一个权重大小，为的就是zookeeper本身的选举。

后边跟了两个端口号，一个是有主，或者说有leader的情况下相互通信使用的端口，一个是无主状态下通信使用的端口号。

### myid

有了上边的配置后，还需要在zookeeper数据目录中新增一个权重文件，把上边配置的权重值写入这个文件，提供给zookeeper读取。

例如我这里再node01的`/root/soft/bigdata/zookeeper/data`执行如下操作：

```

echo 1 > myid

```

在node02的`/root/soft/bigdata/zookeeper/data`执行如下操作：

```

echo 2 > myid

```

同样的，在node03的`/root/soft/bigdata/zookeeper/data`执行如下操作：

```

echo 3 > myid

```

### zookeeper环境变量

之前不论是安装jdk、redis，还是hadoop，都配置了各自的环境变量，目的都是为了能在任意目录方便的执行相应的命令，进行相应的操作，zookeepere也是一样，最好是配置一下环境变量。

### 启动和验证

除了环境变量以外，hadoop本身安装包以及配置都可以只在一台机操作，然后需要分发，或者说复制到其他两台机，然后依次配置好环境变量后，就可以启动zookeeper集群，并验证运行状态。

`zkServer.sh start`可以启动zookeeper服务，`zkServer.sh status`可以查看zookeeper运行状态。

当我们现在一台机执行start后，立即用status会看到显示`not running`，而使用`ps`又能看到zookeeper进程，原因是只有一个zookeeper的情况下，无法选举，所以有进程却无法提供服务。

当再启动一个之后，就会看到权重值较大的会变成`leader`，而权重值小的会变成`follower`，三台都启动以后，就会一个leader，两个follower。

## HA模式HDFS初始化及启动

之前说了，namenode实际是管理datanode，而zookeeper和journalnode的作用又是提高namenode可用性，可以简单看成是对namenode的一种管理。

所以正常情况是namenode在datanode之前运行，而zookeeper和journalnode又需要在namenode之前运行。所以，zookeeper集群正常运行之后，需要再启动journalnode：

```

hadoop-daemon.sh start journalnode

```

因为是重新搭建，和完全分布式一样需要先初始化，之前也说过，这个初始化的作用是清空原有数据，然后会新建一些文件，正常来说这二个操作只应该在环境搭建的时候执行一次，其实时候甚至应该禁用，格式化操作如下：

```

hdfs namenode -format

```

name主从集群通信，需要依据同一个集群id，上边的初始化操作就会生成一个集群id，所以另一个namenode节点也需要存储这个一样的id信息，要从上边同步过来，通过之前需要上边的namenode先运行起来：

```

hadoop-daemon.sh start namenode

```

然后便是同步，上述操作我再master节点操作，即规划的其中一个namenode节点，所以下边的操作就需要再另一个namenode节点，例如node03中：

```

hdfs namenode -bootstrapStandby

```

namenode配置同步之后，就需要再初始化`ZKFC`，使namenode和zookeeper连接起来，这个操作一样是在开始的namenode节点上，例如我这里的master机器。

```

hdfs zkfc -formatZK

```

上述操作之后，基本的搭建就算完成了，然后在活跃的namenode节点执行hdfs启动命令启动即可：

```

start-dfs.sh

```

不出意外的话，启动完成之后，使用hdfs相关命令就可以进行文件的增删改查操作了。

这时候如果要停止，可以一个个的停止，也可以直接使用`stop-all.sh`命令一次停止namenode、journalnode等，zookeeper是单独的，就还是需要单独操作。

后边如果再需要启动HA模式的HDFS集群，也可以直接使用`start-all.sh`进行启动。

## HA验证

一开始说了HA的目的其实就是为了解决namenode单点故障问题，所以需要进行自动选主验证，即，在确定某个namenode是active的情况下是他不可用，然后看另一个是否能正常自动成为activi。

看哪个namenode是active的，有两种方法，一是访问web界面，即hdfs-site.xml中的`dfs.namenode.http-address`配置，例如我上边配了`master:9870`和`node03:9870`，浏览器访问的时候，就能看到具体哪个是active状态。

另一个是在zookeeper中，可以使用`zkCli.sh`进入到zookeeper自带的客户端操作界面，然后使用get获取到当前哪个节点持有锁，例如我配置的namenode集群名叫`mycluster`，则我查询锁状态的命令为：

```

get /hadoop-ha/mycluster/ActiveStandbyElectorLock

```

上述命令也可以看到具体是哪个节点持有锁，在知道当前active节点的情况下，就可以使用`hadoop-daemon.sh stop namenode`来停止这个namenode，然后观察另一个是否会自动变成active，如果变了，并且整个hdfs服务使用正常，则可证明这个HA模式基本搭建成功。