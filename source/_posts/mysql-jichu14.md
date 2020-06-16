---
title: mysql常用操作语法（十三）~~存储过程
date: 2018-04-16 23:42:11
tags: [数据库,mysql]
toc: true
categories: 数据库
---
## 为什么要使用存储过程
在系统实际开发应用中，有可能只需要单条sql语句就能实现想要的功能，但是有时候，要完整实现某个业务，却需要同时执行多条sql才能达到目的。
在这种业务场景中，如果不希望通过客户端屡次执行多条sql，那么存储过程就是其中一个较好的实现方式。
因此，存储过程可以简单的理解为就是多条sql的集合，虽然在存储过程中实际也可以是单条sql。

<!--more-->
## 对存储过程的理解
除了上述的存储过程就是sql语句的集合外，作为一个java程序员，个人觉得也可以把存储过程理解成java中的方法，其实就是一个封装的过程，通过封装可以提升移植性和重用性。
只不过，在数据库的概念中，除了存储过程，还有自定义函数，这两个大同小异，都可以看做java的方法。

## 示例准备
为了对后边语法使用举例，这里先创建一个表：
<pre>
CREATE TABLE `user` (
   `id` int(11) DEFAULT NULL,
   `name` varchar(20) DEFAULT NULL
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8
</pre>

## 存储过程的创建
就像java方法有一定的语法规则一样，数据库存储过程的创建也需要遵循一定的语法规则，大致如下：
<pre>
CREATE PROCEDURE  自定义名称(参数列表)
  [characteristic]  存储过程body
</pre>
其中，参数列表的格式为：[INT/OUT/INOUT] 参数名 参数类型，这里的in代表输入，out代表输出。
characteristic是可选的定义，可以是注释，也可以是对存储过程body的约束，在具体使用的过程中可以再通过网络查询细节。
而存储过程body，就是具体的逻辑sql了。
另外还需要注意的是，自定义名称要尽量使用proce_**这种。
示例如下：
<pre>
CREATE PROCEDURE proce_test4(IN NAME VARCHAR(10))
INSERT INTO USER VALUES(4,NAME);
</pre>
上边的示例的意思是：创建一个名称为proce_test4的存储过程，这个存储过程需要一个varchar类型的输入参数，长度限制为10，参数名是name。这个存储过程要做的事，就是向user表插入一条数据，id是4，name就是调用这个存储过程的时候传入的参数

### 多条sql语句的存储过程
我们知道，在mysql中，“；”是代表结束的标示，所以在上边的示例中，“；”也就代表这个存储过程的创建结束了，那么如果这个存储过程中需要多条sql怎么办呢？
这个时候显然不能在“；”后边直接加，因为上一个“；”已经代表了结束。
也不能去掉之前的“；”号再加，因为这样的话数据库会误以为是一条sql，而出现语法错误，正确的做法是使用类似下边的写法：
<pre>
DELIMITER $$
CREATE PROCEDURE proce_test5(IN NAME VARCHAR(10))
BEGIN
INSERT INTO USER VALUES(5,NAME);
SELECT * FROM USER;
END$$
DELIMITER;
</pre>
大概意思就是，使用`$$`作为分解符，在两个`$$`之间的语句当做一个整体。
而上边这个示例所做的事也比较简单，就是：先插入一条数据到user表中，然后查询出user表中所有的数据。

## 存储过程的调用
要验证上述示例的正确性，就需要使用这个存储过程看一下效果，调用存储过程的基本语法如下：
<pre>
CALL 存储过程名(参数);
</pre>
例如：
<pre>
CALL proce_test5('ptest5');
</pre>

## 存储过程的查询
存储过程创建以后，存放在mysql的系统表中，有时候可能需要查看当前有哪些存储过程，或者某个存储过程具体信息，这就需要用到存储过程的查询：
1、查询某个数据库下有那些存储过程
<pre>
SELECT name FROM mysql.proc WHERE db='数据库名';
</pre>
2、查询某个数据库下所有存储过程的状态信息，例如创建时间、修改时间等
<pre>
SHOW PROCEDURE STATUS WHERE db='数据库名';
</pre>
3、查看某个存储过程详细信息，可以看到里边具体的sql，前提是需要知道存储过程的名字
<pre>
SHOW CREATE PROCEDURE 数据库.存储过程名;
</pre>

##  存储过程的删除
存储过程是一种手段，那么就有他的应用场景，因此也就存在失去作用的时候，这时候可能就需要删除，从而减少数据库资源的消耗，删除语法比较简单：
<pre>
DROP PROCEDURE 存储过程名称
</pre>

## 存储过程的修改
存储过程是可以修改的，但是据我目前所知，只能修改名称以及characteristic这些，而不能修改存储过程body，也就是里边具体的sql。
如果需要修改这个存储过程，网上建议的方法就是先删再建。
因此，对于存储过程的修改，暂时觉得可能应用场景不够多，就暂时放弃细究。