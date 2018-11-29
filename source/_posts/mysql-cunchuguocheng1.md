---
title: mysql常用操作语法（十四）~~复杂的存储过程
date: 2018-04-18 15:32:46
tags: [数据库技术,mysql]
toc: true
categories: [数据库技术,mysql]
---
## 前言
我看到在很多教程中，都是把存储过程和自定义函数一起讲，主要是因为他们两个非常的相像，而且自定义函数从某种程度上讲，更像是存储过程中的特例。
在这种情况下，我就暂时省略掉自定义函数的笔记，直接继续了解更复杂的存储过程。
所谓的更复杂，实际上也就是定义变量，变量赋值，游标以及结构控制语句和循环等，有java语言基础的情况下，就很好理解了，只是其中有些细节需要稍微注意一下。

<!--more-->

## 定义变量和赋值
之前的例子中，在存储过程里只是简单地做一些基本sql操作，而实际应用中自然远不止这么简单，经常需要声明变量，并进行一定的操作。
声明变量的语法是：

<pre>
DECLARE 变量名 变量类型  约束条件
</pre>

例如：

<pre>
DECLARE count INTEGER DEFAULT 100;
</pre>

给变量赋值的常用语法是：

<pre>
SET 变量名 = 值;
</pre>

例如：

<pre>
SET count = 10;
</pre>

上边赋值的语法比较常用，但是还有其他的语法也可以赋值，例如：

<pre>
SELECT COUNT(*) INTO COUNT FROM USER;
</pre>

上边这种根据sql语句的结果进行赋值需要注意的是，当前sql返回值必须是单一的而不能是多个。

## 结构控制
结构控制我目前了解的有两种，使用if和case，结合上边变量的知识以及之前存储过程一起举例进行说明：

<pre>
DELIMITER $
CREATE PROCEDURE proce_test0()
BEGIN
DECLARE COUNT INTEGER DEFAULT 0;
DECLARE result VARCHAR(10) DEFAULT '';
SELECT COUNT(*) INTO COUNT FROM USER;
IF COUNT < 2 THEN
SET result='不合格';
ELSEIF COUNT = 2 THEN
SET result ='及格';
ELSE
SET result = '优秀';
END IF;
SELECT result;
END
$
DELIMITER ;
</pre>

对上边sql的解释如下：
使用$分隔符确定sql语句的起始位置；
创建一个名称是proce_test0的无参存储过程；
在存储过程里声明一个COUNT变量，类型是INTEGER，默认值0；
再声明一个result变量，类型是varchar，长度是10，默认值是空字符串；
查询user表的数据条数，并把结果赋值给变量count；
根据count变量的值，进行if判断：
如果count小于2，给result赋值为“不合格”；
如果coutn等于2，给result赋值为“及格”；
否则，给result赋值为“优秀”；
结束if判断；
输出变量result的值；
结束存储过程；
分隔符定义结尾；

那么，上边的逻辑完全可以用case代替if实现，改过之后的sql应该如下：

<pre>
DROP PROCEDURE IF EXISTS proce_test0;
DELIMITER $
CREATE PROCEDURE proce_test0()
BEGIN
DECLARE COUNT INTEGER DEFAULT 0;
DECLARE result VARCHAR(10) DEFAULT '';
SELECT COUNT(*) INTO COUNT FROM USER;
CASE COUNT 
WHEN COUNT < 2 THEN
SET result='不合格';
WHEN 2 THEN
SET result ='及格';
ELSE
SET result = '优秀';
END CASE;
SELECT result;
END
$
DELIMITER ;
</pre>

## 游标
在上边的例子中，我们声明了变量，并且把sql语句的结果赋值给了变量，但是这里的问题就是只能把单结果的sql赋值给变量。如果要把一个结果集赋值给某个变量进行操作，就需要借助游标实现，例如：

<pre>
DELIMITER $
CREATE PROCEDURE proce_test0()
BEGIN
DECLARE user_id INTEGER;
DECLARE cursor_userId CURSOR FOR SELECT id FROM USER;
OPEN cursor_userId;
FETCH cursor_userId INTO user_id;
SELECT user_id;
CLOSE cursor_userId;
END
$
DELIMITER ;
</pre>

对上述示例的红色标记部分解释如下：
声明一个user_id变量，类型是INTEGER；
创建一个游标，名称是cursor_userId，游标里的内容是user表中的id结果集；
打开游标；
从游标当前位置取出一条数据，赋值给变量user_id，并把游标向前移动一位；
输出变量user_id的值；
关闭游标。

