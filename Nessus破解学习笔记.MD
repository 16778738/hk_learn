# Nessus破解安装笔记

## Windows安装

![image-20220406202836583](C:\Users\86157\AppData\Roaming\Typora\typora-user-images\image-20220406202836583.png)

**Nessus配置SSL**

浏览器自动访问

http://localhost:8834/WelcomeToNessus-Install/welcome

注：如果返回error

则访问https://localhost:8834/或者浏览器访问

https://127.0.0.1:8834:8834



选择Managed Scanner再点击Continue

随便填写用户名密码进行注册

比如用户名root密码root

安装完成后没有Scan扫描按钮，尚未激活，暂不能扫描.

## 破解

管理员身份打开CMD输入如下命令（停止Nessus服务）

命令如下：

net stop "Tenable Nessus"

![image-20220406203157215](E:\学习\picture\image-20220406203157215.png)

将plugin_feed_info.inc替换Nessus目录下的文件

**两个均需要替换**

![image-20220406203256389](E:\学习\picture\image-20220406203256389.png)

开启Nessus服务(启动Nessus服务)

net start "Tenable Nessus"



## 更新插件

根据自己的位置进行更新

![image-20220406203346223](E:\学习\picture\image-20220406203346223.png)

~~~~
D:\Nessus10.1.2>nessuscli.exe update "D:\BaiduNetdiskDownload\Nessus10.1.2破解及插件\all-2.0_202203282143.tar.gz"
~~~~

更新成功后会在自己的扫面插件中查看