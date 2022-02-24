# FUZZ

## *什么是Fuzz技术?* 

~~~markdown
Fuzz是一种基于黑盒的自动化软件模糊测试技术,简单的说一种懒惰且暴力的技术融合了常见的以及精心构建的数据文本进行网站、软件安全性测试;
~~~

*Fuzz的核心思想:*

- 目录Fuzz(漏洞点)
- 参数Fuzz(可利用参数)
- PayloadFuzz(bypass)

针对一部分网站可以扫描的全面，只要你的字典足够强大就可以扫描到绝大多部分的目录和文件

## 应用场景

- 爆破**敏感目录**

- **敏感文件可利用参数**
  - fuzz参数来达到Jsonp劫持以及XSS漏洞等等;
  - 越权验证信息
- FBypass 
  - SQL injection Bypass
  - Open redirect
  - XSS Fuzzer

## 常用工具

- 御剑: 界面化目录和文件扫描
- Dirsearch : 扫描模式和dirbuster是差不多
- Nikto : 作用在于目录的爆破
- wfuzz : 可以进行web应用暴力猜解，也支持对网站目录，登录信息，应用资源文件等的暴力猜解，还可以进行get及post参数的猜解，sql注入，xss漏洞的测试等。该工具所有功能都依赖于字典

**burp插件CO2**: Sqlmapper模块

