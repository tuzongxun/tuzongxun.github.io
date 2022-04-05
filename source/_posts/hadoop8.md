---
title: hbase-ha模式搭建要点和问题记录
date: 2020-9-23 19:59:42
tags: hadoop
toc: true
categories: hadoop
---
之前搭建了单机的hbase，使用伪分布式的hdfs作为数据存储，具体搭建要点和问题有所记录：
[https://blog.csdn.net/tuzongxun/article/details/107915720](https://blog.csdn.net/tuzongxun/article/details/107915720)
后来，伪分布式的hdfs升级为ha模式，hbase自然也是要同步升级成ha的，本以为应该会很顺利，但实际上花的时间还是比预想中的多，因此还是做一个简单的记录，尤其是其中卡住的问题。
<!--more-->
## 机器规划
本次hbase-ha模式搭建规划使用三台机，主机名分别是`node001`、`node002`、`node003`，其中node001为master，node003为back master，而三台都作为regionserver。

## 准备条件
hbase-ha模式搭建，按官网说明，以及目前验证来看，似乎必须要用hdfs，因此主要的依赖条件如下：
>jdk
hdfs-ha集群
zookeeper集群
ssh免密

## hbase-env.sh
之前单机的时候，zookeeper也是使用的hbase自带的，因此在`hbase-env.sh`文件中有这样一行：
```
export HBASE_MANAGES_ZK=true
```

在搭建ha模式的hdfs时已经搭建好了独立的zookeeper集群，所以hbase的ha自然也可以直接使用这个zookeeper集群，而不再使用自带的，因此这里需要修改成不使用自带的：
```
export HBASE_MANAGES_ZK=false
```

## hbase-site.xml
hbase主要的一些配置都在这个文件里，先列一下最终的文件主要内容：
```
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://mycluster/hbase3</value>
</property>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.tmp.dir</name>
    <value>/var/bigdata/hbase/data-local3</value>
</property>
<property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
</property>
<property>
    <name>hbase.master</name>
    <value>node001:60000</value>
 </property>
 <property>
    <name>hbase.zookeeper.quorum</name>
    <value>node002,node003,node004</value>
 </property>
```
上述第一个配置`hbase.rootdir`指向hbase的存储路径，这里还是使用hdfs存储，但是不同的是，把之前固定的ip和端口改成了hdfs集群的名称。
之后的`hbase.cluster.distributed`为`true`开启分布式，`hbase.tmp.dir`配置临时存储目录，`hbase.unsafe.stream.capability.enforce`这个配置保持不变，为了保证启动不报错（具体原因待深究），`hbase.master`指定一个主节点(网上说这个可以不配置，交给zookeeper来选择，时间问题暂时未验证)，`hbase.zookeeper.quorum`配置外部zookeeper的节点。

## regionservers
根据官网说明，需要把集群中的region节点主机写入到这个文件中，这个文件也在`conf`目录下，只不过原本里边只有一个`localhost`，修改之后如下：
```
node001
node002
node003
```

## backup-masters
这个文件原本是没有的，是一个master备份节点的配置，可以不要。但是正常的ha模式一般来说是需要有备份master节点的，所以还是需要这个文件。由于原本没有，就需要手动在`conf`目录下创建并制定备份master节点，例如：
```
node003
```


## core-site.xml和hdfs-site.xml
原以为有了上边的配置就可以了，但是使用`start-hbase.sh`启动时，发现`hbase-root-regionserver-node001.log`日志出现如下的error信息：
```
Caused by: java.lang.IllegalArgumentException: java.net.UnknownHostException: mycluster
        at org.apache.hadoop.security.SecurityUtil.buildTokenService(SecurityUtil.java:417)
        at org.apache.hadoop.hdfs.NameNodeProxiesClient.createProxyWithClientProtocol(NameNodeProxiesClient.java:132)
        at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:351)
        at org.apache.hadoop.hdfs.DFSClient.<init>(DFSClient.java:285)
        at org.apache.hadoop.hdfs.DistributedFileSystem.initialize(DistributedFileSystem.java:160)
        at org.apache.hadoop.fs.FileSystem.createFileSystem(FileSystem.java:2812)
```

上边的`mycluster`并非主机名，而是hdfs的集群名称，看起来是这里不能识别。经过查询，解决方案是复制hadoop里边的`core-site.xml`和`hdfs-site.xml`到hbase的`conf`目录下，然后重新启动。

## Master is initializing
上边操作之后重新启动hbase不在抛error异常，原以为这次成功了，但是使用`hbase shell`操作时却不能正常的操作，而是在命令行界面跑出如下异常：
```
ERROR: org.apache.hadoop.hbase.PleaseHoldException: Master is initializing
        at org.apache.hadoop.hbase.master.HMaster.checkInitialized(HMaster.java:2811)
        at org.apache.hadoop.hbase.master.HMaster.createTable(HMaster.java:2018)
        at org.apache.hadoop.hbase.master.MasterRpcServices.createTable(MasterRpcServices.java:659)
        at org.apache.hadoop.hbase.shaded.protobuf.generated.MasterProtos$MasterService$2.callBlockingMethod(MasterProtos.java)
        at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:418)
        at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:133)
        at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:338)
        at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:318)
```

这个异常在搭建单机hbase的时候实际已经出现过，但是当时等了一会儿就好了，而这次这个异常却没有随时间变化而解决，在网上找了很多解决方案均是失败的情况下，我重新一行行的看了下日志，结果发现在`hbase-root-master-node001.log`文件中有如下几行：
```
2020-09-23 09:17:17,383 WARN  [master/node001:16000:becomeActiveMaster] master.HMaster: hbase:meta,,1.1588230740 is NOT online; state={1588230740 state=OPEN, ts=1600823703586, server=node001,16020,1599788244145}; ServerCrashProcedures=true. Master startup cannot progress, in holding-pattern until region onlined.
```
这个日志只是WARN，所以一开始没有注意到，仔细读了之后猜测可能就是我shell无法操作的原因，于是通过搜索这个问题找到了解决方案，目前不完全确定根本原因，但猜测应该是zookeeper中对于hbase的状态管理出现了异常数据。
因为我的hbase不是直接搭建成ha模式，而是从单机使用本地文件改成hdfs，又改成ha模式，这期间为了处理和验证各种异常还手动删除过`/tmp`等目录下的内容，不确定是否这些操作导致zookeeper里出现了不一致的数据。
最终解决办法就是删除zookeeper里`meta-region-server`，具体操作是：
```
zkCli.sh
deleteall /hbase/meta-region-server
```

实际上网上的答案几乎千篇一律是`rmr /hbase-unsecure/meta-region-server`，但是我发现我的zookeeper进去之后并没有`rmr`命令，也没有`hbase-unsecure`目录，后来看到有地方说`rmr`是过期指令，多半是我的zookeeper-3.6.1比较新，已经完全删除了这个操作，取而代之的是`deleteall`。
而`hbase-unsecure`应该是和`mycluster`一样只是一个集群名字，而我这里就叫`hbase`。
里边诸多细节还有待深究和验证，好在最终再重启hbase之后，不论是`hbase shell`还是`phoenix`都能正常操作了。

注：除了细节上，hbase本身ha搭建还是较简单，需要注意的是需要保证各集群节点内的配置文件一致，包括从hadoop复制来的文件。