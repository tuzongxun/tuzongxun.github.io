---
title: cookie、session和java过滤器理解和使用
date: 2016-11-07 15:09:42
tags: [java,cookie,session,过滤器]
toc: true
categories: java
---
**基础知识理解：**

cookie、session和过滤器通常都是用在web应用中，cookie和session用来保存一定的数据，过滤器Filter则是在浏览器发出请求之后，而后台执行特定的请求之前发生一定的作用。之所以把这三个放一起，是因为有很多时候都会是把他们结合在一起使用，例如有些登陆程序。
<!--more-->
cookie是浏览器的机制，session是服务器的机制，但是实际上cookie也是由服务器生成的，之后返回给浏览器的，并不是浏览器本身生成。当浏览器发送某个请求时，如果拥有有效的cookie则会把这个cookie带在一起。

之所有会有cookie的使用，是因为http协议原本是无状态协议，也就是说通过http协议本身，服务器不能判断浏览器是否之前访问过。

Filter和servlet的写法相似，编写相关代码的时候需要实现Filter接口并重写相关的方法，通常更改较多的是doFilter方法。Filter代码写好以后如果需要发生效用，需要像配置servlet一样在web.xml中 进行一定的配置。

以下是一个简单的结合cookie、session、Servlet和Filter的登陆示例代码：

定义一个用户实体类，充当数据库数据，这里使用单例模式，保证只存在一个实例对象：
<pre>
package models;  
/** 
 *用户信息实体类  
 *@author tuzongxun123 
 */  
public class UserModel {  
    private String userName;  
    private String password;  
    // 单例模式，保证只有一个用户对象实例  
    public static UserModel getInstance() {  
        UserModel user = new UserModel("zhangsan", "123456");  
        return user;  
    }  
    private UserModel(String userName, String pasword) {  
        this.userName = userName;  
        this.password = pasword;  
    }  
    public String getUserName() {  
        return userName;  
    }  
    public String getPassword() {  
        return password;  
    }  
}  
</pre>


用户登陆输入信息index.jsp界面，在form表单的action中使用jsp的特性获得项目路径：
<pre>
&lt;%@ page language="java" import="java.util.*" contentType="text/html; charset=utf-8"  
    pageEncoding="utf-8"%>  
&lt;!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">  
&lt;html>  
&lt;head>  
&lt;meta http-equiv="Content-Type" content="text/html; charset=utf-8">  
&lt;title>cookieAndFilterTest</title>  
&lt;/head>  
&lt;body>  
     &lt;form action="<%=request.getContextPath() %>/loginServlet" method="post">  
        userName：&lt;input type="text" name="userName" />&lt;/br>  
        password：&lt;input type="password" name="password" />&lt;/br>  
        &lt;input type="submit" value="login"/>  
     &lt;/form>  
&lt;/body>  
&lt;/html>  
</pre>


对应的后台servlet：
<pre>
package servletTest;  
import java.io.IOException;  
import javax.servlet.ServletException;  
import javax.servlet.http.Cookie;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import javax.servlet.http.HttpSession;  
import models.UserModel;  
public class LoginServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
            throws ServletException, IOException {  
        this.doPost(req, resp);  
    }  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)  
            throws ServletException, IOException {  
        String userName = req.getParameter("userName");  
        String password = req.getParameter("password");  
        // 模拟数据库数据  
        UserModel user = UserModel.getInstance();  
        String dbUserName = user.getUserName();  
        String dbPassword = user.getPassword();  
        if (dbUserName.equals(userName) && dbPassword.equals(password)) {  
            // 用户名和密码都匹配，证明登陆成功，设置session和cookie  
            HttpSession session = req.getSession();  
            session.setAttribute("userName", userName);  
            session.setAttribute("password", password);  
            Cookie cookie = new Cookie("userName", userName);  
            Cookie cookie2 = new Cookie("password", password);  
            // 设置cookie的存储时长  
            cookie.setMaxAge(60);  
            cookie2.setMaxAge(60);  
            // 把cookie发送给浏览器  
            resp.addCookie(cookie);  
            resp.addCookie(cookie2);  
            // 转发请求到用户列表  
            req.getRequestDispatcher("/userList").forward(req, resp);  
        } else {  
            // 转发请求到登陆页面  
            req.getRequestDispatcher("index.jsp").forward(req, resp);  
        }  
    }  
}  
</pre>


