---
title: tzxblog博客系统-接口文档
date: 2020-2-2 14:09:42
tags: [业余项目,tzxblog]
toc: true
categories: 业余项目
---
# V1.0
# 接口安全说明
所有接口需要实现签名，前台向后台发起请求前需要在header中添加签名参数和签名，以减少后台服务器的被攻击可能性。
签名参数至少包含url、时间戳，有token的接口也需要对token签名，以参数名首字母排序后使用&符号拼接后，进行rsa256算法签名。

<!--more-->

***
# 环境说明

|环境名称|协议|IP/域名|端口|私钥|
|:------|:------|:------|:------|:------|
|开发环境|	HTTP|	127.0.0.1|	8080|-|	
|测试环境|-|-|-|-|		
|联调环境|-|-|-|-|		
|预生产环境|-|-|-|-|			
|生产环境|-|-|-|-|		

# 统一响应对象

|参数名称	|数据类型	|是否必须	|参数描述|
|:------|:------|:------|:------|
|code	|String	|Y	|响应码|
|msg	|String	|Y	|响应描述|
|requestId	|String	|Y	|请求id|
|data	|Object	|N	|响应结果|

# 接口定义
## 文章分类或归档列表查询
***

| 接口名称      | 文章分类或归档列表查询                 ||||
|-----------|-----------------------------|
| URL       | tzxblog/blog/category\-list ||||
| 协议        | HTTP                        ||||
| 请求类型      | GET                         ||||
| 请求参数      | 参数名称                        | 数据类型   | 是否必须                    | 参数描述      |
| Header参数  | timestamp                   | String | Y                       | 时间戳       |
|| sign      | String                      | Y      | 签名                      |
| URL参数     | queryType                   | String | Y                       | index:首页； user:用户； recom:推荐；hot:热门； file:归档   |
|| userId    | String                      | N      | 用户id                    |
| 响应参数      | 参数名称                        | 数据类型   | 是否必须                    | 参数描述      |
|| code      | String                      | Y      | 响应码                     |
|| msg       | String                      | Y      | 响应描述                    |
|| requestId | String                      | Y      | 请求id                    |
|| data      | List<CategoryInfo>          | N      | 分类对象列表，参见附录CategoryInfo |

## 文章列表分页查询
***

| 接口名称      | 文章列表分页查询                ||||
|-----------|-------------------------|
| URL       | tzxblog/blog/blog\-list ||||
| 协议        | HTTP                    ||||
| 请求类型      | GET                     ||||
| 请求参数      | 参数名称                    | 数据类型                         | 是否必须    | 参数描述      |
| Header参数  | timestamp               | String                       | Y       | 时间戳       |
|| sign      | String                  | Y                            | 签名      |
| URL参数     | queryType               | String                       | Y       | index:首页；user:用户；recom:推荐；hot:热门；file:归档   |
|| pageIndex | Long                    | N                            | 页码，默认1  |
|| pagSize   | Long                    | N                            | 页数，默认10 |
|| userId    | String                  | N                            | 用户id    |
| 响应参数      | 参数名称                    | 数据类型                         | 是否必须    | 参数描述      |
|| code      | String                  | Y                            | 响应码     |
|| msg       | String                  | Y                            | 响应描述    |
|| requestId | String                  | Y                            | 请求id    |
|| data      | PageInfo<BlogInfo>      | N                       | 分页博客对象，参见附录PageInfo<BlogInfo> |

## 文章详情查询
***

| 接口名称      | 文章详情查询                ||||
|-----------|-----------------------|
| URL       | tzxblog/blog/bloginfo ||||
| 协议        | HTTP                  ||||
| 请求类型      | GET                   ||||
| 请求参数      | 参数名称                  | 数据类型   | 是否必须 | 参数描述 |
| Header参数  | timestamp             | String | Y    | 时间戳  |
|| sign      | String                | Y      | 签名   |
| URL参数     | blogId                | String | Y    | 博客id |
| 响应参数      | 参数名称                  | 数据类型   | 是否必须 | 参数描述 |
|| code      | String                | Y      | 响应码  |
|| msg       | String                | Y      | 响应描述 |
|| requestId | String                | Y      | 请求id |
|| data      | BlogInfo              | N      | 博客对象 |


