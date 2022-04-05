---
title: jenkins集成sonar问题记录
date: 2022-3-28 21:43:58
categories: 运维 #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [运维,sonar]
---
## 前言
jenkins集成sonar，今天终于跑通了，回头去看似乎很简单，但是实际的过程依然是充满了曲折，尤其是有些细节问题，时间久了多半还是很容易忘记的，因此决定还是做个记录。

<!--more-->

## sonar-scanner安装问题
首先，jenkins集成sonar需要安装"SonarQube Scanner for Jenkins"这个插件，插件安装完了之后，需要进行工具的配置,这个配置路径是"系统管理-》全局工具配置-》SonarQube Scanner"。
![在这里插入图片描述](https://img-blog.csdnimg.cn/39ebb52d93954e1580cfc37b676a153d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

在这个配置中，需要制定一个sonar的客户端程序，可以直接在jenkins的页面上选择安装，但是我这样操作的时候始终下载不成功，尽管下载源修改了，依旧是没能成功。
于是就只好换了一个方法，去官网下载，即下边这个url：
https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/
上边url里提供了一些sonar-scanner-cli的下载，但是即便我下载了里边最老的版本，最终发现使用的也是jdk11。
暂时不确定是否能在这种情况下强行使用jdk8，所以我就又去其他网站找到了3.X的版本。

## 几个关键的配置
上边已经提到了一个配置，即sonarQube-scanner-cli的配置，也就是客户端。
除了客户端之外，还需要配置sonar服务端，配置路径是"系统管理-》系统配置-》SonarQube servers"。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0bb4a5941ac249b39278ffcfa3c76d2b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

在这个配置里，需要指定sonar服务端的url和请求服务端是的用户凭证，所以还需要一个配置就是token，配置路径是"系统管理-》manage credentials-》添加凭据"。
![在这里插入图片描述](https://img-blog.csdnimg.cn/76785a98b7ac4063b69d8849bb0acc6a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

这里需要注意的是凭据类型要选择secret text，secret就是sonar中的auth token，在sonar页面第一次登陆的时候会提示进行设置。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e2c00eacc6f451fac515cb88b6ae9e3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

实际上，这里还有一些细节是需要注意的，否则很可能就会和我一样导致后边的那些问题，这些细节待下边具体问题说到的时候再说。

## 基本参数
sonar扫描的时候是需要一个基础参数的，可以直接在jenkins中配置，也可以放在项目的配置文件中，我选择的是放在项目的配置文件中，即在项目根目录（pom.xml同级目录）创建sonar-project.properties，里边类容如下：
```
sonar.projectKey=my:project
sonar.projectVersion=1.0
sonar.projectName=Springboot base
sonar.language=java
sonar.sourceEncoding=UTF-8

sonar.sources=.
sonar.java.binaries=.
```
这些配置实际是从网上找来的，目前来说，直接影响运行的是最后两个配置，如果配的不对，将会导致找不到文件。

## jenkins pipeline问题
做了sonar的基本配置后，还需要添加触发sonar扫描的步骤，也可以直接在jenkins页面上操作，不过想着这种可能多半不会在正式项目中使用，所以最终我选择了直接在jenkinsFile中增加相应的步骤，代码如下：
```
stage('Code Analysis'){
	steps{
		echo "【Code Analysis】"
		script{
			def sonarHome = tool name: 'sonarqube3.3.0',type: 'hudson.plugins.sonar.SonarRunnerInstallation'
			echo "${sonarHome}"
			withSonarQubeEnv('sonarqube3.3.0'){
				sh "${sonarHome}/bin/sonar-scanner -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_AUTH_TOKEN}"
			}
		}
	}
}
```
这里就有一个细节需要注意，这里的"tool name: 'sonarqube3.3.0'",从整体代码可以看到最终是获取了一个"sonar-scanner"命令的路径，很显然，这是客户端相关的，所以这里的"sonarqube3.3.0"其实是对应的sonarQube-scanner-cli中配置的那个名字。
而"withSonarQubeEnv('sonarqube3.3.0')"这个实际是sonar服务端配置相关的，主要是和"${SONAR_HOST_URL}"、"${SONAR_AUTH_TOKEN}"这两个变量有关系。
这两个变量也是在系统配置那里配置，然后配置SonarQube servers的时候需要勾选上"Environment variables"。
起初我没有明白这种关系，所以一开始以为都只是随便定义的，于是直接复制的代码没有修改，配置的是"withSonarQubeEnv('sonar')"，结果运行的时候就抛出了如下异常：
```
ERROR: SonarQube installation defined in this job (sonar) does not match any
```

## server url问题
在解决了上边的问题后，我依旧运行失败，日志中出现了如下异常:
```
ERROR: SonarQube server [192.168.19.200:9000] can not be reached
INFO: ------------------------------------------------------------------------
INFO: EXECUTION FAILURE
INFO: ------------------------------------------------------------------------
INFO: Total time: 0.651s
INFO: Final Memory: 3M/57M
INFO: ------------------------------------------------------------------------
ERROR: Error during SonarQube Scanner execution
ERROR: Unable to execute SonarQube
ERROR: Caused by: Fail to get bootstrap index from server
ERROR: Caused by: Expected URL scheme 'http' or 'https' but no colon was found
ERROR: 
ERROR: Re-run SonarQube Scanner using the -X switch to enable full debug logging.
```
这个提示相对还是比较清晰的，明确的说了预期url应该是带有"http"或者"https"的，我一开始在"SonarQube servers"中配置的是"192.168.19.200:9000"。
所以这个问题解决起来也很简单，就是在配置的环境变量"SONAR_HOST_URL"里加上协议。

## JSONException问题
在解决了上边协议配置问题再运行的时候，还是失败，日志中出现了如下的提示：
```
Pipeline] End of Pipeline
hudson.remoting.ProxyException: net.sf.json.JSONException: Invalid JSON String
	at net.sf.json.JSONSerializer.toJSON(JSONSerializer.java:143)
	at net.sf.json.JSONSerializer.toJSON(JSONSerializer.java:103)
	at net.sf.json.JSONSerializer.toJSON(JSONSerializer.java:84)
server——
```
这个问题，实际是解决上边问题时留下的坑。
我一开始没有看明白为什么，百度后才知道是因为我在加"http"协议的时候加的有问题，我直接复制了sonar页面上的url，修改"SONAR_HOST_URL"的同时，也修改了"Server URL"，复制的时候多了端口后的斜杠。
最终解决办法就是把"http://192.168.19.200:9000/"改成"http://192.168.19.200:9000"。

## 成功的结果
经过这些问题的处理后，终于用跑成功了一个集成了sonar的基础pipeline，jenkins运行结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4544ea7d152542f8ad9a10188bf8f54a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

sonar页面结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/74de1c8f7d234cc8a1d334de050df959.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5raC5a6X5YuL,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
