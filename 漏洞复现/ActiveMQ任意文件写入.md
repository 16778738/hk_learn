~~~~~
title: 漏洞复现学习(ActiveMQ)
tags: 
  - 渗透测试
  - 漏洞复现
  - ActiveMQ
  -	(CVE-2016-3088)
categories: 
  - 渗透测试
  - ActiveMQ
  - 漏洞复现
  - (CVE-2016-3088)
~~~~~



# ActiveMQ任意文件写入漏洞 (CVE-2016-3088)学习

## 漏洞描述

1. 影响版本：Apache ActiveMQ 5.x~5.14.0
2. 漏洞产生原因：ActiveMQ的web控制台分三个应用，admin、api和fileserver，其中admin是管理员页面，api是接口，fileserver是储存文件的接口；admin和api都需要登录后才能使用，fileserver无需登录。本漏洞出现在fileserver应用中，漏洞原理其实非常简单，就是fileserver支持写入文件（但不解析jsp），同时支持移动文件（MOVE请求）。所以，我们只需要写入一个文件，然后使用MOVE请求将其移动到任意位置，造成任意文件写入漏洞。

## 环境部署

使用win10子系统的Ubuntu安装docker

查看版本信息

```text
http://host:8161/admin/index.jsp?printable=true
```

![image-20220328210805330](E:\学习\picture\image-20220328210805330.png)

## 漏洞利用

1. 首先访问网址查看ActiveMQ的绝对路径：

   ```
   http://127.0.0.1:8161/admin/test/systemProperties.jsp
   ```

   ![image-20220328211038349](E:\学习\picture\image-20220328211038349.png)

2. 然后使用burp进行put，webshell

   - get访问/fileserver/3.txt，将burp拦截的请求进行重构repeater

   ![image-20220328211516107](E:\学习\picture\image-20220328211516107.png)

   - 替换信息，写入有webshell的txt文本

   ![image-20220328211905848](E:\学习\picture\image-20220328211905848.png)

   - 移动到web目录下的api文件夹（`/opt/activemq/webapps/api/s.jsp`）中：

     ```
     Destination:file:///opt/activemq/webapps/api/5.jsp
     ```

   ![image-20220328212314101](E:\学习\picture\image-20220328212314101.png)

   网站进行访问查看api/下是否存在webshell

   ![image-20220328212422354](E:\学习\picture\image-20220328212422354.png)

进行访问：

![image-20220328212532074](E:\学习\picture\image-20220328212532074.png)

![image-20220328212706122](E:\学习\picture\image-20220328212706122.png)

## 修复方案

1、ActiveMQ Fileserver 的功能在 5.14.0 及其以后的版本中已被移除。建议用户升级至 5.14.0 及其以后版本。

2、通过移除 conf\jetty.xml 的以下配置来禁用 ActiveMQ Fileserver 功能

3、打补丁

http://activemq.apache.org/security-advisories.data/CVE-2016-3088-announcement.txt

4、电脑管家等修复漏洞工具