## 文章发布
***

| 接口名称             | 文章发布                   ||||
|------------------|------------------------|
| URL              | tzxblog/blog/add\-blog ||||
| 协议               | HTTP                   ||||
| 请求类型             | POST                   ||||
| 请求参数             | 参数名称                   | 数据类型   | 是否必须   | 参数描述             |
| Header参数         | Content\-Type          | String | Y      | application/json |
|| timestamp        | String                 | Y      | 时间戳    |
|| sign             | String                 | Y      | 签名     |
|| token            | String                 | Y      | token  |
| Body参数           | blogTitle              | String | Y      | 博客标题             |
|| blogContent      | String                 | Y      | 博客内容   |
|| blogCategroy     | String                 | Y      | 网站博客分类 |
|| blogUserCategroy | String                 | Y      | 个人分类   |
|| blogType         | String                 | Y      | 博客类型： original：原创；reprint：转载；translate：翻译；    |
|| blogPower        | String                 | Y      | 博客权限：private:私密；public：公开        |
| 响应参数             | 参数名称                   | 数据类型   | 是否必须   | 参数描述             |
|| code             | String                 | y      | 响应码    |
|| msg              | String                 | y      | 响应描述   |
|| requestId        | String                 | y      | 请求id   |
|| data             | String                 | N      | 博客id   |


## 文章删除
***

| 接口名称      | 文章删除                      ||||
|-----------|---------------------------|
| URL       | tzxblog/blog/remove\-blog ||||
| 协议        | HTTP                      ||||
| 请求类型      | DELETE                    ||||
| 请求参数      | 参数名称                      | 数据类型   | 是否必须  | 参数描述 |
| Header参数  | timestamp                 | String | Y     | 时间戳  |
|| sign      | String                    | Y      | 签名    |
|| token     | String                    | Y      | token |
| URL参数     | blogId                    | String | Y     | 博客id |
| 响应参数      | 参数名称                      | 数据类型   | 是否必须  | 参数描述 |
|| code      | String                    | Y      | 响应码   |
|| msg       | String                    | Y      | 响应描述  |
|| requestId | String                    | Y      | 请求id  |
|| data      | String                    | N      | 博客id  |


## 文章修改
***

| 接口名称             | 文章修改                      ||||
|------------------|---------------------------|
| URL              | tzxblog/blog/update\-blog ||||
| 协议               | HTTP                      ||||
| 请求类型             | POST                      ||||
| 请求参数             | 参数名称                      | 数据类型   | 是否必须   | 参数描述             |
| Header参数         | Content\-Type             | String | Y      | application/json |
|| timestamp        | String                    | Y      | 时间戳    |
|| sign             | String                    | Y      | 签名     |
|| token            | String                    | Y      | token  |
| Body参数           | blogTitle                 | String | Y      | 博客标题             |
|| blogContent      | String                    | Y      | 博客内容   |
|| blogCategroy     | String                    | Y      | 网站博客分类 |
|| blogUserCategroy | String                    | Y      | 个人分类   |
|| blogType         | String                    | Y      | 博客类型：original：原创； reprint：转载；translate：翻译；    |
|| blogPower        | String                    | Y      | 博客权限： private:私密；public：公开        |
| 响应参数             | 参数名称                      | 数据类型   | 是否必须   | 参数描述             |
|| code             | String                    | y      | 响应码    |
|| msg              | String                    | y      | 响应描述   |
|| requestId        | String                    | y      | 请求id   |
|| data             | String                    | N      | 博客id   |


## 评论列表分页查询
***

