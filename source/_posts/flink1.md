---
title: flink初识及集群搭建和简单验证
date: 2020-10-13 22:59:42
tags: flink
toc: true
categories: flink
---
## 批计算和批计算

在软件系统中，尤其是企业级软件，基本离不开数据统计和分析等数据计算。最初，多数常见的统计分析都是基于数据库的数据进行处理，例如某一段时间的活跃用户数统计，这种计算方式称作离线计算，也称作批量计算（个人理解）。

而现实世界中的数据产生方式有很多都是持续不断的，也就是说实际很多场景的数据是就是数据流，这些数据随着时间的流逝，价值会不断的降低，因此就需要尽可能实时的进行处理。

而批计算是一批数据一起处理，尤其是最初数据先入数据库，再拿出来处理，这种方式在数据量日渐爆发的场景下，对于实时分析的业务就会有很多瓶颈，于是渐渐的出现了流计算。

相对于传统的批计算而言，流计算更加的实时，基本是在数据产生并接收到的同时就进行处理，更加符合当前很多要求实时计算的场景。

<!--more-->

## flink发展

流计算，或者说实时计算，最初代表性的技术是storm，除此之外还有spark。目前对这两个技术还没有深入了解，据说storm是真正的实时计算，而spark所谓的实时计算，实际只是缩小了批计算的范围，严格来说依然还是批计算。

因此实际上spark的实时计算没有storm快，但是spark支持实时计算的同时还支持批计算以及机器学习，并且也有它丰富的生态圈，因此spark应用场景也很广。

spark是2013年贡献给apache基金会，在2014年左右正式流行起来，而flink实际上也是差不多的时间贡献给apache基金会，但是当时却没有spark流行。由于乍一听起来和spark功能很重合，都是同时支持批计算、流计算和机器学习，所以之前甚至有人说它生不逢时。

直到2015年之后，阿里巴巴开始注意到这个框架并大量使用和改进，在经过了若干次双11的洗礼之后，这个框架的能力越来越被大众接收，使用的公司也越来越多，于是flink似乎翻身了。

上边说spark的实时计算实际上只是缩小了批计算的范围，而flink的实时计算则是真正的实时计算，所以flink实时计算的性能也要强于spark。

在flink的思想中，数据处理都是基于数据流，实时计算的数据流称作无界流，批计算的数据流称作有界流。

## 无界流

所谓的无界流，就是一段数据有开始时间，没有结束时间，其实就是数据持续在产生，需要持续的分析处理。

## 有界流

同理，有界流就是一段数据有开始时间，也有结束时间，所以其实也很容易发现有界流其实就是无界流的一个特例，在无界流中定义一个结束时间的话，这一段数据就是有界流。



flink的概念还有很多，例如jobManager、taskManager、solt等，对于flink集群来说，还有master和worker，这些概念均关联很多其他技术点，后续再进一步深入。

抱着先知其然再知其所以然的心态，这里先搭建一个简单的flink集群用起来。

## flink简单集群搭建（centos）

flink集群搭建相对简单，首先需要下载安装包，我下载的目前官网最新版`1.11.2`，可在官网[https://flink.apache.org/downloads.html](https://flink.apache.org/downloads.html)处查看最新版本并下载，下载方式多种，我这里使用wget直接下载到虚拟机：

```
wget https://downloads.apache.org/flink/flink-1.11.2/flink-1.11.2-bin-scala_2.11.tgz
```

下载好了之后是解压：

```
tar -zxf flink-1.11.2-bin-scala_2.11.tgz
```

然后是简单的配置，分为两步，一个是配置jobManager，一个是配置taskManager，其他配置暂时默认。

jobManager只需要修改解压后目录的`conf`目录的`flink-conf.yaml`文件，找到`jobmanager.rpc.address`这一行，把后边的`localhost`改为实际的`jobManager`节点的主机名，我这里就是`node001`，因此配置修改如下：

```
jobmanager.rpc.address: node001
```

然后同样是`conf`目录下，修改`workers`文件，早期版本的可能不叫`workers`，而是`slaves`，这个和hbase新旧版本中文件命名有点像。在`workers`文件中加入规划的taskManager节点主机名，例如我修改后如下：

```
node002
node003
node004
```

上述配置需要各个节点保持一致，所以需要把修改好的文件包括整个flink分发到其他机器上，也就是把解压后的这个flink的目录传到另外几个节点上，例如我是在node001上操作的，然后就可以使用类似如下的命令分发到其他机器：

```
scp -r flink-1.11.2 node002:`pwd`/
```

分发完成之后，各个机器需要配置一下环境变量，修改/etc/profile文件，加入flink内容，然后就可以在规划的jobManager节点执行启动命令启动flink集群：

```
start-cluster.sh
```

上述命令实际是flink解压后目录下的bin目录下的脚本，执行上述脚本后会看到日志依次打印出各个节点的启动情况：

```
Starting cluster.
Starting standalonesession daemon on host node001.
Starting taskexecutor daemon on host node002.
Starting taskexecutor daemon on host node003.
Starting taskexecutor daemon on host node004.
```

启动完成就要可以进行验证，最直接的验证就是访问自带的web界面，默认就是开启的，使用`8081`端口，例如我这里就可以使用`http://node001:8081`进行访问。

除此之外，为了更进一步的验证，参考官网示例，可以写一个简单的java代码验证。

## java程序验证flink

编写一个简单的flink程序，需要引入flink相应的依赖包，如下：

```
<dependency>
	<groupId>org.apache.flink</groupId>
	<artifactId>flink-scala_2.11</artifactId>
	<version>1.11.2</version>
</dependency>
<dependency>
	<groupId>org.apache.flink</groupId>
	<artifactId>flink-streaming-scala_2.11</artifactId>
	<version>1.11.2</version>
</dependency>
<dependency>
	<groupId>org.apache.flink</groupId>
	<artifactId>flink-clients_2.11</artifactId>
	<version>1.11.2</version>
</dependency>
```

然后根据官网[https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/datastream_api.html](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/datastream_api.html)的示例编写如下代码：

```
package com.tzx.study.demo.flink;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

public class FlinkTest {
    public static void main(String [] args)throws Exception{
        StreamExecutionEnvironment env= StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<Tuple2<String, Integer>> dataStream = env
            .socketTextStream("localhost", 8888)
            .flatMap(new Splitter())
            .keyBy(value -> value.f0)
            .sum(1);
        dataStream.print();
        env.execute("Window WordCount");
    }

    public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
            for (String word: sentence.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }
}

```

上述代码的意思是，创建一个执行环境，如果是idea等开发工具运行，就创建本地运行环境，如果是把程序生成可执行jar放到flink集群运行，就是集群环境。

然后建立一个本地的端口是8888的socket文件数据流连接，读到每行数据以空格分隔，然后计算数量。

上述代码在`main`方法中，因此是可以直接运行的，需要注意的是，运行之前需要先开启`8888`的`socket`端口监听，否则会启动失败，如果是本地idea测试，需要windows上启动这个端口，我是直接执行`nc -lp 8888`命令，后来把这个程序生成jar放到集群环境中运行，就需要在运行的linux节点中监听这个端口，例如`nc -lk 8888`，windows和linux中稍有区别。
当然了，如果要更方便的验证，也完全可以直接把`localhost`换成实际的主机名，这样就不需要分别在不同环境启动这个端口。

运行之后，在nc监听的界面输入相应信息，便可以看到实时输出的统计数据，代表简单的flink集群和程序验证成功，也标志着第一步成功迈出，接下来就是基于此的进一步应用和理解。