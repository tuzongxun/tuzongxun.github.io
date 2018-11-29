---
title:  Mongodb3.0.5副本集搭建及spring和java连接副本集配置
date: 2016-11-07 15:09:44
tags: [数据库技术,mongodb]
toc: true
categories: [数据库技术,mongodb]
---
这是去年写的一篇文档，最近突然发现并没有发不出来，因此现在补上，希望能对某些朋友有所帮助。因为当时记录时没有截图，因此这里看起来可能就比较单调。
<!--more-->
### 基本环境：
>mongdb3.0.5数据库
>spring-data-mongodb-1.7.2.jar
>mongo-java-driver-3.0.2.jar
>linux-redhat6.3
>tomcat7
 
### 搭建mongodb副本集：
#### 安装
分别在三台linux系统机上安装mongodb,(为避免和机器上原有的mongodb端口冲突，这里设为57017)：
>192.168.0.160
>192.168.0.211（192.168.0.33上的虚拟机）
>192.168.0.213（192.168.0.4上的虚拟机）

每个mongodb的安装这里就不细说了，可以参考我的安装方面的文档，注意先不要更改用户验证方式。另外，这里如果没有三台机，也可以只用一台机开三个端口，同时准备三个数据存储目录。

#### 启动
以副本集的方式启动三个mongodb：
只是在单机mongodb启动的基础上加入副本集参数—replSet，例如启动160的：
<pre>
/home/admin/mongodb3051/mongodb305/bin/mongod –f  /home/admin/mongo3051/conf/mongodb.conf  --replSet reptest  
</pre>

其中，reptest是指定的副本集名称，另外两台机也也要和这个一样。如：
<pre>
/mongodb3051/mongodb305/bin/mongod –f /mongodb3051/conf/mongodb.conf  --replSet repTest  
</pre>

#### 配置
在任意一台机上配置副本集，这里在160上配置：
##### 进入160上的mongo sehll(数据操作界面)：
<pre>
/home/admin/mongodb3051/mongodb305/bin/mongo –port 57017  
</pre>
##### 切换到admin数据库：
<pre>
use admin  
</pre>
##### 配置副本集：
<pre>
config={_id:”reptest”,members:[{_id:0,host:”192.168.0.160:57017”},{_id:1,host:”192.168.0.211:57017”},{_id:,host:”192.168.0.213:57017”}]}  
</pre>
##### 加载副本集配置文件：
<pre>
rs.initiate(config)  
</pre>
##### 查看副本集状态：
<pre>
rs.status()  
</pre>
   正常情况下可以看到160会是主服务器，显示PRIMARY,如果是，就直接进行以下操作，如果不是，就切换到PRIMARY上进行以下操作（换到另一个mongo）；

##### 增加用户：
<pre>
db.createUser({“user”:”admin”,”pwd”:”admin”,”roles”:[“root”]})  
</pre>
##### 更改用户验证方式：  
<pre>
varschema=db.system.version.findOne({“_id”:”authSchema”})  
schema.currentVersion=3  
db.system.version.save(schema)  
</pre>
##### 删除用户：
<pre>
db.dropUser(“admin”)  
</pre>
##### 重新建立用户(系统中和上边建立的用户验证方式不一样)：
<pre>
db.createUser({“user”:”admin”,”pwd”:”admin”,”roles”:[“root”]})  
</pre>
##### 关闭三个mongodb：
<pre>
db.shutDownServer()或者kill命令  
</pre>
##### 在160的数据库的data目录中建立keyFile文件：     
<pre>
cd /home/admin/mongodb3051/data  
openssl  rand  –base64 753  >  keyFile  
</pre>
##### 给keyFile文件设置600权限（必须设置600权限）：
<pre>
chmod 600 keyFile  
</pre>
##### 把这个keyFile文件上传到另外两台机上mongodb的data目录中：     
<pre>
scp –r keyFile root@192.168.0.211/mongodb3051/data  
scp –r keyFile root@192.168.0.213/mongodb3051/data  
</pre>
##### 在mongodb.conf文件中加入keyFile,例如160：
<pre>
keyFile=/home/admin/mongodb3051/data/keyFile  
</pre>
##### 重新启动mongodb，使用replSet和auth参数：
<pre>
/home/admin/mongodb3051/mongodb305/bin/mongod –f /home/admin/mongo3051/conf/mongodb.conf --replSet reptest  --auth  
</pre>
##### 在priority中设置副本集成员的优先级，给160设置最高优先级，优先级默认都是1：   
<pre>
config=rs.conf()  
config.members[0].priority=2  
rs.reconfig(config)  
</pre>
 这样的话，只要160的mongodb是开着的，那么主服务器就会是160
 
### Spring中连接副本集的配置：
<pre>
&lt;?xml version="1.0"encoding="UTF-8"?>  
&lt;beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:mongo="http://www.springframework.org/schema/data/mongo"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
            http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
            http://www.springframework.org/schema/data/mongo  
         http://www.springframework.org/schema/data/mongo/spring-mongo.xsd">  
    &lt;!-- Factory bean that creates the Mongoinstance -->  
    &lt;mongo:mongo-client replica-set="192.168.0.160:57017" credentials="admin:admin@admin" id="mongo">   
       &lt;mongo:client-options write-concern="SAFE" connections-per-host="100"  
            threads-allowed-to-block-for-connection-multiplier="50"  />   
    &lt;/mongo:mongo-client>   
    &lt;mongo:db-factory  id="mongoDbFactory"dbname="admin" mongo-ref="mongo"/>   
    &lt;bean id="mongoTemplate"class="org.springframework.data.mongodb.core.MongoTemplate">   
       &lt;constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />   
    &lt;/bean>    
&lt;/beans>  
</pre>
只需要配置一个ip，就会自动切换。用户验证格式：username：password@dbname。
 
### java中连接副本集的代码：
<pre>
public DB getMongoDB() {  
      try {  
        ServerAddress sa = new ServerAddress("192.168.0.160", 57017);  
        ServerAddress sa1 = new ServerAddress("192.168.0.211", 57017);  
        ServerAddress sa2 = new ServerAddress("192.168.0.213", 57017);  
        List<ServerAddress> sends = new ArrayList<ServerAddress>();  
        sends.add(sa);  
        sends.add(sa1);  
        sends.add(sa2);  
        List<MongoCredential> mongoCredentialList = new ArrayList<MongoCredential>();  
        mongoCredentialList.add(MongoCredential.createMongoCRCredential("admin", "admin","admin".toCharArray()));  
        DB mongoDB = new MongoClient(sends,mongoCredentialList).getDB("admin");  
      } catch (Exception e) {  
        throw new RuntimeException("连接MongoDB数据库错误", e);  
      }  
    return mongoDB;  
  }  
</pre>
用户验证格式是：username，dbname，password