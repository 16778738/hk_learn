# CDN学习笔记

~~~~
CDN即内容分发网络，起初是为了提高网络的通讯效率，后被用于IP的隐藏技术。
~~~~

## CDN绕过方法

~~~markdown
1. dns解析 使用nslookup 网址 ip
2. 不同的ip地域多次ping
		有些CDN可能只对国内的IP加装了CDN流量分发，而没有对其他地域加CDN
3. 使用搜索引擎
		shoda，钟馗，qianx
4. rss订阅搜寻
5. zmap
		https://linux.cn/article-5860-1.html
6. 网站证书查询，whois挖掘全面域名相关信息
		网址查询参考： 
        真实IP查询：https://www.ipip.net/ip.html
        超级ping检测是否存在cdn：https://ping.aizhan.com/
        站长超级ping检测：http://ping.chinaz.com/
        网站数据挖掘参考：https://sitereport.netcraft.com/
        端口多种检测，IP查询参考：https://viewdns.info/
        cdn检测地域范围：http://www.17ce.com/
        指纹信息查询：http://finger.tidesec.com/

~~~

