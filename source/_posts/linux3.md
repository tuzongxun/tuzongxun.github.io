---
title: centos7中redis、mongodb、kafka安装记录
date: 2020-9-8 23:09:42
tags: linux
toc: true
categories: linux
---
一个完整的java后台系统，通常会涉及到非常多的技术，例如数据库、缓存、消息中间件等，除此之外，从部署层面讲，可能还离不开nginx、docker这些，要更加熟练的使用这些技术，加深理解，必不可少的需要有自己的环境。
随着上次centos6.5的系统升级到centos7.8，打算把hadoop、redis、mongodb、kafka等这些软件都迁移到新的虚拟机系统中，docker和hadoop的安装部署最近都有相关记录，这次先补充redis、mongodb和kafka。
<!--more-->
## redis安装
redis的安装，实际上也是最近有过记录，可以参考[Linux中redis安装及软件安装相关Linux知识要点](https://tuzongxun.blog.csdn.net/article/details/107170447)。
需要说明的是，系统升级到centos7.8后，安装过程有一点波折，解决办法参考如下转载的两篇文章：
[https://tuzongxun.blog.csdn.net/article/details/108461134](https://tuzongxun.blog.csdn.net/article/details/108461134)
[https://tuzongxun.blog.csdn.net/article/details/108461618](https://tuzongxun.blog.csdn.net/article/details/108461618)

## mongodb安装
### 下载安装包
在https://www.mongodb.com/try/download/community这个页面选择需要的安装类型、版本和环境，然后复制下载链接，例如https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.4.0.tgz，然后可以在linux中使用wget下载安装包：
```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.4.0.tgz
```
### 解压
使用tar解压安装包:
```
tar xf mongodb-linux-x86_64-rhel70-4.4.0.tgz
```
我规划的安装目录是`/opt/mongodb`，解压完的名字太长就重命名了下，之后整个安装目录为`/opt/mongodb/mongodb4.4`。

### 配置
mongodb可以什么都不配置，完全使用默认方式启动，但是启动可能会报错，因为默认需要几个目录。
但是为了一步到位，我选择直接使用配置文件方式启动，在`/opt/mongodb/mongodb4.4`目录下创建conf目录，然后在目录中创建mongodb.conf文件并加入如下内容：
```
dbpath=/opt/mongodb/mongodb4.4/data/
logpath=/opt/mongodb/mongodb4.4/logs/mongodb.log
fork=true
port=27017
bind_ip=0.0.0.0
auth=false
```
要注意的是，上边目录和文件名的命令都是可以自定义的。
上述内容含义如下：
>指定数据存储目录；
指定日志存储目录；
开启后台运行；
指定端口号（27017默认，不指定也可以）；
配置绑定ip，0.0.0.0可以任意机器访问；
关闭用户权限认证

上述配置中有指定目录和文件，和hadoop配置文件里的目录和文件会自动创建不同，mongodb这里的不会自己创建，所以相关目录需要提前创建好。

### 配置环境变量
这是个常规操作，就不多说了，目的是为了更方便的执行相关命令，而不用每次都进入或者层层指定安装目录。
之前有java、redis，加入mongodb后的`/etc/profile`相关内容如下：
```
export JAVA_HOME=/opt/java/jdk1.8.0_261
export REDIS_HOME=/opt/redis/redis6
export MONGODB_HOME=/opt/mongodb/mongodb4.4
export PATH=$PATH:$JAVA_HOME/bin:$REDIS_HOME/bin:$MONGODB_HOME/bin
```

### 启动和验证
mongodb安装较简单，有了上边配置，即可使用配置文件方式启动，例如：
```
mongod -f /opt/mongodb/mongodb4.4/conf/mongodb.conf
```

然后可以使用mongo命令进入自带的客户端操作界面验证是否真的安装部署成功。

## kafka安装
kafka之前一直在用，但是都是用的公司开发环境上的，自己的环境上实际还是第一次安装，不过，这些安装也都大同小异：
### 下载安装包
```
wget https://downloads.apache.org/kafka/2.6.0/kafka_2.12-2.6.0.tgz
```

### 解压
```
tar -xf kafka_2.12-2.6.0.tgz
```

### 配置
kafka依赖于zookeeper，本身就自带一个zookeeper，所以不改配置也可以。
只不过一般还是都会使用外部的zookeeper，而搭建hdfs-ha模式时，我也刚好搭建了zookeeper集群，所以kafka自然也就使用这个zookeeper集群了，需要修改安装目录下的配置文件，例如`/opt/kafka/kafka_2.12-2.6.0/config`下的`server.properties`，把`zookeeper.connect`改成如下配置：
```
zookeeper.connect=node002:2181,node003:2181,node004:2181
```

### 环境变量
和上边一样，为了方便操作，也配置一下环境变量，加入kafka之后的`/etc/profile`相关内容如下：
```
export JAVA_HOME=/opt/java/jdk1.8.0_261
export REDIS_HOME=/opt/redis/redis6
export MONGODB_HOME=/opt/mongodb/mongodb4.4
export KAFKA_HOME=/opt/kafka/kafka_2.12-2.6.0
export PATH=$PATH:$JAVA_HOME/bin:$REDIS_HOME/bin:$MONGODB_HOME/bin:$KAFKA_HOME/bin
```

### 启动
配置好以后，就可以使用配置文件启动kafka:
```
kafka-server-start.sh /opt/kafka/kafka_2.12-2.6.0/config/server.properties
```

### 验证
这里验证包括三步，分别是创建topic、生产消息和消费消息，首先创建一个名为test的topic：
```
kafka-topics.sh --create --zookeeper node002:2181,node003:2181,node004:2181  --replication-factor 1 --partitions 1 --topic test
```
执行结果输出如下：
```
Created topic test
```

topic创建好了之后，可以直接使用上边的topic生产和消费消息，也可以先看一下topic列表：
```
kafka-topics.sh  -list --zookeeper node002:2181,node003:2181,node004:2181
```

为了验证创建的topic确实可用，则命令行连接kafka向指定topic发送消息：
```
kafka-console-producer.sh --broker-list node001:9092 --topic test
```

然后可以退出上述操作界面，或者直接新开一个界面再连接kafka消费指定topic的消息：
```
kafka-console-consumer.sh --bootstrap-server node000:9092 --topic test --from-beginning
```