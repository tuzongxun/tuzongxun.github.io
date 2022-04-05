---
title: hadoop分布式安装及配置初步解析（坑坑不息）
date: 2020-8-6 16:59:42
tags: hadoop
toc: true
categories: hadoop
---
linux中hadoop的安装教程，网上也有不少了，例如我自己搭建过程中参考的这几篇：
[https://blog.csdn.net/weixin_44198965/article/details/89603788](https://blog.csdn.net/weixin_44198965/article/details/89603788)
[https://blog.csdn.net/qq_25615395/article/details/89083580](https://blog.csdn.net/qq_25615395/article/details/89083580)
[https://juejin.im/post/6856984821059895303/](https://juejin.im/post/6856984821059895303/)

然而教程是不少，但或许是环境不一样，也或者思路不一样，所以参考实现的过程中总会发现这样那样的一些问题。
当然了，这些问题产生的原因很可能是每个人技术栈不一样，别人以为你应该知道的，实际你不知道，也就导致有些没说的细节就成了自己动手时的拦路虎。
所以，结合上边的几篇教程，再结合自己实际操作，我觉得还是可以记录一下的，万一刚好有看的人容易接受我的描述思路呢，那么这篇文章便有了更多的意义。

<!--more-->
## 解压
上一篇[hadoop安装环境准备和关联知识解析](https://blog.csdn.net/tuzongxun/article/details/107811747)中，我已经下载好了安装包，所以这里只需要先解压一下：
```
tar -zxvf hadoop-3.1.3.tar.gz
```

分布式部署，应该是至少需要3台机的，所以我准备了3台，分别是192.168.139.9、192.168.139.19、192.168.139.29，都是centos6.5的虚拟机，分配内存1G。
要提一点的是，开篇三个链接的第三个，是我同事写的，他就遇到了虚拟机内存只分配了512M，结果运行时内存溢出。

三台机的部署，可以一个个的分别解压然后配置运行，也可以先只在一台机上解压并配置，之后运用上一篇提到过的scp等操作复制到其他两台机，我这里便选择一台机解压并配置，然后再复制到另外两台机的方式。

## 配置
基础的分布式模式搭建，主要是配置如下几个文件，这里说的基础就是以启动并检测运行状态成功为目标。
>hadoop-env.sh
core-site.xml
hdfs-site.xml
yarn-site.xml
mapred-site.xml
workers

上边几个文件，都在hadoop安装目录下的`etc/hadoop`目录里边，例如我的安装目录是`/root/soft/bigdata/hadoop/hadoop-3.1.3`，则第一个文件的路径即`/root/soft/bigdata/hadoop/hadoop-3.1.3/etc/hadoop/hadoop-env.sh`。
有的教程里边可能就列了上边几个文件，但是实际操作的时候发现3.1.3版本实际还必须修改下边几个，否则也会启动失败，2.7版本可以不改。
>start-dfs.sh
stop-dfs.sh
start-yarn.sh
stop-yarn.sh

上边几个文件换了个目录，在安装目录下的`sbin`目录中，这部分的配置我会放在执行`start-all.sh`启动失败时再讲。

### hadoop-env.sh
这个文件里目前就修改一下JAVA_HOME就可以了，如下：
```
export JAVA_HOME=/root/soft/jdk1.8.0_261
```
这里实际有个疑问，因为本身我的机器是配置全局的JAVA_HOME的，原本我想着直接里边写的`JAVA_HOME=$JAVA_HOME`应该能直接读取到系统环境变量的，但实际操作的时候并不是想象的这样，所以还必须改这个文件，指向jdk实际路径。

### core-site.xml
```
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://master:9000</value>
</property>
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/hadoop/hadoopdata</value>
</property>
```
上述两个配置，需要加到`<configuration>`和`</configuration>`中间，下边几个也都一样。

两个配置的含义，第一个我查了不少资料，目前还没找到准确的说明，留待后边一步步深入之后再来解释。
可以说的是，里边配置的master实际是机器的hostname，即`/etc/sysconfig/network`这个文件里的`HOSTNAME`的值，也可以是ip地址，这两个都亲测过，可用。
这里的master是我这样命名的，并不一定要叫这个。
还有一种说法是也可以配置域名，只不过一般情况下我们自己是用不上这个的，我没试，理论上应该也是可行的。
第二个配置，是指向一个临时的hadoop存储目录，有的教程说需要事先手动创建好，但实际操作的时候我并没有手动创建，而是执行后边的`hadoop namenode -format`时自动会创建，也就是说这里实际是可以指向一个不存在的目录的。

### hdfs-site.xml
```
<property>
	<name>dfs.replication</name>
	<value>3</value>
</property>
```
上边这个配置，看起来是副本集数量，但实际操作的时候我改变了数字，结果显示的活动节点数依然保持不变，所以又是需要后续理解的问题，这里只是照别人教程先依葫芦画瓢。

### yarn-site.xml
```
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>
<property>
	<name>yarn.resourcemanager.address</name>
	<value>master:18040</value>
</property>
<property>
	<name>yarn.resourcemanager.scheduler.address</name>
	<value>master:18030</value>
</property>
<property>
	<name>yarn.resourcemanager.resource-tracker.address</name>
	<value>master:18025</value>
</property>
<property>
	<name>yarn.resourcemanager.admin.address</name>
	<value>master:18141</value>
</property>
<property>
	<name>yarn.resourcemanager.webapp.address</name>
	<value>master:18088</value>
</property>
```
上述配置具体含义也需要后续学习理解的过程中不断补充，里边的master同样的是主机名，也可以用ip，上边有过解释。而后边的端口，如果对比别的教程会发现很多都不一样，因此也是自定义的端口，作用应该就是hadoop启动时在这些端口上启动进程，所以保证和本机其他应用端口不冲突就好了。
需要先说一点的是，`yarn.resourcemanager.webapp.address`这个配置实际就是后边可以验证集群活动节点数的，可以再浏览器访问。

### mapred-site.xml
```
<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>
```
上边的配置仅从字面看，似乎很好理解，但实际作用依然还不是特别明了，只知道默认的是local，这里需要改成yarn，也要留到后边再多看下官网文档以及其他资料再做理解。

### workers
这个文件，我的理解是配置副本节点，这个才会真的影响到后边看到的活动节点数量，而不是上边的`dfs.replication`。
这个文件在2.7版本的hadoop中叫slaves，在3.1.3版本中叫workers，我这里的配置如下：
>#localhost
node01
node02

上边文件默认有一行localhost，所以可能有的人单机运行伪分布式模式时，会发现不改这里似乎也可以，因为本身都是本机，都指向了`localhost`，亦`127.0.0.1`。
我这里配置的两个，分别是192.168.139.19和192.168.139.29的hostname，这里要注意的是，似乎必须是hostname，并且要做host映射，即修改`/etc/hosts`文件，我这里三台机配置如下：
```
192.168.139.19 node01
192.168.139.29 node02
192.168.139.9 master
```
原本我想着这里只是一个映射关系，那么是不是可以改成和hostname不一样的虚拟域名，反正都是要配置hosts的，但是实际上把配置改成和hostname不一样时，后边启动就有问题，执行`start-all.sh`时，两个从节点怎么都启动不了。
如上边的配置，我的workers里配置了两个节点，那么后续启动以后在web界面就会看到两个活动节点，如果这里再加上主机名master，则后续web界面就能看到三个活动节点。

## 配置hadoop环境变量
有了上边的配置，已经几近于大功告成了，可以直接到hadoop安装目录的bin目录以及sbin目录执行各种操作。
但是为了更加方便，为了随便哪个目录下都能方便的执行hadoop相关命令，就需要再配置一下hadoop的环境变量，配置方式和JAVA_HOME、REDIS_HOME如出一辙：
```
export HADOOP_HOME=/opt/hadoop/hadoop
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```
注：上边的配置，实际上分为永久配置和临时配置，临时配置即直接在bash命令行一次执行上边的操作，这里在当前运行阶段，也有说是当前登录会有效，机器重启就没了。永久配置，则要把上述代码放到`/etc/profile`文件中，然后执行`source /etc/profile`，所以上边的环境变量配置最好就是修改文件，我也是这样做的。

## 格式化（初始化）
环境变量之后，就算是纯安装阶段最后一步操作了，需要在主节点执行格式化操作，看起来更像是初始化。
```
hadoop namenode -format
```
上边的操作基于hadoop环境变量的配置，如果没有配置环境变量，或者配置有问题，可能提示hadoop命令找不到。
这个操作，就会初始化`core-site.xml`中配置的`hadoop.tmp.dir`目录，我这里就是`/opt/hadoop/hadoopdata`，原本是不存在这个目录的，执行了上边命令后就会自动创建，并且里边会生成一些子目录及文件。
这里要说的是，有的教程说几个节点都需要把这里边的内容复制过去，但是我并没有复制，目前看来似乎没有影响。
还有说法是里边有一个VERSION文件，里边的`clusterID`、`datanodeUuid`、`storageID`会影响到web界面看到的活动节点数量，我没有复制，也没有生成，似乎也没有影响看到的数量。

## 启动hadoop集群（此处有坑）
至此，照葫芦画瓢式的搭建算是完成了，然后剩下的好像就是启动和验证，按照一些网上的教程，就是执行`start-all.sh`启动，然后执行`jps`查看启动情况，于是我便在执行了上述`hadoop namenode -format`命令所在的机器中运行了启动命令：
```
start-all.sh
```

对于最开始2.7版本的，似乎就是这样了，执行上边命令就会提示要输入另外两台机的ssh密码，依次在提示的时候输入相应的机器登陆密码即可启动。
只是2.7版本我也只是走到了这一步，没有再进一步验证。

而为什么上边说好像是搭建完成，就剩启动和验证了呢，是因为3.1.3版本其实还没有搭建结束。
在执行上述命令时，会发现先抛出如下异常：
```
ERROR: Attempting to operate on yarn nodemanager as root
ERROR: but there is no YARN_NODEMANAGER_USER defined. Aborting operation.ls
```
网上搜索之后发现是要配置一些user权限，于是就还需要修改开头提到的`start-dfs.sh`、`stop-dfs.sh`、`start-yarn.sh`、`stop-yarn.sh`这四个文件，在两个dfs.sh文件开头加入如下配置：
```
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```
在两个yarn.sh文件中加入如下配置：
```
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

有了上边的配置，再次启动，会发现输出了如下的一些提示：
```
node02: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
node01: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)
```
之后就结束了，没有像2.7一样提示让输ssh密码，同时，执行`jps`会发现本机显示的远不是其他资料所说成功的样子，而另外两台机执行`jps`后更是除了输出`jps`，啥都没有。
之所以这样，是因为3.1.3版本必须配置sshkey，2.7也可以这样做，但似乎不是强制，而3.1.3没有了选择。

## sshkey配置
sshkey的作用就是在ssh连接的时候免密，互相配置公私钥进行交互，步骤如下：
```
ssh-keygen -t rsa
cat id_rsa.pub >>  authorized_keys
scp authorized_keys root@192.168.139.19:/root/.ssh/authorized_keys
```
第一行是在本机（我这里即192.168.39.9）生成公私钥，第二行是把公钥追加写入文件authorized_keys中，第三步是复制上边的authorized_keys文件到另一台机。
这里就会有一个问题，因为192.168.139.19机器还没有生成公私钥，所以是没有.ssh目录的，执行第三行机会提示目录或文件不存在，不知道是否所有linux系统内都这样。
因此更准确的做法是先依次在三台机生成自己的公私钥，第二步是把第一台机的公钥追加到authorized_keys文件，第三步是把authorized_keys复制到其他机器。
网上有人说需要把几台机的公钥都追加写入到一个文件，然后再复制到每台机，但是个人以为可以不用。
因为就目前实操来说，只有在namenode机器执行`start-all.sh`命令，才能正常启动其他机器，即便是按上边说的把几个公钥都追加到一个文件中，然后复制到每个机器，结果在非namenode节点执行启动命令时，会发现最终找不到`ResourceManager`。
只是，几乎所有文章都是这样说的，而我实际操作就不是这样，究竟是其他文章不太对，还是我刚好走了一条过程不对，结果看起来也对的路，这就不得而知了。

到这里，才算是真的配置搭建完成，再回到主节点执行`start-all.sh`启动，就会看到启动成功，然后执行jps验证几个节点，就会有如下的输出：
```
[root@master hadoop]# jps
52709 NodeManager
52008 NameNode
52329 SecondaryNameNode
52589 ResourceManager
53935 Jps

[root@master ~]# jps
1552 SecondaryNameNode
1937 Jps
1340 NameNode
1789 ResourceManager
```

不少教程到这里似乎就结束了，好像这样就代表成功了，然而并不是这样，这个只是看起来成功了，或者说只是单节点启动成功了而已。

## 节点交互问题
在上边`yarn-sit.xml`配置中，有如下一段：
```
<property>
	<name>yarn.resourcemanager.webapp.address</name>
	<value>master:18088</value>
</property>
```
这里是可以通过浏览器访问，然后看到节点运行情况的，但是我访问的时候却看到活动节点数是1，
而且就是执行启动命令的这个节点是活动的。（这里注意，虚拟机安装配的是虚拟域名，如果是外部物理机浏览器访问，需要在物理机的hosts中配置虚拟域名映射，否则访问不通）
这就是一个很尴尬的问题，于是通过查资料和日志，发现192.168.139.19和29日志里有这样的内容：
```
org.apache.hadoop.ipc.Client: Retrying connect to server: master/192.168.139.9:18025. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
ERROR org.apache.hadoop.yarn.server.nodemanager.NodeManager: RECEIVED SIGNAL 15: SIGTERM
```
看起来就是连接不上yarn资源管理器，最终发现是因为hosts配置中，把master映射到了127.0.0.1，然后其他节点连接yarn资源管理器时就导致master转到127.0.0.1，然后就出现回环。
于是，把`/etc/hosts`里边127.0.0.1后边的master映射去掉，先执行`stop-all.sh`停止，再执行`start-all.sh`启动，然后再web界面便可以看到活动节点数不再是1了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806162621464.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3R1em9uZ3h1bg==,size_16,color_FFFFFF,t_70)