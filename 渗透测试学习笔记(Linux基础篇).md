# Linux基础操作学习

exploit-db.com：内核漏洞发布网站

内核版本：XX.YY.ZZ（XX代表主版本号，YY代表次版本号，ZZ代表补丁次数）

次版本号奇数代表开发版，偶数代表稳定版



## Linux系统目录结构：

-  root：管理员家目录
-  bin：存放二进制可执行文件（ls，cat等），即存放普通命令
-  sbin：存放管理员执行的系统命令
-  boot：存放用于系统引导时的各种文件
-  dev：存放设备文件，磁盘硬盘等
-  etc：存放系统管理和配置文件
-  home：用户家目录，相当于windows用户目录、
-  var：放日志文件，网站根目录
-  lib：存放库文件
- 用于存放系统应用程序，最庞大的目录，要用到的应用程序和文件几乎都在这里
-  media，mnt：挂载光驱、光盘目录
-  lost+found：平常是空的，系统非正常关机而无家可归的文件在这里
-  proc：系统开机后临时生成的文件
-  srv：存放一些服务
-  **tmp**：用于存放各种临时文件，是公用的临时文件存储点，有源代码编译权限
-  opt：存放外部程序
-  selinux：服务防火墙
-  sys：存放系统文件

## 二、Linux常用命令

1.文件操作类

~~~~markdown
a. pwd：查看当前路径
b. cd 目录：转到目录
    cd .. ：返回上一级目录
    cd - ：后退
c. du -sh：统计文件大小
d. ls：查看文件，文件夹
    -l：长格式显示详细信息
    -a：显示所有子目录和文件信息，包括隐藏文件
    -d：显示目录本身属性
    -R：递归显示内容
    -h：以更易读的方式（K,M等）显示大小
e. chomd （39:50）这里看视频更方便一些
    权限：r读（4），w写（2），x执行（1），d表示为文件夹
    用户：u所有者，g用户分组，o其他人，a所有人
    chomd g+x 1.txt：对1.txt的用户组添加可执行权限（g-x即为去除权限）
    chomd 777 a.txt：所有权限设置为可读可写可执行
    chown test:test a.jpg：改变所有者和用户组
    搭建网站时：chown -R apache:apache /var/www/html 递归缩小权限
f. touch：创建文件或更新文件时间标记
g. mkdir：创建目录 -p创建递归目录
h. cp：复制文件或目录 -r递归复制 -f强制覆盖 -p保持源文件属性不变
i. rm：删除文件或目录 -r递归 -f强制
j. mv：移动文件或目录
k. wc：统计文件中出现的单词数量，字节数量和行数
l. cat：查看文件内容
find：查找文件或目录
    格式：find [查找范围] [查找条件] -name：按文件名查找
    -size：按文件大小查找
    -user：按文件属性查找
    -type：按文件类型查找
m.压缩命令
gzip：gzip [文件] [压缩文件名].gz
         gzip -d xxx.gz 解压缩
         bzip2：bizp2 [文件] [压缩文件名].bz2 解压缩同上
n.打包命令tar
    tar -cvf x.tar 1 2 3 把1,2,3文件打包为x.tar，可以再配合压缩
    tar -xvf x.tar 解包
    tar -tvf x.tar 查看包里文件
    tar -rvf x.tar 4 把4追加到x.tar中
    tar -jxvf x.tar.bz2 bz2下一步解压所有文件
    tar -zxvf x.tar.bz2 gzip下一步解压所有文件
~~~~

2.系统操作命令

~~~~markdown
a. uname -r：查看系统内核版本，-a查看完整信息
b. hostname：查看主机名（后加名称可更改）
c. ifconfig：查看网卡ip信息（dhclient eth0：获取网卡ip）
d. cat /proc/cupinfo：查看系统cpu信息
cat /proc/meminfo：查看系统内存信息
e. reboot：重启
f. Halt：关机
    用户账户命令（1:10:30）
    创建用户：useradd [用户]
    删除用户：userdel -r [用户] -r表示连用户宿主目录一并删除
    etc/passwd：存放系统账号
	linux查看管理员是看uid和pid号，0为管理员
h. ps -aux：查看系统进程
    top：动态查看进程 q退出
    kill：杀死进程
netstat -tnlp：查看本地开放端口信息
netstat -an：查看与外部连接情况详细信息
~~~~

~~~~
LAMP平台：Linux+Apache+Mysql+PHP
LNMP平台：Linux+Nginx+Mysql+PHP
WAMP平台：Winodws+Apache+Mysql+PHP
WNMP平台：Winodws+Nginx+Mysql+PHP
~~~~

3.vi编辑器：

~~~~markdown
语法：vi [-options] [+[n]] [file]
    -r用于恢复系统突然崩溃时正在编辑的文件，-R用于以只读方式打开文件
    +n用来指明进入vi后直接位于文件的第n行，如果不指定n，则位于最后一行
命令模式：进入即为命令模式，可输入命令操控
    i. G：进入文本尾部 gg：返回文首
    ii. ctrl+g：代表显示信息，行号
    iii. dd：删除光标所在行 D：删除光标所在位置到行尾 6dd：删除6行
    iv. yy：复制当前行 p：粘贴 dd完在p相当于剪切
插入模式：按a，i，o任意一个键进入插入模式，当记事本用
底行模式：esc到命令模式，按shift+：
    i. :wq：保存并退出
    ii. :q！：退出不保存
~~~~

5.**LAMP****搭建配置**

~~~~markdown
1. 配置好yum环境
2. 挂载光驱 mount /dev/sr0 /media
3. yum -y install httpd mysql mysql-server php php-mysql 安装这几个包
4. service httpd start
service mysqld start
5. mysqladmin -u root password 123123 为mysql设置账号密码
6. 拖源码
7. chown -R apache:apache /var/www/html 改权限
~~~~

**搭建旁站**

~~~markdown
Apache配置文件：vi /etc/httpd/conf/httpd.conf
Listen 80：侦听的端口
DocumentRoot：网站根目录
DirectoryIndex：默认首页
~~~

