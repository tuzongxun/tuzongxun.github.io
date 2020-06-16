---
title: mysql常用操作语法（十五）~~触发器
date: 2018-04-25 23:42:11
tags: [数据库,mysql]
toc: true
categories: 数据库
---
# 理解
mysql触发器的概念，从某种程度来说，比较像java中的aop。也就是根据一定的规则，拦截某一类情况，然后在适当的时机（before/after）执行一些其他的逻辑。 
个人觉得，这是个比较好理解的概念和场景。同时，在存储过程之后再来看这个功能，那么它的使用也同样很简单。
<!--more-->

# 创建触发器
触发器大概的语法如下所示：
<pre>
CREATE TRIGGER 自定义名称
触发时机  触发事件 ON 触发事件所在的表名
FOR EACH ROW
触发需要执行的逻辑;
</pre>

那么在理解语法之前，先举个例子，假如有这样一个场景，我有table1和table2两张表，需要每当table2中插入一条数据之后，修改table1中的count字段，在原来count的基础上加1，那么这个触发器就应该是这样：
<pre>
CREATE TRIGGER trigger_t1
AFTER INSERT ON table2
FOR EACH ROW
UPDATE table1 SET table1.count=table1.count+1;
</pre>

上边示例中每一行的解释是： 
>创建一个触发器，自定义名称是trigger_t1; 
触发器生效的场景是，向table2表insert数据之后触发； 
触发器对每一行生效，这一行基本是固定写法； 
触发器触发后做的事，是修改table1的count字段，使其在原有的基础上加一。

这里需要理解的就是，AFTER不是固定的，但是却只能从特定的一些关键词中选择，比如AFTER代表之后，BEFORE代表之前；而INSERT代表了数据操作的动作，同样的可以是其他操作，例如UPDATE;然后就是具体执行的业务逻辑，可以根据需要自定义。

# 进阶
上边的触发器应该算是最简单的触发器，但是和存储过程一样，触发器的执行逻辑中可能并不都只存在一个简单的逻辑。 
当需要在触发器里进行稍复杂的逻辑处理时，就需要和写存储过程一样，结合分隔符以及begin、end等关键词一起实现。 
就如上边的例子中其实有一个小小的问题，那就是当table1没有数据的时候，那么无论table2中insert多少数据，table1都不会有变化。所以上边的写法可以变成下边这样：
<pre>
DELIMITER $
CREATE TRIGGER trigger_t0
AFTER INSERT ON table2
FOR EACH ROW
BEGIN
DECLARE cn INTEGER;
SELECT COUNT(id) INTO cn FROM table1;
IF cn != 0 THEN
UPDATE table1 SET table1.count = table1.count + 1 ;
ELSE
INSERT INTO table1 VALUES(1,1);
END IF;
END
$
DELIMITER ;
</pre>

上边内容解释如下： 
>使用符号分割，标示位于符号分割，标示位于中间的内容为一个整体，而不是单单用分号分割； 
创建一个名称是trigger_t0的触发器； 
触发器触发事件是，向table2插入数据只好触发； 
对每一行都生效； 
开始具体逻辑； 
声明一个变量cn，类型是INTEGER; 
查询table1的数据数量，并赋值给变量cn； 
判断变量cn的值是否不等于0； 
符合上边的条件，则更新数据库中count的值，在原基础加一； 
否则； 
向table1中插入一条数据； 
具体逻辑结束 
分隔符结尾； 
分割符声明结尾。

# 触发器查询
触发器的名称是自定义的，但是不能和已有的触发器重名，所以有的时候可能需要查询当前已经存在哪些触发器，可以使用如下语法查询：
`SHOW TRIGGERS FROM 数据库名`
例如，如果要查询test库中已有的触发器，就可以这样写：
`SHOW TRIGGERS FROM test`
尤其注意，是库名，不是表名。 
上边的查询会查询出该库所有的触发器，如果只要查询某个触发器的详细情况，可以用类似下边的查询语句：
`SHOW CREATE TRIGGER trigger_t1`

# 删除
如果触发器失效了，为了减少资源消耗，可能需要进行删除。同时，在某些初始化的场景下，也会经常在创建该触发器之前先执行删除操作，所以删除的语法也是必要的，基本语法如下：
`DROP TRIGGER 触发器名称`
而一般用的比较多的，是在删除前先进行一个判断，判断是否存在，那么就成了下边这样：
`DROP TRIGGER IF EXISTS 触发器名称`