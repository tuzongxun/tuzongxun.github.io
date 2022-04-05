---
title: hadoop和hbase的关系及hbase安装与验证
date: 2020-8-10 19:59:42
tags: hadoop
toc: true
categories: hadoop
---
从hadoop官网首页就可以看到，hadoop项目本身自带的模块现在有五个，即：
>hadoop common
hdfs
yarn
mapReduce
hadoop Ozone

第一项从名称就可以看出来是基础功能模块，hdfs是文件存储系统，yarn是调度和集群管理，mapReduce是数据计算处理，这几个都是学习使用hadoop一开始就必然会接触的。
最后一个hadoop Ozone是分布式对象存储系统，这个是对hdfs的一种补充，是一个相对较新的内容，在人们口中出现的频率相对较低，可能很多人一开始都不知道，包括我。

<!--more-->
从这里也可以看出来，听说大数据的人一般都会听到的hbase、hive、spark等，其实并不属于hadoop项目本身，而是可以基于hadoop使用的的其他关联项目。
对于hadoop本身的模块来说，只要hadoop相应版本支持，那么只要安装了hadoop就可以使用这些模块功能，而关联项目，就必须独立安装和部署，然后在配置中关联起来。

hadoop可以做很多事，但是提到他，肯定第一时间是想到大数据，而提到大数据，就一定会想到数据处理计算和数据存储。
如果先抛开hadoop Ozone不谈，目前我了解的大数据系统中数据存储一般都离不开hdfs和hbase，个人理解，hbase从某种意义上来说，也算是hdfs的一种补充，因为从前边了解的就可以知道，hdfs是不支持文件内容修改的，有利也必然就有弊。
我们知道数据库最终也是以文件形式持久化存储的，不论是mysql、mongodb还是hbase，hbase现在的底层文件系统就是通过hdfs来支持的，这个我验证过没有问题，但网上说换成其他文件系统也没问题，这个需要后续进一步验证。
基于以上思路，在初步学习了hdfs后，就应该是再对hbase进行一个基础扫盲，安装、配置和基础操作。

