# 前言

在这篇文章：[谈一谈如何在 Python 开发中拒绝 SSRF 漏洞](https://juejin.im/entry/57f0ff2a2e958a00555268b7) 里面提出：

![title](https://leanote.com/api/file/getImage?fileId=5da45c88ab6441691e000a9b)

这种处理流程真的能避免 SSRF 吗？

可以避免的有：

1. 直接访问内网 IP
2. 302 跳转
3. xip.io/xip.name 及短链接变换等 URL 变形
4. 畸形 URL
5. iframe 攻击
6. IP 进制转换

![title](https://leanote.com/api/file/getImage?fileId=5da5229fab64412aac000105)
......

看起来似乎很完美，但是还有一种攻击可以绕过此种处理流程，就是 DNS Rebinding Attack。

# DNS Rebinding Attack（DNS 重绑定攻击）

**攻击原理：**

一般进行 ssrf 防御的模式如下：

![title](https://leanote.com/api/file/getImage?fileId=5da52b9cab64412ca4000190)



1. 获取到输入的URL，从该URL中提取host
2. 对该host进行DNS解析，获取到解析的IP
3. 检测该IP是否是合法的，比如是否是私有IP等
4. 如果IP检测为合法的，则进入curl的阶段发包

观察到，在这个流程中，一共进行了两次DNS解析：第一次是对URL的host进行DNS解析，第二次是使用CURL发包的时候进行解析。这两次DNS解析是有时间差的，我们可以使用这个时间差进行绕过。

事件差对应的DNS中的机制是`TTL`。TTL表示DNS里面域名和IP绑定关系的Cache在DNS上存活的最长时间。即请求了域名与iP的关系后，请求方会缓存这个关系，缓存保持的时间就是TTL。而缓存失效后就会删除，这时候如果重新访问域名指定的IP的话会重新建立匹配关系及cache。

在上面的流程中，如果在DNS第二次解析的时候，我们能够更换URL对应的IP，那么在TTL之后、缓存失效之后，重新访问此URL的话，就能获取被更换后的IP。如果我们把第一次解析的IP设为合法IP，就能绕过host合法性检查了；把第二次解析的IP设为内网IP，就达到了SSRF访问内网的目的。

在这个过程中，对于浏览器来说，整个过程访问的都是同一域名，所以认为是安全的。这就会导致绕过。

所以总结一下：

DNS 重绑定攻击的原理是：利用服务器两次解析同一域名的短暂间隙，更换域名背后的ip达到突破同源策略或过waf进行ssrf的目的。


**时间窗口问题：**

TTL 最理想的设置是0，即在第一次解析之后，立马换位我们想要访问的内网IP。

但是现实是：现在国内购买的域名大都无法直接将TTL设置为0，例如阿里云的域名，最小的TTL是10分钟。而某些国外的域名可以设置TTL=0。

这种情况其实问题不大，在某些情况下，我们甚至可以对同一个域名设置两个A记录（一个内网、一个外网），这样会random访问两条记录中的一个。这样就会变成有概率的成功，不是完全成功而已。

在实战中我们还可能遇到 DNS 缓存的问题。即使我们在前面实现的时候设置了TTL为0，但是有些公共DNS服务器，比如114.114.114.114还是会把记录进行缓存，完全不按照标准协议来，遇到这种情况是无解的。但是8.8.8.8是严格按照DNS协议去管理缓存的，如果设置TTL为0，则不会进行缓存。

这种情况只能看命了。


**DNS重绑定攻击步骤：**


恶意网站：http://attacker.com用到很短的生存时间（TTL），比如60s来记录，页面上包含一个恶意的Javascript有效载荷，利用著名的WebRTC内部IP漏洞获取本机的内部IP地址，对内网进行B&O设备扫描。

可以自动创建并删除图像标记来寻找IP地址包含/images/BO_processing_grey.gif，找到扫描结束，DNS重绑定开始

例如找到地址为：192.168.1.10，我们把这个ip发给攻击者，在客户端的JavaScript负载等待超过一分钟，一分钟之后，在试图得到http://attacker.com/1000/Bo_network_settings.asp。

DNS此时过期，我们将http://attacker.com解析到了192.168.1.10，此时仍然认为是同源。






1. 在网站 SSRF 漏洞处访问精心构造的域名。网站第一次解析域名，解析域名获取 IP 地址 A；
2. 经过网站后端服务器的检查，判定此IP为合法IP。
3. 网站获取URL对应的资源（在一次网络请求中，先根据域名服务器获取IP地址，再向IP地址请求资源），第二次解析域名。此时已经过了ttl的时间，解析记录缓存IP被删除。第二次解析到的域名为被修改后的 IP 即为内网IP B；
4. 攻击者访问到了内网 IP。


DNS rebinding被广泛用于bypass同源策略，绕过ssrf的过滤等等。

# DNS Rebinding 攻击演示

Bendawang 师傅总结了三种实现方法：
1. 实现方法一：特定域名实现
2. 实现方法二：简单粗暴的两条A记录
3. 实现方法三：自建DNS服务器

我比较喜欢偷懒，因为 ceye.io 上面已经有 DNS Rebinding 功能了。比如我随便把两条解析记录设置为：
![title](https://leanote.com/api/file/getImage?fileId=5da5392aab64412aac00020f)

ceye说会 “the dns answer section will randomly return one of them”，有点慌。

![title](https://leanote.com/api/file/getImage?fileId=5da53a28ab64412ca400022a)

![title](https://leanote.com/api/file/getImage?fileId=5da53cfaab64412aac00025b)

不是稳定的两个IP交替，这种情况下就是撞运气了。如果能达到第一次访问的是144.x.x.x这个外网地址，第二次访问的是 127.0.0.1，成功的希望就大了一些。

个人不是很建议使用ceye的这个DNS Rebinding，尝试中单位时间内好多次都访问了同一个IP。感觉这个切换率还不如同域名绑定两个 ANAME 呢。

另一个利用是否成功的因素取决于目标网站的业务逻辑中的 TTL：

 - Java默认不存在被DNS Rebinding绕过风险（TTL默认为10）  
 - PHP默认会被DNS Rebinding绕过
 - Linux默认不会进行DNS缓存

还是换成**自建DNS服务器**的方式：

在 snowming.space 上操作（狗爹上买的域名）：

![title](https://leanote.com/api/file/getImage?fileId=5da53f0dab64412aac00026a)

在snowming.me上操作（狗爹上买的域名）：

![title](https://leanote.com/api/file/getImage?fileId=5da53fb1ab64412ca4000282)

在144.168.57.70这台服务器上开启dns服务:



``` python
from twisted.internet import reactor, defer
from twisted.names import client, dns, error, server

record={}

class DynamicResolver(object):

    def _doDynamicResponse(self, query):
        name = query.name.name

        if name not in record or record[name]<1:
            # 随意一个 IP，绕过检查即可
            ip="104.160.43.154"
        else:
            ip="127.0.0.1"

        if name not in record:
            record[name]=0
        record[name]+=1

        print name+" ===> "+ip

        answer = dns.RRHeader(
            name=name,
            type=dns.A,
            cls=dns.IN,
            ttl=0,
            payload=dns.Record_A(address=b'%s'%ip,ttl=0)
        )
        answers = [answer]
        authority = []
        additional = []
        return answers, authority, additional

    def query(self, query, timeout=None):
        return defer.succeed(self._doDynamicResponse(query))

def main():
    factory = server.DNSServerFactory(
        clients=[DynamicResolver(), client.Resolver(resolv='/etc/resolv.conf')]
    )

    protocol = dns.DNSDatagramProtocol(controller=factory)
    reactor.listenUDP(53, protocol)
    reactor.run()



if __name__ == '__main__':
    raise SystemExit(main())

```

注1：此脚本可以做到第一次请求解析记录时返回第一个外网 IP，第二次请求解析记录的时候返回一个第二个内网 IP。
注2：里面`ttl`设为了0。

在终端中运行：

```
pip install twisted
sudo python dns_server.py 
```

![title](https://leanote.com/api/file/getImage?fileId=5da54b7dab64412ca40002a9)


![title](https://leanote.com/api/file/getImage?fileId=5da54b0aab64412ca40002a6)

`注：`有时候也会两次解析到第一个外网IP，后面都解析到第二个内网或主机IP。多尝试几次就好。


展示一下一个全盲 SSRF 访问此域名之后的解析记录：

![title](https://leanote.com/api/file/getImage?fileId=5da55114ab64412aac00057c)

![title](https://leanote.com/api/file/getImage?fileId=5da55119ab64412aac00057d)

这说明此全盲 SSRF 无上面那种SSRF防御流程。直接访问了 URL 获取资源。这中间只经历了一次 DNS 解析。

我这里没有具有这种防御机制的 SSRF 漏洞，遇到了再看。

---------

参考链接（排名有先后）：
[1] [关于DNS-rebinding的总结](http://www.bendawang.site/2017/05/31/%E5%85%B3%E4%BA%8EDNS-rebinding%E7%9A%84%E6%80%BB%E7%BB%93/)，Bendawang's Website，Bendawang，2017年5月31日
[2] [DNS rebinding](https://cl0und.github.io/2018/01/28/DNS%20rebinding/)，cL0und 博客，cL0und，2018年1月28日
[3] [Use DNS Rebinding to Bypass SSRF in Java](https://mp.weixin.qq.com/s/545el33HNI0rVi2BGVP5_Q)，美丽联合集团安全应急响应中心，JoyChou@美联安全，2017年9月11日
[4] [通过DNS重新绑定绕过过度同源策略攻击传输分析](https://www.anquanke.com/post/id/97366)，安全客，2018年2月6日
[5] [DNS REBINDING](https://zhuanlan.zhihu.com/p/25604072)，知乎，adrain，2017年3月7日
[6] [DNS Rebinding技术绕过SSRF/代理IP限制](https://blog.csdn.net/u011721501/article/details/54667714)，CSDN，隐形人真忙，2017年1月22日 
[7] [Windows Server 部署DNS服务](http://www.bubuko.com/infodetail-2710888.html)，2018年8月2日
[8] [DNS Rebinding 域名重新绑定攻击技术](https://www.freebuf.com/column/194861.html)，FREEBUF，漏斗社区，2019年1月23日 

