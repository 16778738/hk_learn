# Linux安全管理

1. 禁用不适用的用户

   ~~~markdown
   cat /etc/passwd
   cat /etc/group
   
   加固方式：
   username -L user 或 passwd -l user /锁定用户
   username -U user 或 passwd -u user /解锁用户
   ~~~

2. 用户密码策略

   ~~~text
   1.检查
   /etc/pam.d/system-auth
   /etc/pam.d/passwd
   /etc/pam.d/common-password
   是否进行了口令复杂度设置
   password requisite pam_cracklib.so 
   ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1
   2./etc/login.defs中是否进行了口令周期设置
   	cat /etc/login.defs | grep PASS 或change -l user
   ~~~

   加固方式

   ![image-20220326122742773](E:\学习\picture\image-20220326122742773.png)

   

3. 使用ssh协议

   service status ssh

4. 禁止root用户远程登录

   ~~~
   cat /etc/ssh/sshd_config | grep PermitRootLogin
   
   vi /etc/ssh/sshd_config 
   PermitRootLogin 设置为no
   
   禁止root用户远程telnet：
   编辑 /etc/pam.d/login,配置auth required pam_securetty.so
   ~~~

5. 设置会话超时退出

   ~~~~
   cat /etc/profile |grep TMOUT
   
   加固
   root运行， vi /etc/profile ,增加export TMOUT=600(单位秒)
   ~~~~

6. 设置登录失败次数并锁定

   ![image-20220326132504709](E:\学习\picture\image-20220326132504709.png)

7. 日志记录

   ![image-20220326132647601](E:\学习\picture\image-20220326132647601.png)

   ![image-20220326132741666](E:\学习\picture\image-20220326132741666.png)

   

8. 日志远程存储

   ![image-20220326132817166](E:\学习\picture\image-20220326132817166.png)

   

9. 减少history命令记录

   ![image-20220326133016808](E:\学习\picture\image-20220326133016808.png)

   

10. 关闭不需要的服务

![image-20220326133146250](E:\学习\picture\image-20220326133146250.png)