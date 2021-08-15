SSRF的利用不难，但是我个人感觉它很少见。在乌云看了结界师的SSRF记录，看到他各种内网漫游各大互联网公司内网，真的很羡慕。

也看了一些 lijiejie 大神的。后面根据结界师的思路，找wap这种存在转码的网站，找了一圈，但是基本没找到。又找存在 url 跳转或者 redirect 的地方，其中最像的就是这个网站：https://illinois.edu/?url= ，但是我用 Burp Collaborator client 试了又试，就是没记录啊.....
于是继续找，一次又一次被谷歌判定为机器人，中间尝试了无数网站，直到页面开的太多失去响应。但是在这个时候，我无意中查看了一下 Burp Collaborator client 的记录，居然看到了结果！

![title](https://leanote.com/api/file/getImage?fileId=5d9f137aab64415c47000e1d)

于是又回去在浏览器历史纪录里面根据 Collaborator Server 的域名查找，终于找到了那个存在 SSRF 的网站！

在当时，我没有特别注意这个网站，因为它的页面回显是：
![title](https://leanote.com/api/file/getImage?fileId=5d9f13e9ab64415a49000f0d)

# 漏洞证明
该漏洞点是这样的：
![title](https://leanote.com/api/file/getImage?fileId=5d9f15c9ab64415a49000f12)

把furl后面的 url 换成我的 Burp Collaborator client，可以收到回显：
![title](https://leanote.com/api/file/getImage?fileId=5d9f16c6ab64415c47000e2a)

或者使用 nc 进行验证：
![title](https://leanote.com/api/file/getImage?fileId=5d9f1781ab64415c47000e2c)

顺便得知了对面的机器是Windows。

有人说我这不是 SSRF，下图是存在此SSRF漏洞的网站IP和访问Burp Colleborator Server的IP的对比，完全可以说明的确是从该网站服务器发出的请求。

![title](https://leanote.com/api/file/getImage?fileId=5d9f40e0ab64415c47000f18)
# 漏洞分析
我也很想内网漫游，但是做不到。因为这个洞真的非常鸡肋。

本来我以为它很有用，因为这个ssrf最开始的漏洞点，也就是这个：
![title](https://leanote.com/api/file/getImage?fileId=5d9f15c9ab64415a49000f12)

后面的url是不能直接进入的，即这个资源不能直接获取，因为那是一个学校的系统，需要登陆VPN才能进入系统。看起来，这个ssrf可以帮助我们绕过身份验证通往第三方系统。

但是其实这个洞用处不大，因为尝试对127.0.0.1进行端口探测，过程中发现返回的数据包都是一样的，就是“系统出错，请稍后重试”这一句话。所以从返回包的长度是无法判断出端口是否开启的。另外响应码也一直是200。响应码也无法看出来什么。

`注：` file协议是搞不出东西的，因为根本没回显，反正就是那一句“系统出错，请稍后重试”报错。



## 根据端口响应速度利用全盲 ssrf 漏洞


- 探测本机开放端口：

![title](https://leanote.com/api/file/getImage?fileId=5db062caab6441319500085b)
![title](https://leanote.com/api/file/getImage?fileId=5db062b1ab64413390000838)

- 探测本机未开放端口

![title](https://leanote.com/api/file/getImage?fileId=5db06303ab6441339000083f)
![title](https://leanote.com/api/file/getImage?fileId=5db06316ab64413195000862)

![title](https://leanote.com/api/file/getImage?fileId=5db06327ab6441339000084c)
![title](https://leanote.com/api/file/getImage?fileId=5db06330ab64413195000863)

可以看出来，响应时间存在很大差异，以此可以看出端口的开放。比如，本机开放了80端口，没有开放端口1、2。

但是通过此漏洞无法探知其他主机的存活情况：

随便构造了两个内网IP进行探测:

![title](https://leanote.com/api/file/getImage?fileId=5db063bfab64413390000854)
![title](https://leanote.com/api/file/getImage?fileId=5db063c7ab64413195000869)


![title](https://leanote.com/api/file/getImage?fileId=5db063d0ab6441319500086b)
![title](https://leanote.com/api/file/getImage?fileId=5db063dbab6441319500086c)


然后试了找到的此资产的一个绝对存活的主机：

![title](https://leanote.com/api/file/getImage?fileId=5db06588ab6441339000086c)

响应时间没有差别。

所以此漏洞的利用也就止步于此了。

虽然，我试了用lijiejie查找子域名，被我找出来一个内网网段：
![title](https://leanote.com/api/file/getImage?fileId=5d9f3fbeab64415a49001009)

尝试去ssrf但是没有回显。洞本身太鸡肋了，没法用。

在我看的所有使用ssrf进行内网漫游的漏洞报告中，最少最少，响应码也要有变化。所以我这个可以放弃了。

另外在在子域名结果中，随便一番探测，发现这个网站居然使用了创宇盾。没想到现在的普通学院安全意识还是挺高的。
![title](https://leanote.com/api/file/getImage?fileId=5d9f42bfab64415a4900101e)

这个洞就只能到这里了，希望下次可以挖出可以内网看看的 ssrf。



