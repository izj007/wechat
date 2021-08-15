# 前言
IP 变形主要是为了 bypass，在 SSRF 中，我们可以通过 IP 变形来避免命中黑名单。

# 方法1：省略 0
**x.x = x.0.0.x**

![title](https://leanote.com/api/file/getImage?fileId=5da496cfab64416721000e0e)

**x.x.x = x.x.0.x**

![title](https://leanote.com/api/file/getImage?fileId=5da4986eab64416721000e2c)

# 方法2：xip.io（xip.name）
![title](https://leanote.com/api/file/getImage?fileId=5da4a5efab6441691e000ec4)

# 方法3：短链接

通过短链接生成器生成url对应的短链接。

比如：https://tool.chinaz.com/tools/dwz.aspx

转换本机地址:

![title](https://leanote.com/api/file/getImage?fileId=5da4aa7cab6441691e000ecd)

转换内网地址：

![title](https://leanote.com/api/file/getImage?fileId=5da4aa20ab6441691e000ecb)

# 方法4：进制转换

通过 [IPFuscator](https://github.com/vysecurity/IPFuscator) 这个工具可实现进制转换。进制转换包括八进制、十进制、十六进制、混合进制。进制转换后的结果跟原四位点分十进制IP完全等同。

![title](https://leanote.com/api/file/getImage?fileId=5da4a32dab64416721000e54)



# 方法5：畸形 URL

此种方法可以 bypass ssrf 的白名单过滤机制。

http://www.baidu.com@zhihu.com = http://zhihu.com

http://www.baidu.com@127.0.0.1/ = http://127.0.0.1



# 方法6：punycode

> IDN（英语：Internationalized Domain Name，缩写：IDN）即为国际化域名，又称特殊字符域名，是指部分或完全使用特殊的文字或字母组成的互联网域名。包括法语、阿拉伯语、中文、斯拉夫语、泰米尔语、希伯来语或拉丁字母等非英文字母，这些文字经多字节万国码编译而成。在域名系统中，国际化域名使用Punycode转写并以美国信息交换标准代码（ASCII）字符串储存。


由于cURL也支持IDN，可以进行Punycode编码，所以我们也可以用来绕过日常的 ssrf 等漏洞的利用限制。


![title](https://leanote.com/api/file/getImage?fileId=5da4bf44ab64412ca4000025)

punycode解码：①②⑦ => 127

punycode 编码工具网站：https://tw.piliapp.com/symbol/

----

参考链接：


[1] [那些不为人知的ip](http://www.luteam.com/?p=211)
[2] [Punycode](https://www.lz1y.cn/2018/07/18/Punycode/)，垃圾桶，LZ1Y，2018年7月18日