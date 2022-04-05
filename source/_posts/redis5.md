---
title: springboot集成和使用redis常用功能
date: 2020-8-4 11:59:42
tags: redis
toc: true
categories: redis
---
纯粹的redis命令行操作，可能就跟纯粹的mysql命令行操作一样，会有一种就是为了用而用的感觉。所以一般来说redis作为一个缓存中间件，都会辅以其他的客户端语言进行操作，比如java。
redis是开源的，java也是开源的，这就注定了java中连接redis的客户端不止一种，常见的有jedis、redisson、lettuce。
在如今一切都在高速运行的社会环境下，java软件开发也是极其追求代码产出效率，所以随处可见的都是springcloud、springboot。
springboot约定优于配置的思想，一定程度上也确实大大简化了开发工作，所以在学习java中的redis集成和使用时，我也一样选择了使用springboot。

<!--more-->
## springboot集成redis
springboot中集成redis，和springboot集成其他一些组件类似，基本都是引入一个starter即可：
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 数据源连接配置
有了依赖的引入后，接下来就是配置redis，springboot中如果是使用yml格式，则基础配置如下：
```
spring:
    redis:
        host: 192.168.139.9
        password:
        database: 0
        port: 6379
```

以上host指向Redis服务器的ip，这里是我自己虚拟机的ip，没有设置密码，所以密码没有。
redis默认有16个database，可以在配置文件中修改，我没有修改，还是使用默认的16个，而这里客户端连接使用0号database。同样的，端口号使用默认6379.

## restTemplate使用
springboot中对redis的大多数操作都进行了统一的封装，即RedisTemplate，对于之前说到的redis的五种基本数据类型以及一些常用高级功能的操作，都是通过RedisTemplate进行。
>redisTemplate.opsForValue()可以获取一个对string类型的操作；
redisTemplate.opsForList()可以获取一个对list类型的操作；
redisTemplate.opsForHash()可以获取一个对hash类型的操作；
redisTemplate.opsForSet()可以获取一个对set类型的操作；
redisTemplate.opsForZSet()可以获取一个对sorted_set类型的操作

基于上边的内容，就可以演示5种类型对数据的基本存储操作，例如：
```
redisTemplate.opsForValue().set(key,value);
redisTemplate.opsForList().leftPush(key,value);
redisTemplate.opsForHash().put(key,key,value);
redisTemplate.opsForSet().add(key,value);
redisTemplate.opsForZSet().add(key,value,0);
```

在基础数据类型中说道，string里边其实还要很多类型，比如bitmap，目前我使用的springboot-redis版本似乎还不完全支持bitmap的所有操作，例如保存bitmap有现成的封装，可以写成下边这样：
```
redisTemplate.opsForValue().setBit(key,0,true);
```
但是像bitmap的统计bitcount就没有直接操作的方法，需要自己写回调函数进行操作：
```
redisTemplate.execute((RedisCallback<Long>)connection -> connection.bitCount((key).getBytes()));
```

除了基础数据类型的简单操作，redisTemplte也可以进行过期设置，以及管道等高级功能的操作，例如;
```
redisTemplate.expire("k111",200, TimeUnit.SECONDS);
redisTemplate.expireAt("k111",new Date());

redisTemplate.executePipelined(new RedisCallback<Object>() {
    @Override
    public Object doInRedis(RedisConnection connection) throws DataAccessException {
        connection.openPipeline();
        connection.set("k111".getBytes(),"t111".getBytes());
        connection.set("k222".getBytes(),"t222".getBytes());
        return null;
    }
});
```
过期操作其实也很简单，和其他一些操作一样，都和命令行操作极其相似，基本上点一下api提示就可以看懂用法，管道操作就相对复杂一点，也是需要自己实现一定的回调函数进行处理，但是实际也比较简单。

## 连接池简单配置
实际使用redis的时候，都不会是上边那样仅仅配置ip和端口等就行了，而是还会加上其他的配置，例如连接池。
上边说java中连接redis常用的客户端有三种，springboot默认使用的就是lettuce，所以默认连接池的配置也是使用lettuce，加上连接池的几个简单配置后，redis配置如下：
```
spring:
    redis:
        host: 192.168.139.9
        password:
        database: 0
        port: 6379
        lettuce:
            shutdown-timeout: 100ms
            pool:
                maxIdle: 5
                maxActive: 20
```

这里需要注意的是，lettuce连接池虽然可以配置，但是实际上还缺少依赖，如果就这样启动，会发现启动会报错：
```
Caused by: java.lang.NoClassDefFoundError: org/apache/commons/pool2/impl/GenericObjectPoolConfig
	at org.springframework.data.redis.connection.lettuce.LettucePoolingClientConfiguration$LettucePoolingClientConfigurationBuilder.<init>(LettucePoolingClientConfiguration.java:94) ~[spring-data-redis-2.3.1.RELEASE.jar:2.3.1.RELEASE]
	at org.springframework.data.redis.connection.lettuce.LettucePoolingClientConfiguration.builder(LettucePoolingClientConfiguration.java:51) ~[spring-data-redis-2.3.1.RELEASE.jar:2.3.1.RELEASE]

```

上边错误提示其实很明显了，就是缺少commons-pool2相关的类，所以这里就需要再引入相关依赖;
```
<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-pool2</artifactId>
</dependency>
```

有了上边依赖之后，再次启动，便可以正常使用了。

## key显示不正常问题
上边的操作看起来没什么，但其实会有一个问题，如果取一下数据就会发现数据的key并不是我们存进去的内容，会有一部分像是乱码一样。
解决办法之一是使用springboot封装的高阶API，即StringRedisTemplate，另一个办法就是配置redis序列化，例如：
```
  @Autowired(required = false)
    public void setRedisTemplate(RedisTemplate redisTemplate) {
        RedisSerializer stringSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        redisTemplate.setHashKeySerializer(stringSerializer);
        redisTemplate.setHashValueSerializer(stringSerializer);
        this.redisTemplate = redisTemplate;
    }
```

**相关文章**

[缓存和redis相关基础知识](https://tuzongxun.blog.csdn.net/article/details/106878541)
[Linux中redis安装及软件安装相关Linux知识要点](https://tuzongxun.blog.csdn.net/article/details/107170447)
[redis数据类型要点知识及应用场景](https://tuzongxun.blog.csdn.net/article/details/107371302)
[redis管道、事务、发布订阅、过期、过滤器等常用高级功能小记（上篇）](https://tuzongxun.blog.csdn.net/article/details/107646610)
[redis管道、事务、发布订阅、过期、过滤器等常用高级功能小记（下篇)](https://tuzongxun.blog.csdn.net/article/details/107686577)
[springboot集成和使用redis常用功能](https://blog.csdn.net/tuzongxun/article/details/107794207)