| 接口名称      | 评论列表分页查询                      ||||
|-----------|-------------------------------|
| URL       | tzxblog/comment/comment-list ||||
| 协议        | HTTP                          ||||
| 请求类型      | GET                           ||||
| 请求参数      | 参数名称                          | 数据类型                            | 是否必须    | 参数描述   |
| Header参数  | timestamp                     | String                          | Y       | 时间戳    |
|| sign      | String                        | Y                               | 签名      |
| URL参数     | pageIndex                     | Long                            | N       | 页码，默认1 |
|| pagSize   | Long                          | N                               | 页数，默认10 |
|| userId    | String                        | N                               | 用户id    |
|| blogId    | String                        | N                               | 博客id    |
| 响应参数      | 参数名称                          | 数据类型                            | 是否必须    | 参数描述   |
|| code      | String                        | Y                               | 响应码     |
|| msg       | String                        | Y                               | 响应描述    |
|| requestId | String                        | Y                               | 请求id    |
|| data      | PageInfo<CommentInfo>          | N                             | 评论分页对象，参见附录PageInfo<CommentInfo> |

## 文章评论
***

| 接口名称           | 文章评论                         ||||
|----------------|------------------------------|
| URL            | tzxblog/comment/add\-comment ||||
| 协议             | HTTP                         ||||
| 请求类型           | POST                         ||||
| 请求参数           | 参数名称                         | 数据类型   | 是否必须  | 参数描述             |
| Header参数       | Content\-Type                | String | Y     | application/json |
|| timestamp      | String                       | Y      | 时间戳   |
|| sign           | String                       | Y      | 签名    |
|| token          | String                       | Y      | token |
| Body参数         | blogId                       | String | Y     | 博客id             |
|| commentContent | String                       | Y      | 评论内容  |
| 响应参数           | 参数名称                         | 数据类型   | 是否必须  | 参数描述             |
|| code           | String                       | Y      | 响应码   |
|| msg            | String                       | Y      | 响应描述  |
|| requestId      | String                       | Y      | 请求id  |
|| data           | String                       | N      | 博客id  |


## 文件列表分页查询
***

| 接口名称      | 文件列表分页查询                ||||
|-----------|-------------------------|
| URL       | tzxblog/file/file-list ||||
| 协议        | HTTP                    ||||
| 请求类型      | GET                     ||||
| 请求参数      | 参数名称                    | 数据类型                         | 是否必须    | 参数描述   |
| Header参数  | timestamp               | String                       | Y       | 时间戳    |
|| sign      | String                  | Y                            | 签名      |
| URL参数     | pageIndex               | Long                         | N       | 页码，默认1 |
|| pagSize   | Long                    | N                            | 页数，默认10 |
| 响应参数      | 参数名称                    | 数据类型                         | 是否必须    | 参数描述   |
|| code      | String                  | Y                            | 响应码     |
|| msg       | String                  | Y                            | 响应描述    |
|| requestId | String                  | Y                            | 请求id    |
|| data      | PageInfo<FileInfo       | N                       | 评论分页对象，参见附录PageInfo<FileInfo |

## 文件上传
***

| 接口名称      | 文件上传                      ||||
|-----------|---------------------------|
| URL       | tzxblog/file/upload\-file ||||
| 协议        | HTTP                      ||||
| 请求类型      | POST                      ||||
| 请求参数      | 参数名称                      | 数据类型   | 是否必须  | 参数描述                 |
| Header参数  | Content\-Type             | String | Y     | multipart/form\-data |
|| timestamp | String                    | Y      | 时间戳   |
|| sign      | String                    | Y      | 签名    |
|| token     | String                    | Y      | token |
| Form参数    | fileName                  | String | Y     | 文件名称                 |
|| file      | MultipartFile             | Y      | 文件    |
| 响应参数      | 参数名称                      | 数据类型   | 是否必须  | 参数描述                 |
|| code      | String                    | Y      | 响应码   |
|| msg       | String                    | Y      | 响应描述  |
|| requestId | String                    | Y      | 请求id  |
|| data      | String                    | N      | 文件id  |

