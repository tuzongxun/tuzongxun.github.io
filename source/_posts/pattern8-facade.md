---
title: 《设计模式》学习笔记8——外观模式
date: 2018-1-5 9:43:58
categories: [[读书笔记,设计模式],[设计模式]] #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [设计模式,读书笔记]
---
## 定义
外观模式引用书中的定义如下：
<!--more-->
>为子系统中的一组接口提供一个统一的入口。外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。 
>外观模式又称为门面模式，它是一种对象结构型模式。外观模式是迪米特法则的一种具体实
现，通过引入一个新的外观角色可以降低原有系统的复杂度，同时降低客户类与子系统的耦
合度。

## 理解
对于这个模式的理解，我觉得着重点在于几个词上，分别是`一组接口`、`统一入口`、`降低复杂度`。
结合我们目前的系统进行说明,首先说什么是一组接口。
在我们的系统中，由于业务需要，有三种日志，分别是系统日志、告警日志、监控日志，我用非正式的代码简单表示如下：
系统日志类接口定义：
<pre>
/**
 *日志统一管理类，外观类
 *@author tzx
 *@date 2018年1月4日
 */
public class LogHandle {
	private SysLogService sysLogService = new SysLogService();
	private AlarmLogService alarmLogService = new AlarmLogService();
	private MonitorLogService monitorLogService = new MonitorLogService();
	public void error(String msg, Object... args) {
		sysLogService.error(msg, args);
		alarmLogService.error(msg, args);
		monitorLogService.error(msg, args);
	}
}
</pre>
告警日志类接口定义：
<pre>
/**
 *告警日志接口
 *@author tzx
 *@date 2018年1月4日
 */
public class AlarmLogService {
	private Logger sysLog = LoggerFactory.getLogger(this.getClass());
	public void error(String msg, Object... args) {
		String sys = "alarm";
		msg = "#" + msg + "#" + sys + "#";
		sysLog.error(msg, args);
	}
}
</pre>
监控日志类接口：
<pre>
/**
 *监控日志接口
 *@author tzx
 *@date 2018年1月4日
 */
public class MonitorLogService {
	private Logger sysLog = LoggerFactory.getLogger(this.getClass());
	public void error(String msg, Object... args) {
		String sys = "monitor";
		msg = "|" + msg + "|" + sys + "|";
		sysLog.error(msg, args);
	}
}
</pre>
上边三个日志接口为了表示功能的不同，简单地对日志内容进行了不同的拼接，实际项目中自然不会这样。
这三个日志接口，都是属于日志的操作，只是打印出的日志内容被不同的系统采集再使用，而且他们基本上也都同时被使用到，所以可以说他们就是属于外观模式定义中的那`一组接口`。
在没有使用外观模式的情况下，我们在业务代码中需要打印日志的时候可能就会是下边所示这样：
<pre>
/**
 *博客操作类
 *@author tzx
 *@date 2018年1月4日
 */
public class BlogService {
	private SysLogService sysLogService = new SysLogService();
	private AlarmLogService alarmLogService = new AlarmLogService();
	private MonitorLogService monitorLogService = new MonitorLogService();
	public void addBlog() {
		try{
		System.out.println("");
		}catch (Exception e) {
			sysLogService.error("系统异常{}", new Object[] { e.getMessage() });
			alarmLogService.error("系统异常{}", new Object[] { e.getMessage() });
			monitorLogService.error("系统异常{}", new Object[] { e.getMessage() });
		}
	}
}
</pre>
<pre>
/**
 *用户操作类
 *@author tzx
 *@date 2018年1月4日
 */
public class UserService {
	private SysLogService sysLogService = new SysLogService();
	private AlarmLogService alarmLogService = new AlarmLogService();
	private MonitorLogService monitorLogService = new MonitorLogService();
	public void addUser() {
		try {
			System.out.println("");
		} catch (Exception e) {
			sysLogService.error("系统异常{}", new Object[] { e.getMessage() });
			alarmLogService.error("系统异常{}", new Object[] { e.getMessage() });
			monitorLogService.error("系统异常{}", new Object[] { e.getMessage() });
		}
	}
}
</pre>
很显然，两个service中有大量重复的代码。这里只有两个类打印日志，就需要一共调用6次日志打印方法，每多出一个类需要打印这样的日志，就会以3的倍数增加代码。
而且这里的示例几乎是最简单的，在实际业务开发的时候可能就远不止三行。
同时，像上边这种做法，假如后续日志系统增多，需要增加到四种甚至五种或者更多，那个每个打印日志的类都需要进行相应的修改，很显然，这是极不利于维护和拓展的。
因此，按照常规的思路，可能就会有两种做法：
一种是把这种打印日志的代码进行提取，封装为一个方法进行调用。但是这种做法会存在一定的问题，首先，这个方法本身和业务是无关的，放在业务类中不合适。其次，除非所有需要打印日志的类都有一个共同的父类，把这个方法放在父类中，否则就需要定义多次该方法。
于是乎，就有了第二种做法，那就是把这段代码提取到另外一个类中，那么这个类就是所谓的外观类了：
<pre>
/**
 *日志统一管理类，外观类
 *@author tzx
 *@date 2018年1月4日
 */
public class LogHandle {
	private SysLogService sysLogService = new SysLogService();
	private AlarmLogService alarmLogService = new AlarmLogService();
	private MonitorLogService monitorLogService = new MonitorLogService();
	public void error(String msg, Object... args) {
		sysLogService.error(msg, args);
		alarmLogService.error(msg, args);
		monitorLogService.error(msg, args);
	}
}
</pre>
有了这样一个类之后，我们的业务代码调用的时候就只需要一次就可以搞定，而这个管理日志操作的类就是`统一接口`。
<pre>
/**
 *用户操作类
 *@author tzx
 *@date 2018年1月4日
 */
public class UserService {
	private LogHandle logHandle = new LogHandle();
	public void addUser() {
		try {
			System.out.println("");
		} catch (Exception e) {
			logHandle.error("系统异常{}", new Object[] { e.getMessage() });
		}
	}
}
</pre>
与此同时，如果后续日志系统变化，不论是增加新的日志类型，还是删减旧的日志类型，都只需要改动logHandle类就可以了，而且每个调用日志的地方代码也都减少了很多，这就大大`降低了系统的复杂度`。

## 要点
那么根据上边的理解和示例，外观模式有如下一些要点：
1. 需要有一组接口，他们几乎都是成套被使用；
2. 一个外观类或接口，实际操作上边的一组接口，而提供一个公开的接口方法供外部调用；
3. 客户端只需要一次调用外观类的接口，而不需要分别调用最开始那一组接口的每一个。

在没有学这个模式的时候，我觉得单例模式是最简单的设计模式，而现在来看，这个似乎才是最简单的设计模式。


demo源码可在github下载：<https://github.com/tuzongxun/mypattern>
