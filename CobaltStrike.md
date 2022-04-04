# Cobalt Strike

CS是什么？

~~~~
Cobalt Strike是一款渗透测试神器，常被业界人称为CS神器。Cobalt Strike已经不再使用MSF而是作为单独的平台使用，它分为客户端与服务端，服务端是一个，客户端可以有多个，可被团队进行分布式协团操作。

Cobalt Strike集成了端口转发、扫描多模式端口Listener、Windows exe程序生成、Windows dll动态链接库生成、java程序生成、office宏代码生成，包括站点克隆获取浏览器的相关信息等。

早期版本Cobalt Srtike依赖Metasploit框架，而现在Cobalt Strike已经不再使用MSF而是作为单独的平台使用。

这个工具的社区版是大家熟知的Armitage(一个MSF的图形化界面工具)，而Cobalt Strike大家可以理解其为Armitage的商业版。
~~~~

## 开启服务器

```cmd
teamserver your_ip your_passowrd [config_file]
```

**创建监听器**

在CS客户端中打开 Cobalt Strike —》Listeners，之后点击Add，此时弹出New Listener窗口，在填写监听器的相关信息之前，需要先来了解监听器有哪些类型。

Cobalt Strike有两种类型的监听器：

- Beacon

  Beacon直译过来就是灯塔、信标、照亮指引的意思，Beacon是较为隐蔽的后渗透代理，Beacon监听器的名称例如：

  ```text
  windows/beacon_http/reverse_http
  ```

- Foreign

  Foreign直译就是外部的，这里可以理解成`对外监听器`，这种类型的监听器主要作用是给其他的Payload提供别名，比如Metasploit 框架里的Payload，笔者个人理解Foreign监听器在一定程度上提高了CS的兼容性。

### MSF 与 CS 的结合利用

如果想使用MSF对目标进行漏洞利用，再通过这个漏洞来传输Beacon的话，也是可以的。

1、首先在MSF上选择攻击模块

2、接着在MSF上设置Payload为`windows/meterpreter/reverse_http`或者`windows/meterpreter/reverse_https`，这么做是因为CS的Beacon与MSF的分阶段协议是相兼容的。

3、之后在MSF中设置Payload的LHOST、LPORT为CS中Beacon的监听器IP及端口。

4、然后设置 `DisablePayloadHandler` 为 True，此选项会让 MSF 避免在其内起一个 handler 来服务你的 payload 连接，也就是告诉MSF说我们已经建立了监听器，不必再新建监听器了。

5、再设置 `PrependMigrate` 为 True，此选项让 MSF 前置 shellcode 在另一个进程中运行 payload stager。如果被利用的应用程序崩溃或被用户关闭，这会帮助 Beacon 会话存活。

6、最后运行`exploit -j`，-j 是指作为job开始运行，即在后台运行。

## 鱼叉式网络钓鱼 

用CS进行钓鱼需要四个步骤：

1、创建一个目标清单

2、制作一个邮件模板或者使用之前制作好的模板

3、选择一个用来发送邮件的邮件服务器

4、发送邮件

