---
title: 完美解决centos8 appstream问题
tags: centos8
abbrlink: 60074
date: 2022-05-06 18:57:55
---

# 【Centos8】Linux 为 repo ‘AppStream‘ 下载元数据失败、Could not resolve host: mirrors.cloud.aliyuncs.com

解决问题：

运行以下命令备份之前的repo文件。

```shell
rename '.repo' '.repo.bak' /etc/yum.repos.d/*.repo
```

运行以下命令下载最新的repo文件

~~~
wget https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo -O /etc/yum.repos.d/Centos-vault-8.5.2111.repo
wget https://mirrors.aliyun.com/repo/epel-archive-8.repo -O /etc/yum.repos.d/epel-archive-8.repo
~~~

运行以下命令替换repo文件中的链接，就是这一步出错了

官方的

~~~
sed -i 's/mirrors.cloud.aliyuncs.com/url_tmp/g'  /etc/yum.repos.d/Centos-vault-8.5.2111.repo &&  sed -i 's/mirrors.aliyun.com/mirrors.cloud.aliyuncs.com/g' /etc/yum.repos.d/Centos-vault-8.5.2111.repo && sed -i 's/url_tmp/mirrors.aliyun.com/g' /etc/yum.repos.d/Centos-vault-8.5.2111.repo
sed -i 's/mirrors.aliyun.com/mirrors.cloud.aliyuncs.com/g' /etc/yum.repos.d/epel-archive-8.repo
~~~

http://mirrors.cloud.aliyuncs.com需要替换为http://mirrors.aliyun.com，但是官方提供的命令没替换完，如果有执行官方提供的命令还是不行的话执行下面的命令：

~~~~~
sed -i 's/mirrors.cloud.aliyuncs.com/mirrors.aliyun.com/g'  /etc/yum.repos.d/Centos-vault-8.5.2111.repo 
sed -i 's/mirrors.cloud.aliyuncs.com/mirrors.aliyun.com/g'  /etc/yum.repos.d/epel-archive-8.repo
~~~~~

运行以下命令重新创建缓存

yum clean all && yum makecache

执行成果，yum install也可以正常使用了
