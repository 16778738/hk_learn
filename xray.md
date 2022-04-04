1. 使用基础爬虫爬取并对爬虫爬取的链接进行漏洞扫描

   ```
   xray webscan --basic-crawler http://example.com --html-output vuln.html
   ```

2. 使用 HTTP 代理进行被动扫描

   ```
   xray webscan --listen 127.0.0.1:7777 --html-output proxy.html
   ```

   设置浏览器 http 代理为 `http://127.0.0.1:7777`，就可以自动分析代理流量并扫描。

   > 如需扫描 https 流量，请阅读下方文档 `抓取 https 流量` 部分

3. 只扫描单个 url，不使用爬虫

   ```
   xray webscan --url http://example.com/?a=b --html-output single-url.html
   ```

4. 手动指定本次运行的插件

   默认情况下，将会启用所有内置插件，可以使用下列命令指定本次扫描启用的插件。

   ```
   xray webscan --plugins cmd-injection,sqldet --url http://example.com
   xray webscan --plugins cmd-injection,sqldet --listen 127.0.0.1:7777
   ```

5. 指定插件输出

   可以指定将本次扫描的漏洞信息输出到某个文件中:

   ```
   xray webscan --url http://example.com/?a=b \
   --text-output result.txt --json-output result.json --html-output report.html
   ```