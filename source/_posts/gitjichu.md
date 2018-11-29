---
title: git的几个基本操作
date: 2017-10-09 15:23:50
tags: git
categories: 博客搭建 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
---
**要把本地的数据提交到github远程仓库，需要几个必要的步骤，以下是根据自己的操作简单记录。**

**前提条件是本地安装好了git，并且做好了相应配置**
<!--more-->
# 基础操作

1. 首先需要在本地建立一个目录，例如docBlog，后边文件都存在这里；

2. cmd命令行进入到docBlog目录，例如：`cd E:\docBlog`，执行`git init`命令，意思是当前目录初始化一个本地git仓库，然后会看到生成了一个.git目录；

3. 把需要上传到github远程仓库的数据存在docBlog下,或者说在当前目录下新建子目录和相应文件，例如readme.md；

4. 在docBlog下执行`git status`,查看文件状态，会看到那些文件进行了改动；这个改动是相对于本地仓库而言；

5. 执行`git add 相应的文件或目录`；例如`git add readme.md`；这个大概的意思是把指定的文件或目录加到提交暂存区，不是真正的提交；

6. 执行`git commit -m "注释内容"`，正式提交add的所有内容到本地git仓库；

7. 执行`git remote add origin 远程github仓库的url`，例如：`git remote add origin https://github.com/tuzongxun/docBlog.git`我的理解是和远程仓库建立一个初步连接，感觉像是办某些长期业务的第一次申请一样，例如办一张手机卡；

8. 执行`git push -u origin master`；这个命令会把本地仓库的内容正是提交到远程仓库中，-u的意思我理解成新办的手机号第一次交话费的激活动作，因为这个也是只需要一次；

9. 后续每次有了新的改动，都可以依次像下边这样执行：
`git status` 查看状态；
`git add` 提交到暂存区（如果是删了文件，就要用delete，可以git help查看）；
`git commit -m`  正式提交;
`git push origin master` 提交到远程库（-u只需第一次时使用）；

10. 更多git操作可参考[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

# 分支合并
1. 创建一个分支，例如创建dev分支，`git branch dev`；

2. 切换分支，例如切换到dev分支，`git checkout dev`;

3. 上述两步合并，即创建并切换，`git checkout -b dev`；

4. 合并分支，例如要把dev内容合并到master，先切换到master分支，然后执行`git merge dev`；

5. 删除分支，例如删除dev分支，`git branch -d dev`；

6. 查看已有分支，以及当前所在分支，`git branch`；

# 标签tag
1. 查看当前已有分支，`git tag`;

2. 在当前位置创建分支，例如创建v1.0标签，`git tag -a v1.0`;也可以创建的时候添加注释，`git tag -a v1.0 -m "1.0版本"`；

3. 在历史提交的某个版本上穿件标签，先要查出历史版本，`git log --pretty=oneline`,或者直接`git log --oneline`;
然后可以看到一串提交id，例如`abb56d0`,然后创建标签，`git tag -a v1.0 abb56d0`；

4. 标签版本提交，`git push origin v1.0`；

5. 查看某个标签版本，`git show v1.0`；

6. 删除本地标签，`git tag -d v1.0`；

7. 删除远程标签，`git push origin --delete v1.0`；

# 更新
1. 获取当前分支、tag远程最新代码，`git pull`。