## hbase下载
hbase可以从官网找到安装包下载页面http://hbase.apache.org/downloads.html，这个页面罗列了很多版本，通过查看release notes信息，我看到2.2.5版本已经支持3.2.x版本的hadoop，而我的hadoop是3.1.3，所以就选择了这个版本。
安装包获取方式多种多样，在redis安装和hadoop安装文章中都有讲过，想了解的可以移步
[hadoop安装环境准备和关联知识解析](https://blog.csdn.net/tuzongxun/article/details/107811747)
[Linux中redis安装及软件安装相关Linux知识要点](https://blog.csdn.net/tuzongxun/article/details/107170447)
我这里就直接使用最快捷的方式：
```
wget https://downloads.apache.org/hbase/2.2.5/hbase-2.2.5-bin.tar.gz
```

## 解压
```
tar -zxvf hbase-2.2.5-bin.tar.gz
```

## hbase-env.sh配置
解压完之后及时配置，首先要配置的是hbase-env.sh这个文件，这个文件在安装目录的conf目录下，例如我的hbase安装目录是`/root/soft/bigdata/hbase/hbase-2.2.5`，则这个文件路径是`/root/soft/bigdata/hbase/hbase-2.2.5/conf/hbase-env.sh`，这个文件中需要配置如下内容
```
export JAVA_HOME=/root/soft/jdk1.8.0_261
export HBASE_CLASSPATH=/root/soft/bigdata/hbase/hbase-2.2.5/conf
export HBASE_MANAGES_ZK=true
```
要注意的是实际操作是换成自己的jdk和hbase的安装路径。

## hbase-sit.xml配置
然后要配置的就是hbase-sit.xml，这里需要指定hdfs系统中存储hbase数据的目录，由于我的hadoop是分布式模式，所以配置中还需要开启分布式模式，这些配置都需要放在`<configuration>`和`</configuration>`中间：
```
<property>
      <name>hbase.rootdir</name>
      <value>hdfs://192.168.139.9:9000/hbase</value>
</property>
<property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
</property>
<!--
<property>
      <name>hbase.tmp.dir</name>
      <value>./tmp</value>
</property>
-->
<property>
      <name>hbase.unsafe.stream.capability.enforce</name>
      <value>false</value>
</property>
```
以上几项配置，第一项就是指向了我搭建的hdfs文件系统，其中hbase还是一个不存在的目录，后续使用过程中会自动创建。
第二项配置就是针对我的hadoop分布式模式的配置，hbase默认这个配置是false，也就是单机模式。
第三项看起来是一个临时目录，hbase这个配置文件中原本自带的，暂时不确定用处，我就先注释了。
第四项，也是hbase配置文件自带的，看网上解释，是避免出现启动错误，我就先保持不变。

## hbase环境变量配置
这是一个习惯性的操作，可以使操作更加方便，能够任意目录执行hbase相关命令，不想配的也可以不配置：
```
export HBASE_HOME=/root/soft/bigdata/hbase/hbase-2.2.5
export PATH=$PATH:$HBASE_HOME/bin
```

## ServerNotRunningYetException
按网上教程，有了上边配置应该就可以启动了，前提是先保证hadoop已经启动完成。
我在启动了hadoop之后，执行hbase启动命令`start-hbase.sh`发现结果报错：
```
ERROR: org.apache.hadoop.hbase.ipc.ServerNotRunningYetException: Server is not running yet 
```
网上搜了下，说是因为hdfs开启了安全模式，但我使用`hdfs dfsadmin -safemode get`查看hdfs安全模式状态后，发现我这里本身就是关闭的，证明这个错误的原因必然不止这一个。(注：`hdfs dfsadmin -safemode enter`可以手动开启，`hdfs dfsadmin -safemode leave`可以手动关闭)
于是查了下hbase的启动日志，最终看到原来是自己粗心，在配置hdfs的ip时，`192.168.139.9`错误的打成了`12.168.139.9`，把ip改成正确的之后便成功启动。

## multiple SLF4J bindings
虽然hbase启动成功了，但是启动时却出现了告警提醒：
```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/root/soft/bigdata/hadoop/hadoop-3.1.3/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
```
这个提醒还是很明确的，就是hbase和hadoop相关的日志依赖有冲突，于是给hbase中的日志jar改了名，然后就好了。

## 验证
### shell连接
hbase启动成功后，像mysql、mongodb、redis一样，hbase也有自带的shell客户端工具，可以连接进行操作：
```
hbase shell
```
执行上述命令后，就能进入到hbase的命令行操作界面，或许是我机器原因，进入时间有点久。

### 创建表
hbase中创建一个表结构的简单操作如下：
```
create 'user','name','age','addr','phone','email'
```
上述命令的意思是，创建一个名称为user的表，包含name、age、addr、phone、email等属性。

### Master is initializing
在上边创建表的时候，发生了一个小插曲，第一次创建时抛出了如下异常：
```
ERROR: org.apache.hadoop.hbase.PleaseHoldException: Master is initializing
        at org.apache.hadoop.hbase.master.HMaster.checkInitialized(HMaster.java:2811)
        at org.apache.hadoop.hbase.master.HMaster.createTable(HMaster.java:2018)
        at 
```
从字面意思看，就是master正在启动中，于是我稍等了一会儿重新执行了下，就成功创建了。

### 查看表结构
创建了hbase表之后，可以使用describe命令查看表结构，例如
```
describe 'user'
```
我这里上边的命令输出如下：
```
Table user is ENABLED                                                       
user                                                                        
COLUMN FAMILIES DESCRIPTION                                                 
{NAME => 'addr', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERS
ION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE 
=> 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS =>
 '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE =
> 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_
BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOC
KSIZE => '65536'}                                                           
{NAME => 'age', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSI
ON_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE =
> 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => 
'0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE =>
 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_B
LOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCK
SIZE => '65536'}                                                            
{NAME => 'email', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VER
SION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE
 => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS =
> '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE 
=> 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH
_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLO
CKSIZE => '65536'}                                                          
{NAME => 'name', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERS
ION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE 
=> 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS =>
 '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE =
> 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_
BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOC
KSIZE => '65536'}                                                           
{NAME => 'phone', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VER
SION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE
 => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS =
> '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE 
=> 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH
_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLO
CKSIZE => '65536'}                                                          
5 row(s)
QUOTAS                                                                      
0 row(s)
Took 4.6373 seconds                     
```
至此，证明hbase安装配置也确实是可用的。

## 停止hbase
hbase停止原本很简单，启动是`start-hbase.sh`，那么正常里说停止就是`stop-hbase.sh`，实际上也确实是的。
但是我在停止的时候却遇到一个小问题，第一次停止的时候，一直在stopping状态。
网上说需要先执行`hbase-daemons.sh stop regionserver`，我试了下，确实很有效，然后再执行`stop-hbase.sh`就立即停止了，只不过，这个现象也仅出现了一次，后边每次直接`stop-hbase.sh`都能很快停止。

以上部分内容参考如下文章：
[http://dblab.xmu.edu.cn/blog/2442-2/](http://dblab.xmu.edu.cn/blog/2442-2/)