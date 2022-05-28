---
title: nmap的学习使用
tags:
  - nmap
abbrlink: 7336
date: 2022-05-26 19:39:36
---

# nmap的学习使用





# 目标说明

`-iL` 从已有的ip列表文件中读取并扫描

`-iR+扫描数量` 随机选择目标进行扫描

`--exclude+ip` 不扫描此ip

# 主机发现

`-sL` 列出要扫描的ip

`-sn` 不进行端口扫描

**`-Pn` 将所有主机都默认为在线，跳过主机发现**

`-PS/PA/PU/PY` 使用TCP、SYN/ACK、udp或SCTP协议去发现端口

`-PE/PP/PM`：使用ICMP响应（echo）、时间戳或子网掩码请求来发现探测

`-PO` 使用IP协议的ping

`-n` 不做DNS解析

`-R` 总是做DNS反向解析

`--dns-servers`指定自定义的DNS服务器

`--system-dns` 使用操作系统的DNS

`--traceroute` 追踪每台主机的跳转路径

# 扫描技术

**`-sS/sT/sA/sW/sM`：使用SYN、TCP、全连接Connect()、ACK、Window、Maimon来进行扫描**

`-sU` UDP扫描

`-sN/sF/sX` 使用TCP Null(无flag)、FIN、Xmas（FIN+Push+Urgent）扫描

`--scanflags +flags` 自定义TCP扫描的flags

`-sI` 僵尸机扫描

`-sY/sZ` 使用SCTP协议的INIT/COOKIE-ECHO扫描

`-sO` 进行IP协议扫描

`-b <FTP relay host>`：指定FTP中继主机进行FTP反弹扫描
端口说明和扫描规则
`-p` 只扫描指定的端口

`--exclude-ports` 不对此端口进行扫描

**`-F` 快速模式，扫描比默认端口数量更少的端口**

`-r` 有序地扫描端口而不是随机地扫描

`--top-ports <number>` 扫描排名指定的数字前几位的最常用的端口

`--port-ratio <ratio>` 扫描比输入的比例更常用的端口

# 服务、版本探测

**`-sV`：探测开启的端口来获取服务、版本信息**

`--version-intensity <level>`：设置探测服务、版本信息的强度

`--version-light`：强度为2的探测强度

`--version-all`：强度为9的探测强度

`--version-trace`：将扫描的具体过程显示出来

# 脚本扫描

`-sC`：等同于–script=default

`--script=<Lua scripts>`：指定使用Lua脚本进行扫描

`--script-args=<n1=v1,[n2=v2,...]>`：指定脚本的参数

`--script-args-file=filename`：指定提供脚本参数的文件

`--script-trace`：显示全部发送和收到的数据

`--script-updatedb`：更新脚本的数据库

`--script-help=<Lua scripts>`：显示脚本的相关信息

# 系统探测

**`-O`：进行系统探测**

`--osscan-limit`：限制系统探测的目标，如只探测Linux系统

`--osscan-guess`：更侵略性地猜测系统

# 定时和性能

**`-T<0-5>`：设置时序模块，越高越快**

`--min-hostgroup/max-hostgroup <size>`：指定最小、最大的并行主机扫描组大小

`--min-parallelism/max-parallelism <numprobes>`：指定最小、最大并行探测数量

`--min-rtt-timeout/max-rtt-timeout/initial-rtt-timeout <time>`：指定最小、最大的扫描往返时间

`--max-retries <tries>`：指定最大的重发扫描包的次数

`--host-timeout <time>`：指定超时时间

`--scan-delay/--max-scan-delay <time>`：指定每次探测延迟多长时间，即两次探测之间间隔多少时间

`--min-rate <number>`：最小的发包速率

`--max-rate <number>`：最大的发包速率

# 防火墙、IDS绕过和欺骗

`-f; --mtu <val>`：设置MTU最大传输单元

**`-D <decoy1,decoy2[,ME],...>`：伪造多个IP地址和源地址一同发送包，从而隐藏在众多的IP地址中而不易被发现**

`-S <IP_Address>`：伪造源地址

`-e <iface>`：使用指定的接口

`-g/--source-port <portnum>`：使用指定的源端口

`--proxies <url1,[url2],...>`：指定代理服务器进行扫描

`--data <hex string>`：在发送包的数据字段中追加自定义的十六进制字符串

`--data-string <string>`：在发送包的数据字段中追加自定义的ASCII字符串

`--data-length <num>`：在发送包的数据字段中追加随机的数据

`--ip-options <options>`：使用指定的IP选项发送包

`--ttl <val>`：设置TTL值

`--spoof-mac <mac address/prefix/vendor name>`：伪造源Mac地址

`--badsum`：发送伪造TCP/UDP/SCTP校验和Checksum的数据包

# 输出

`-oN/-oX/-oS/-oG <file>`：分别输出正常、XML、s|

# 杂项

`-6`：扫描IPv6的地址

`-A`：一次扫描包含系统探测、版本探测、脚本扫描和跟踪扫描

`--datadir <dirname>`：指定自定义的数据文件位置

`--send-eth/--send-ip`：使用原始以太网帧或IP数据包发送

`--privileged`：假设用户有全部权限

`--unprivileged`：假设用户缺少原始套接字权限

**`-V`：输出版本号**

`-h`：输出帮助信息

~~~~
nmap --iflist（查看本地路由与接口）

nmap -e 08:00:27:47:63:E6 103.10.87.148（指定mac和ip地址）

nmap -T4 -F -n -Pn -D 192.168.1.100,192.168.1.101,192.168.1.102,ME192.168.1.103（地址诱骗）

nmap -sV --spoof-mac 08:00:27:47:63:E6 103.10.87.148（虚假mac地址）

nmap -sV --source-port 900 103.10.87.148 --source-port（指定源端口）

nmap -p1-25,80,512-515,2001,4001,6001,9001 10.20.0.1/16（扫描思科路由器）

nmap -sU -p69 -nvv 192.168.1.253（扫描路由器的ftp协议）

nmap -O -F -n 102.10.87.148（-F快速扫描）

nmap -iR 100000 -sS -PS80 -p 445 -oG nmap.txt（随机产生10万个ip地址，对其445端口进行扫描，扫描结果以greppable（可用grep命令提取）格式输出到nmap.txt）

nmap --script=brute 102.10.87.148（暴力破解）
~~~~