## 循环
上述示例中，只是从游标中取出了一条数据，如果要取出所有数据，比较好的方法就是使用循环遍历，有while、loop以及repeat,例如：

<pre>
DELIMITER $
CREATE PROCEDURE proce_test0()
BEGIN
DECLARE user_id INTEGER;
DECLARE flag INTEGER DEFAULT 0;
DECLARE idStr NVARCHAR(20) DEFAULT '';
DECLARE cursor_userId CURSOR FOR SELECT id FROM USER;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET flag = 1;
OPEN cursor_userId;
WHILE flag != 1 DO
FETCH cursor_userId INTO user_id;
IF user_id = 5 THEN
IF idStr = '' THEN
SET idStr = user_id;
ELSE
SET idStr = CONCAT(idStr,'|',user_id);
END IF;
END IF;
END WHILE;
CLOSE cursor_userId;
SELECT idStr;
END
$
DELIMITER ;
</pre>

上述存储过程中，我想做的事是这样的：
把user表中所有id是5的数据用“|”连接成一个字符串。
对上述红色部分的解释是：
声明一个名称为user_id的变量，类型是INTEGER；
声明一个名称为flag的变量，类型是INTEGER，默认值是0。这个变量的作用是作为while循环的条件；
声明一个名称是idStr的变量，类型是varchar，长度20，默认值空字符串；
声明一个游标，上边解释过；
设置游标读取到末尾的时候，改变变量flag的值；
打开游标；
使用while循环，添加条件flag ！= 1；
从游标读取数据，并移动游标位置，上边解释过；
外层if判断user_id的值，等于5的时候进入内层if；
内层if判断idStr是否是空字符串，如果是，就把user_id的值赋值给idStr，也就是第一个id前边不要“|”，否则进行字符串拼接；
结束内层if；
结束外层if；
结束while循环；
关闭游标；
输出变量idStr的值。

上边的存储过程把while循环替换成loop循环之后如下：

<pre>
DELIMITER $
CREATE PROCEDURE proce_test0()
BEGIN
DECLARE user_id INTEGER;
DECLARE flag INTEGER DEFAULT 0;
DECLARE idStr NVARCHAR(20) DEFAULT '';
DECLARE cursor_userId CURSOR FOR SELECT id FROM USER;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET flag = 1;
OPEN cursor_userId;
loop_test:LOOP
IF flag =1 THEN
LEAVE loop_test;
END IF;
FETCH cursor_userId INTO user_id;
IF user_id = 5 THEN
IF idStr = '' THEN
SET idStr = user_id;
ELSE
SET idStr = CONCAT(idStr,'|',user_id);
END IF;
END IF;
#FETCH cursor_userId INTO user_id;
END LOOP;
CLOSE cursor_userId;
SELECT idStr;
END
$
DELIMITER ;
</pre>

重点在红色部分，其中loop_test是自定义的名字，可以理解成java中for循环前的标签。然后需要if判断一个临界条件来终止循环，终止的语法就是`leave 循环名`，相当于java的for循环里的break。最后的end loop结束循环。

同样的，上边的写法也可以用repeat循环来代替：

<pre>
DELIMITER $
CREATE PROCEDURE proce_test0()
BEGIN
DECLARE idStr VARCHAR(20) DEFAULT '';
DECLARE user_id INTEGER;
DECLARE flag INTEGER DEFAULT 0;
DECLARE cursor_id CURSOR FOR SELECT id FROM USER;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET flag = 1;
OPEN cursor_id;
REPEAT
FETCH cursor_id INTO user_id;
IF user_id =5 THEN
IF idStr = '' THEN
SET idStr = user_id;
ELSE
SET idStr = CONCAT(idStr,"|",user_id);
END IF;
END IF;
UNTIL flag =1 END REPEAT;
CLOSE cursor_id;
SELECT idStr;
END
$
DELIMITER ;
</pre>

repeat循环的终止，需要借助于until关键字，这种语法结构的写法其实很像是java中的do while循环，先给一个关键词，然后是循环体，最后写终止循环的条件。

通过上边的示例，就可以基本了解变量声明和赋值，结构控制语句，游标，循环等语法的用法，从而组合起来生成相对复杂的存储过程了。