---
title: git的几个基本操作
date: 2017-10-09 15:23:50
tags: git
categories: git #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
---
**要把本地的数据提交到github远程仓库，需要几个必要的步骤，以下是根据自己的操作简单记录。**

**前提条件是本地安装好了git，并且做好了相应配置**
<!--more-->
# 初始化及基础操作
一、首先需要在本地建立一个目录（或者本地的项目），例如docBlog，后边文件都存在这里；
二、后续步骤：

## 命令行方式
1. cmd命令行进入到docBlog目录，例如：`cd E:\docBlog`，执行`git init`命令，意思是当前目录初始化一个本地git仓库，然后会看到生成了一个.git目录；

2. 把需要上传到github远程仓库的数据存在docBlog下,或者说在当前目录下新建子目录和相应文件，例如readme.md；

3. 在docBlog下执行`git status`,查看文件状态，会看到那些文件进行了改动；这个改动是相对于本地仓库而言；

4. 执行`git add 相应的文件或目录`；例如`git add readme.md`；这个大概的意思是把指定的文件或目录加到提交暂存区，不是真正的提交；

5. 执行`git commit -m "注释内容"`，正式提交add的所有内容到本地git仓库；

6. 执行`git remote add origin 远程github仓库的url`，例如：`git remote add origin https://github.com/tuzongxun/docBlog.git`我的理解是和远程仓库建立一个初步连接，感觉像是办某些长期业务的第一次申请一样，例如办一张手机卡；

7. 执行`git push -u origin master`；这个命令会把本地仓库的内容正是提交到远程仓库中，-u的意思我理解成新办的手机号第一次交话费的激活动作，因为这个也是只需要一次；

8. 后续每次有了新的改动，都可以依次像下边这样执行：
`git status` 查看状态；
`git add` 提交到暂存区（如果是删了文件，就要用delete，可以git help查看）；
`git commit -m`  正式提交;
`git push origin master` 提交到远程库（-u只需第一次时使用）；

## tortoisegit图形界面方式
1. 进入需要git管理的目录下，例如/docBlog,然后右键选择`git create repository here`，等同于`git init`，会在当前目录生成.git目录；

2. 继续在当前目录右键后选择`git commit`,默认是mater分支，填写注释、选择需要提交的文件后提交，然后文件会被提交到本地git仓库；

3. 创建远程git仓库，复制仓库地址，例如`https://github.com/tuzongxun/docBlog.git`；

4. 在需要git操作的目录或者文件上右键然后选择Tortoisegit，再选择`settings`，粘贴上边复制的远程仓库地址，如下图：
![gitremote](/images/git/gitremote.png)

5. 右键Tortoisegit，选择`git pull`，先同步一下远程库；

6. 右键Tortoisegit，选择`git push`，提交到远程仓库；

注：以上步骤按顺序操作，即最基本的初始化操作；

7. 通常的常规操作步骤只需要`git pull`、`git commit`、`git push`即可，不过更建议的方式是`git pull`、`git commit`、`git pull`、`git push`，这样可以一定程度上防止系统自动的merge；

**更多git操作可参考[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)**

***
# 分支和合并
## 基础操作
1. 创建一个分支，例如创建dev分支，`git branch dev`；

2. 切换分支，例如切换到dev分支，`git checkout dev`;

3. 上述两步合并，即创建并切换，`git checkout -b dev`；

4. 创建分支并切换之后可以直接先push推送到远端，推送时不会显示创建分支前的改动（用命令行push可能会把idea或eclipse特有的文件也推送上去）；

5. 合并分支，例如要把dev内容合并到master，先切换到master分支，然后执行`git merge dev`；

6. 删除分支，例如删除dev分支，`git branch -d dev`；

7. 查看已有分支，以及当前所在分支，`git branch`；

## Tortoisegit图形界面合并分支步骤
以下步骤针对有权限控制，不能直接push到目标分支的场景。
1. 选择`git pull`更新当前分支远程代码（例如需要把dev分支合并到master分支，这里就选择dev（源分支）），若有冲突，解决冲突；

2. 选择`git commit`提交本地没问题的代码到本地git仓库；

3. 选择`git push`推送本地分支代码到远程仓库；

4. 选择`git pull`或者`git sync`更新(同步)需要合并的目标分支的远程代码（例如需要把dev分支合并到master分支，这里就在dev分支选择`git pull`或`git sync`，然后在出现的界面的remote branch里选择master（目标分支））,若果有冲突，解决冲突；

5. 选择`git commit`提交本地合并后的源分支最新代码到本地git仓库；这一步只有在解决过冲突的情况下需要，如果pull之后本地没有冲突，实际可以直接飘到下一步。

6. 选择`git push`推送本地合并后的源分支最新代码到远程git仓库；

7. 进入github或者gitlab或gitee远程网页界面，进入需要合并的仓库，选择`branchs`；

8. 找到刚刚提交的源分支，选择`merge request`,进入合并请求界面：

9. 在合并请求界面中填写标题和描述，指定处理人（即该项目具有master权限的人，如果自己有master权限，也可以是自己），选择需要合并的源分支和目标分支，如下图：
![gitmerge](/images/git/merge.png)

10. 等待上边指定的具有master权限的处理人进行合并确认。

# 标签tag
**分支和标签最大的区别：tag的位置是固定的，在给指定提交打好标签以后，它就固定于此位置。branch的位置会不断变动的，随着分支的向前推移或者向后回滚，都在不断变化**
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