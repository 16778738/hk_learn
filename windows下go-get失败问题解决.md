---
title: windows下go get失败问题解决
tags: '-go'
abbrlink: 60453
date: 2022-04-23 11:32:56
---

使用go get 命令总是报错，发现在国内需要使用需要访问外网

原因是缺少golang.org/x/net 的依赖包，github已经有托管依赖包，安装下载其依赖包就能解决了：

~~~~
#%GOPATH%---是安装go时设置的变量名称，GOPATH路径
mkdir -p %GOPATH%\src\golang.org\x 
cd %GOPATH%\src\golang.org\x 
git clone https://github.com/golang/net.git
~~~~

完成后可以使用go get 命令试一下

![image-20220423113435773](E:\学习\picture\image-20220423113435773.png)
