# 问题

遇到了一个问题，就是同一个域名，我在两个网站上查到它的 IP 是不一样的：

![title](https://leanote.com/api/file/getImage?fileId=5dcc08c8ab644147030004ea)

![title](https://leanote.com/api/file/getImage?fileId=5dcbf345ab6441450a000467)

这是为什么呢？这是因为 CDN。


CDN 即 `content delivery network`，通过在网络各处放置节点服务器所构成的在现有的互联网基础之上的一层智能虚拟网络，CDN系统能够实时地根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。

开了CDN 之后，会智能匹配当地最近的节点来的，所以请求的实际 IP 不同。

# CDN 的配置

要检测 CDN 就要从 CDN 的配置说起，配置 CDN 一般只有两种方式：

**方法 1：**

是给域名设置一个 `cname` 类型的记录，让它指向 cdn 厂商提供的另一个域名，这种域名有前缀一般是看起来很随机那种。

**方法 2：**

把域名的 `NS` 记录指向 CDN 厂商的 DNS 服务器 IP。

Github 上面有一个开源项目，收集了所有常见的 cdn 厂商的 ip 范围与 cname 信息，用可于判断目标配置了具体哪个 CDN。 

或者有的 CDN 会改 http 响应头，能在响应头里面直接看出来。


# CDN 的检测

为什么我们需要检测 CDN？个人觉得一个比较常见的场景是：我们获取了一个批量入侵上千个网站的任务，第一步我们需要获取其真实 IP，在获取真实 IP 之后我们才能进行进一步的信息收集工作（如通过 socket 获取端口 banner）。但是这个前提是，我们需要获取域名对应的真实 IP。这就要求我们先判断哪些域名有 CDN 防护，那么我们需要进一步绕过 CDN 去获取真实 IP；哪些域名没有 CDN 防护，我们就可以直接获取真实 IP。也就是说，首先我们要按是否有 CDN 进行一个分类。

有人说，检测 CDN 很简单。多地 ping 不就行了，但是这种简单局限于较少的检测量。多地 ping 延迟高，需要等待较长的时间，如果有几千个域名，难道一个一个去多地 ping 吗？显然这样效率是不够高的。

所以对于 CDN 的检测，我们分为**单个域名的 CDN 检测**和**域名批量 CDN 检测**两种，使用不同的方法。

## 单个域名的 CDN 检测：
**方法 1：**

直接访问 ping 到的 IP，加了 CDN 之后，不能直接通过访问 IP 访问网站。

**方法 2：**

多地 ping，看得到的 IP 是不是一样的。不一样则可能有 CDN。

全球 ping 推荐网站：https://wepcc.com/（快、稳定）

**方法 3：**

是看 DNS 解析，一般有的 CNAME 的大多数是 CDN。



下面是几个根据 CNAME 记录判断 CDN 的例子，CNAME 绑定的域名会有 `cdn` 相关字符。

![title](https://leanote.com/api/file/getImage?fileId=5dcbf86eab6441470300047a)

![title](https://leanote.com/api/file/getImage?fileId=5dcbf959ab6441450a0004a1)

![title](https://leanote.com/api/file/getImage?fileId=5dcbf9ebab6441470300047e)


`注意：`以上3种方法请结合使用！！！比如这个网站 ipip.net，我这样看上去似乎没有 CDN：

![title](https://leanote.com/api/file/getImage?fileId=5dd2419bab64414b290013cb)

但是`多地 ping` 就会发现有 CDN：

![title](https://leanote.com/api/file/getImage?fileId=5dd241c1ab64414b290013cd)

所以说要结合多个方法，有一种方法判断出有 CDN 那就是有 CDN！！！




## 域名批量 CDN 检测：

通过脚本 [iscdn.py](https://github.com/al0ne/Vxscan/blob/master/lib/iscdn.py)。


此脚本来自于 Github 项目：[vxscan](https://github.com/al0ne/Vxscan)(Author: al0ne)，此文件依赖项目里的其他文件（mmdb 等）。



这个脚本的原理是利用 ASN。ASN 是自治系统号，相同组织的 ASN 是一样的，所以一个 CDN 厂商的 IP 段可能都在一个 ASN 里。

![title](https://leanote.com/api/file/getImage?fileId=5dd237b7ab6441492c0012f7)


在此脚本中，作者搜集了常见 CDN 厂商的 ASN 号：

![title](https://leanote.com/api/file/getImage?fileId=5dd238e0ab64414b29001352)

以及国内外常见的 CDN 段：

![title](https://leanote.com/api/file/getImage?fileId=5dd23967ab64414b29001359)

判断逻辑如下：

1. 解析域名，获取解析之后的 IP；
2. 判断此 IP 是否命中国内外常见 CDN 段，如果命中，说明有 CDN；
3. 对此 IP 查询其对应的 ASN 号（通过 `response = reader.asn(host)` 这一句代码）。如果获取的 ASN 号命中了上面的国内外常见 CDN 厂商的 ASN 号，说明有 CDN；
4. 如果通过这两次判断都没有命中，那么极有可能说明是真实 IP。



![title](https://leanote.com/api/file/getImage?fileId=5dd22ce7ab64414b290012d8)

为什么要通过两次判断呢？常见 CDN 段是从一些网站官网上公开的信息获取的，准确率高，但是涵盖不是很全。ASN 号是从 提供域名就可以找到ASN的工具 —— http://bgp.he.net/ 这个网站逐个收集的，覆盖面广，但是容易误报。通过两次查询去命中，主要是尽可能的判断出加了 CDN 的情况。

总之，可以利用此脚本，批量的去做第一步：域名分类的工作，找出哪些网站是加了 CDN 防护的，那么下一步我们就会用不一样的接口去批量获取其真实 IP。

另外为什么批量通过`多地 ping` 去获取 CDN 判断结果呢？主要是效率问题，`多地 ping` 需要等待的时间长，效率低。再加上批量，所以判断效率肯定是比不上通过 ASN 号静态比对的。

# 绕过 CDN 寻找真实 IP

建议直接放弃。


# 绕过 CDN 之后判断得到的真实 IP 是不是正确的

找到真实 IP 后 先访问 IP 看看和原站是否一样。

或者绑定 hosts（在 hosts 文件里面加入「真实IP」和域名），再去访问那个域名，看看能不能正确访问到。




------------------------

参考文档：

1. [域名信息](https://websec.readthedocs.io/zh/latest/info/domain.html)，Web安全学习笔记

