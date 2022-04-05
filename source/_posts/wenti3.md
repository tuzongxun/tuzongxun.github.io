---
title: 近期生产问题和解决方案记录
date: 2020-10-23 15:43:58
categories: 问题整理 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 问题整理
---
前几天生产环境上线，过程不算顺利，总结下原因大概有这么几点：

1. 前期准备工作不足，依赖的中间件软件安装后未经过充分验证，例如mongodb；
2. 临时改配置，sit和预生产的中间件单机，生产使用集群，集群环境的使用未得到充分验证；
3. 不可抗因素，踩到了springboot版本的bug。

<!--more-->

以下是部分可记录供后续参考的问题和解决方案：

## mysql-sqlmode问题

这个问题实际不是生产的问题，是之前预生产出现的，之前没有记录，这里一并记了。

部署预生产后，数据库分页操作报错，这个问题遇到很多次了，只是运维同事搭建数据库时没有提前处理。

错误如下：

```
ORDER BY clause is not in GROUP BY clause and contains nonaggregated column 'information_schema.PROFILING.SEQ' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

解决方式不算复杂，修改sql_mode即可，可参考如下博客：

[https://www.cnblogs.com/kenshinobiy/p/9580701.html](https://www.cnblogs.com/kenshinobiy/p/9580701.html)

## redis value must not be null问题

生产环境部署时，使用redis的服务一直在重启，而日志却没有任何异常。

开始怀疑是健康检查有问题，使用`http://ip:port/actuator/health`发现确实结果是`down`，而不是`up`，但是由于日志没有异常，难以查看更多信息。

为了进一步查找问题，springboot增加如下配置，使得健康检查返回信息更多一些：

```
management:
  endpoint:
    health:
      show-details: always
```

有了上述配置之后，果然找到了问题，访问健康检查url后，发现是redis连接的健康经检查失败了。

诡异是，程序启动时成功的，对redis的操作都正常，只是启动成功过不了多久就持续重启。

再考虑到dev、sit、preprod环境都是正常的，问题点倒是缩小了，这三个环境都是单机的redis，而生产用了redis集群。

但是项目是支持集群模式的，代码和配置本身应该也没问题，否则也不可能正常操作redis。

最终，发现是`springboot2.3.1`版本有bug，这里边对redis集群的健康检查有问题，会导致redis集群模式下的健康检查失败。

[https://github.com/spring-projects/spring-boot/issues/21514](https://github.com/spring-projects/spring-boot/issues/21514)

找到了原因，解决方案就相对简单，而且不止一种选择，可以直接关闭健康检查，显然这不符合生产环境的要求。

也可以只关闭redis的健康检查，看起来好一点，但是如果redis真出了问题却也会导致健康检查检查不出来。

于是我们选择了第三种，即升级springboot的版本到解决了上述bug的版本，目前升级到了`2.3.4`。

第三种应该是目前较优方案，只看后续的全面回归测试是否有影响。

## gitlab的坑

这是一个说严重很严重，说不严重好像也不严重的问题，gitlab的web界面进行合并操作时，默认会勾选删除源分支选项。

在生产代码发布时，突然发现合并后原来的分支不见了，于是仔细检查了下之前的操作，便发现了上边的坑。

虽然重新`push`一下就没事了，但是删除本身就是个很敏感的操作，再次提醒做事需仔细，上线当小心。