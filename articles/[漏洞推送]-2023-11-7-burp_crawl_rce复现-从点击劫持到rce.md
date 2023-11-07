#  burp_crawl_rce复现-从点击劫持到rce

原创 kkk mr [ 漏洞推送 ](javascript:void\(0\);)

**漏洞推送** ![]()

微信号 gh_d45bcadf18d7

功能介绍 专注于安全漏洞、威胁情报发掘。

____

___发表于_

收录于合集 #漏洞分析 5个

> 漏洞来源 https://hackerone.com/reports/1274695

## 漏洞环境

burpSuite 2021.7

https://portswigger-
cdn.net/burp/releases/download?product=pro&version=2021.7&type=jar

  

poc文件见h1链接

## 漏洞复现

添加一个扫描任务，用于启动burp内置的无头chrome

![]()

启动以后用 `python3 -m http.server`启动一个http服务，然后用chrome打开burp.html

打开以后会尝试 扫描本地的chrome的debug端口

![]()

扫描到了以后会创建一个iframe 地址是 `http://127.0.0.1:49576/`

该页面的内容为，多个A链接:

![]()

然后使用点击劫持，当点击CLICK ME以后实际上是点击 `http://127.0.0.1:49576/`下的 about:blank链接

![]()

iframe发生跳转,地址为

`https://chrome-devtools-
frontend.appspot.com/serve_file/@4bb19460e8d88c3446b360b0df8fd991fee49c0b/inspector.html?ws=127.0.0.1:49576/devtools/page/9D8411A3AA381D422364000736AE56D9&remoteFrontend=true`

这个地址中包含最重要的
`ws=127.0.0.1:49576/devtools/page/9D8411A3AA381D422364000736AE56D9`这个地址可用于chrome调试

那么这个时候，问题就是怎么拿到这个iframe中的地址了，因为这个同源策略，我们在top页面上只能拿到`http://127.0.0.1:49576/`这个地址，点击a连接跳转以后的`https://chrome-
devtools-frontend.appspot.com`这个地址我们是拿不到的

比如这个页面，我们在127上iframe到另一个域的网站，然后a链接跳转到7k7k，我们通过src拿到的还是127的地址

![]()

如果想要拿到真实的src地址，就要同源，我再iframe一个7k7k,在这个iframe下就能拿到真实的src了

![]()

于是攻击者利用了appspot.com下的一个dom
xss漏洞,新建一个iframe页面，然后把地址通过postmessage发送到top页面，得到了ws地址

`https://chrome-devtools-
frontend.appspot.com/serve_rev/@191797/devtools.html?remoteFrontendUrl=javascript:top.postMessage(top.frames[1].location.href,"*")`

拿到这个地址以后，通过`chrome-remote-interface`就可以操作浏览器的行为了。配置文件下载路径

![]()

然后用blob协议发起文件下载请求，就能够实现任意文件写入了

值得注意的是，这个作者实现mac rce的方式也是很有价值是通过，覆盖burp的`vmoptions`来实现的

![]()

给jvm设置极小的内存，burp会迅速内存耗尽，触发了`OnOutOfMemoryError`选项导致命令执行。

在这个案例中，一共使用了js扫描探测端口->点击劫持->dom xss->操纵浏览器实现任意文件写入->jvm
rce。攻击链相当复杂，虽然实战中可利用可能性不大，但是相当具有参考价值。

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