上边登陆后跳转的请求：
<pre>
package servletTest;   
import java.io.IOException;  
import javax.servlet.ServletException;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import models.UserModel;  
public class UserListServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
            throws ServletException, IOException {  
        this.doPost(req, resp);  
    }  
    @Override  
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)  
            throws ServletException, IOException {  
        UserModel user = UserModel.getInstance();  
        //在浏览器中打印出用户列表书数据  
        resp.getWriter().write(  
                "userName:" + user.getUserName() + "," + "password:"  
                        + user.getPassword());  
    }  
}  
</pre>


项目web.xml配置：
<pre>
&lt;?xml version="1.0" encoding="UTF-8"?>  
&lt;web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"  
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"  
    id="WebApp_ID" version="2.5">      
  &lt;!-- 访问时的项目名称 -->  
  &lt;display-name>cookieAndFilterTest&lt;/display-name>  
  &lt;!-- servlet配置 -->  
  &lt;servlet>  
     &lt;servlet-name>login&lt;/servlet-name>  
     &lt;servlet-class>servletTest.LoginServlet&lt;/servlet-class>  
  &lt;/servlet>  
  &lt;servlet-mapping>  
     &lt;servlet-name>login&lt;/servlet-name>  
     &lt;url-pattern>/loginServlet&lt;/url-pattern>  
  &lt;/servlet-mapping>  
  &lt;servlet>  
     &lt;servlet-name>userList&lt;/servlet-name>  
     &lt;servlet-class>servletTest.UserListServlet&lt;/servlet-class>  
  &lt;/servlet>  
  &lt;servlet-mapping>  
     &lt;servlet-name>userList&lt;/servlet-name>  
     &lt;url-pattern>/userList&lt;/url-pattern>  
  &lt;/servlet-mapping>  
  &lt;!-- 过滤器设置，浏览其发送请求后首先会走这里 -->  
  &lt;filter>  
     &lt;filter-name>loginFilter&lt;/filter-name>  
     &lt;filter-class>filterTest.FilterTest&lt;/filter-class>  
  &lt;/filter>  
  &lt;filter-mapping>  
     &lt;filter-name>loginFilter&lt;/filter-name>  
     &lt;url-pattern>/*&lt;/url-pattern>  
  &lt;/filter-mapping>  
  &lt;!-- 输入项目名访问的默认页面 -->  
  &lt;welcome-file-list>  
    &lt;welcome-file>index.jsp&lt;/welcome-file>  
  &lt;/welcome-file-list>  
&lt;/web-app>  
</pre>

java过滤器代码：
<pre>
package filterTest;   
import java.io.IOException;  
import javax.servlet.Filter;  
import javax.servlet.FilterChain;  
import javax.servlet.FilterConfig;  
import javax.servlet.ServletException;  
import javax.servlet.ServletRequest;  
import javax.servlet.ServletResponse;  
import javax.servlet.http.Cookie;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import models.UserModel;  
public class FilterTest implements Filter {  
    @Override  
    public void destroy() {  
    }  
    @Override  
    public void doFilter(ServletRequest request, ServletResponse response,  
            FilterChain chain) throws IOException, ServletException {  
        // 登陆请求、初始请求直接放行  
        HttpServletRequest req = (HttpServletRequest) request;  
        HttpServletResponse resp = (HttpServletResponse) response;  
        String uri = req.getRequestURI();  
        if ("/cookieAndFilterTest/loginServlet".equals(uri)  
                || "/cookieAndFilterTest/".equals(uri)) {  
            // 放行  
            chain.doFilter(request, response);  
            return;  
        }  
        // 不是登陆请求的话，判断是否有cookie  
        Cookie[] cookies = req.getCookies();  
        if (cookies != null && cookies.length > 0) {  
            String userName = null;  
            String password = null;  
            // 判断cookie中的用户名和密码是否和数据库中的一致，如果一致则放行，否则转发请求到登陆页面  
            for (Cookie cookie : cookies) {  
                if ("userName".equals(cookie.getName())) {  
                    userName = cookie.getValue();  
                }  
                if ("password".equals(cookie.getName())) {  
                    password = cookie.getValue();  
                }  
            }  
            UserModel user = UserModel.getInstance();  
            if (user.getUserName().equals(userName)  
                    && user.getPassword().equals(password)) {  
                chain.doFilter(request, response);  
                return;  
            } else {  
                // 重定向到登陆界面  
                req.getRequestDispatcher("/index.jsp").forward(req, resp);  
                return;  
            }  
        } else {  
            req.getRequestDispatcher("/index.jsp").forward(req, resp);  
            return;  
        }  
    }  
    @Override  
    public void init(FilterConfig arg0) throws ServletException {  
    }  
}
</pre>  