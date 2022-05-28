---
title: wpscan学习使用
tags:
  - wpscan
abbrlink: 30079
date: 2022-05-25 10:40:57
---

# wpscan学习使用

```go
wpscan --update 更新漏洞库
--url   | -u <target url>  要扫描的`WordPress`站点.
--force | -f   不检查网站运行的是不是`WordPress`
--enumerate | -e [option(s)]  枚举
```

（1） 扫描wordpress用户
wpscan --url http://www.xxxxx.xxx/ --enumerate u

（2）扫描主题
wpscan --url http://www.xxxxx.xxx/ --enumerate t

（3）扫描主题中的漏洞
wpscan --url http://www.xxxxx.xxx/ --enumerate vt

（4）扫描插件
wpscan --url http://www.xxxxx.xxx/ --enumerate p

（5）扫描插件中的漏洞
wpscan --url http://www.xxxxx.xxx/ --enumerate vp

（6）使用WPScan进行暴力破解
wpscan --url http://www.xxxxx.xxx/ -e u --wordlist /root/桌面/password.txt

-P -U 后面的参数最好使用文件的绝对路径
wpscan --url http://www.xxxxx.xxx/ /home//passwords.txt -U /home/username.txt

（7）api token使用
wpscan --url https://www.xxxxx.xxx/ --disable-tls-checks --api-token +获取到的token

（8）https 情况下
-disable-tls-checks #禁用SSL/TLS证书验证。

wpscan --url https://www.xxxxx.xxx/ --enumerate vt --disable-tls-checks
