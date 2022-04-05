---
title: 若依管理系统RuoYi-Cloud版搭建记录
date: 2020-10-30 15:43:58
categories: 问题整理 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: 问题整理
---
1、redis安装

Linux中redis安装及软件安装相关Linux知识要点https://tuzongxun.blog.csdn.net/article/details/107170447



2、nginx安装

wget https://nginx.org/download/nginx-1.9.9.tar.gz

## 解压
tar -zxvf nginx-1.9.9.tar.gz

./configure --prefix=/usr/local/nginx


src/core/ngx_murmurhash.c:37:11: error: this statement may fall through [-Werror=implicit-fallthrough=]
h ^= data[2] << 16;
是编译器将警告当成了错误处理
打开 nginx的安装目录/objs/Makefile，去掉CFLAGS中的-Werror，再重新make



-Wall 表示打开gcc的所有警告
-Werror，它要求gcc将所有的警告当成错误进行处理


make
make install



3、natapp



5、内存问题