## 文件下载
***

| 接口名称      | 文件下载                        ||||
|-----------|-----------------------------|
| URL       | tzxblog/file/download\-file ||||
| 协议        | HTTP                        ||||
| 请求类型      | POST                        ||||
| 请求参数      | 参数名称                        | 数据类型   | 是否必须  | 参数描述                 |
| Header参数  | Content\-Type               | String | Y     | multipart/form\-data |
|| timestamp | String                      | Y      | 时间戳   |
|| sign      | String                      | Y      | 签名    |
|| token     | String                      | Y      | token |
| URL参数     | fileId                      | String | Y     | 文件id                 |
| 响应参数      | 参数名称                        | 数据类型   | 是否必须  | 参数描述                 |
|-|-|-|-|-|

## 用户基本信息查询
***

| 接口名称      | 用户基本信息查询              ||||
|-----------|-----------------------|
| URL       | tzxblog/user/userinfo ||||
| 协议        | HTTP                  ||||
| 请求类型      | GET                   ||||
| 请求参数      | 参数名称                  | 数据类型   | 是否必须 | 参数描述 |
| Header参数  | timestamp             | String | Y    | 时间戳  |
|| sign      | String                | Y      | 签名   |
| URL参数     | userId                | String | Y    | 用户id |
| 响应参数      | 参数名称                  | 数据类型   | 是否必须 | 参数描述 |
|| code      | String                | Y      | 响应码  |
|| msg       | String                | Y      | 响应描述 |
|| requestId | String                | Y      | 请求id |
|| data      | UserInfo              | N      | 用户信息 |


## 用户注册
***

| 接口名称       | 用户注册                  ||||
|------------|-----------------------|
| URL        | tzxblog/user/register ||||
| 协议         | HTTP                  ||||
| 请求类型       | POST                  ||||
| 请求参数       | 参数名称                  | 数据类型   | 是否必须 | 参数描述             |
| Header参数   | Content\-Type         | String | Y    | application/json |
|| timestamp  | String                | Y      | 时间戳  |
|| sign       | String                | Y      | 签名   |
| Body参数     | userAccount           | String | Y    | 账号               |
|| password   | String                | Y      | 密码   |
|| rePassword | String                | Y      | 密码确认 |
|| userPhone  | String                | Y      | 手机号  |
|| userEmail  | String                | Y      | 邮箱   |
|| userName   | String                | N      | 姓名   |
|| userNick   | String                | N      | 昵称   |
|| userAge    | Integer               | N      | 年龄   |
|| userDesc   | String                | N      | 简介   |
|| userAddr   | String                | N      | 地址   |
| 响应参数       | 参数名称                  | 数据类型   | 是否必须 | 参数描述             |
|| code       | String                | Y      | 响应码  |
|| msg        | String                | Y      | 响应描述 |
|| requestId  | String                | Y      | 请求id |
|| data       | String                | N      | 用户id |

## 用户登录
***

| 接口名称      | 用户登录               ||||
|-----------|--------------------|
| URL       | tzxblog/user/login ||||
| 协议        | HTTP               ||||
| 请求类型      | POST               ||||
| 请求参数      | 参数名称               | 数据类型   | 是否必须 | 参数描述             |
| Header参数  | Content\-Type      | String | Y    | application/json |
|| timestamp | String             | Y      | 时间戳  |
|| sign      | String             | Y      | 签名   |
| Body参数    | userAccount        | String | Y    | 账号               |
|| password  | String             | Y      | 密码   |
| 响应参数      | 参数名称               | 数据类型   | 是否必须 | 参数描述             |
|| code      | String             | Y      | 响应码  |
|| msg       | String             | Y      | 响应描述 |
|| requestId | String             | Y      | 请求id |
|| data      | LoginInfo          | N      | 登录信息 |

***
# 附录
## 附录一
### CategoryInfo
| 属性名  | 数据类型   | 是否必须 | 属性描述 |
|------|--------|------|------|
| id   | String | Y    | 分类id |
| name | String | Y    | 分类名称 |

### PageInfo
| 属性名         | 数据类型   | 是否必须 | 属性描述 |
|-------------|--------|------|------|
| pageIndex   | Long   | Y    | 页码   |
| pageSize    | Long   | Y    | 每页数量 |
| totalCount  | Long   | Y    | 总数   |
| pageCount   | Long   | Y    | 页数   |
| pageContent | Object | N    | 分页内容 |

### BlogInfo
| 属性名              | 数据类型   | 是否必须 | 属性描述   |
|------------------|--------|------|--------|
| id               | String | Y    | 博客id   |
| blogTitle        | String | Y    | 博客标题   |
| blogAbstract     | String | Y    | 博客摘要   |
| readCount        | Long   | Y    | 阅读数    |
| commentCount     | Long   | Y    | 评论数    |
| updateTime       | String | Y    | 更新时间   |
| createTime       | String | Y    | 创建时间   |
| blogContent      | String | Y    | 博客内容   |
| blogCategroy     | String | Y    | 网站博客分类 |
| blogUserCategroy | String | Y    | 个人分类   |
| blogType         | String | Y    | 博客类型： original：原创；reprint：转载；translate：翻译；    |
| blogPower        | String | Y    | 博客权限：private:私密； public：公开        |
| userId           | String | Y    | 用户id   |
| userNick         | String | Y    | 用户昵称   |


### CommentInfo
| 属性名            | 数据类型   | 是否必须 | 属性描述 |
|----------------|--------|------|------|
| id             | String | Y    | 评论id |
| updateTime     | String | Y    | 更新时间 |
| createTime     | String | Y    | 创建时间 |
| commentContent | String | Y    | 评论内容 |

### FileInfo
| 属性名        | 数据类型   | 是否必须 | 属性描述 |
|------------|--------|------|------|
| id         | String | Y    | 文件id |
| updateTime | String | Y    | 更新时间 |
| createTime | String | Y    | 创建时间 |
| fileUrl    | String | Y    | 文件路径 |
| fileName   | String | Y    | 文件名称 |
| fileDesc   | String | Y    | 文件描述 |
| downCount  | Long   | Y    | 下载次数 |

### UserInfo
| 属性名         | 数据类型    | 是否必须 | 属性描述  |
|-------------|---------|------|-------|
| id          | String  | Y    | 文件id  |
| updateTime  | String  | Y    | 更新时间  |
| createTime  | String  | Y    | 创建时间  |
| userAccount | String  | Y    | 账号    |
| userName    | String  | N    | 姓名    |
| userNick    | String  | Y    | 昵称    |
| userAge     | Integer | N    | 年龄    |
| userDesc    | String  | N    | 简介    |
| userAddr    | String  | N    | 地址    |
| userPhone   | String  | Y    | 手机号   |
| userEmail   | String  | Y    | 邮箱    |
| userImg     | String  | Y    | 头像url |

### LoginInfo
| 属性名   | 数据类型      | 是否必须 | 属性描述      |
|-------|-----------|------|-----------|
| user  | UserInfo  | Y    | 用户对象      |
| token | UserToken | Y    | 用户token对象 |

### UserToken
| 属性名                  | 数据类型   | 是否必须 | 属性描述    |
|----------------------|--------|------|---------|
| access_token        | String | Y    | AT      |
| expires_in          | Long   | Y    | AT有效期   |
| refresh_token       | String | Y    | RT      |
| refresh_expires_in | Long   | Y    | RT有效期   |
| scope                | String | N    | 用户权限，备用 |

# 项目地址
项目代码和文档均以github托管，地址如下：
<https://github.com/tuzongxun/tzxblog>