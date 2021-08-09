##  对blackhat和defcon一些议题的小结

原创 w8ay [ Hacking就是好玩 ](javascript:void\(0\);)

**Hacking就是好玩** ![]()

微信号 gh_aed6cfc863ed

功能介绍 写安全工具的同时，写写字解闷~

____

__

收录于话题

对blackhat和defcon一些感兴趣的议题做了一些小结。

## 静态分析

blackhat里面一个静态分析引擎的议题。

https://www.blackhat.com/us-21/briefings/schedule/index.html#do-you-speak-my-
language-make-static-analysis-engines-understand-each-other-22797

之前研究过语义和污点分析，所以想看看。但这个议题总体来说我没有我想找的东西，我想看看污点分析具体是怎样做的，使用什么算法，怎样更有效率。

作者的议题主要就是将了如果一个应用是php，一个应用是python，两者相互调用，怎样分析呢，结果就是缝合怪，php的分析使用php的分析器开一个rpc，python的分析用python的分析开个prc，定义好数据传输格式就行。

我在想使用像codeql一样，定义一个通用的识别规则不就好了吗。

![](https://gitee.com/fuli009/images/raw/master/public/20210809102429.png)image-20210806164212636

可能作者想的是针对甲方的场景，我想的是针对安全研究员挖洞的场景吧。

作者给了几个开源静态分析工具的实现，哪天看一下学习学习 - =

github.com/facebook/mariana-trench

github.com/facebook/pyre-check

github.com/facebook/sapp

## Exchange Server新的攻击面

作者介绍了exchange server的一些历史和总体的架构变化以及验证的方式，然后介绍了一些攻击链，漏洞很精彩。

  1. ProxyLogon 是通过一个ssrf访问到授权后的后端，操作授权的后端可以写入任意文件。

  2. ProxyOracle: 作者发现了可以用`Padding Oracle Attack`来解密cookie中加密的用户数据，又发现了一个xss，但是exchange server有httponly验证，可以通过ProxyLogon的ssrf来绕过httponly验证，把cookie发送到恶意服务器上，再到服务器上使用Padding Oracle Attack进行解密。意味着目标只需要点击一个链接就能获取到目标邮箱的用户和密码。

  3. ProxyShell：在Pwn2Own 2021上展示的利用链来接管 Exchange 并获得 200,000 美元的赏金。

    1. 首先是一个路径转换的逻辑错误再次造成ssrf，接下来便是寻找如何将ssrf变成命令执行。

    2. 之前的攻击链已经被修复，于是找到了`Exchange PowerShell Remoting`，Exchange PowerShell Remoting是一个命令行界面，用于 Exchange 任务的自动化，但是它需要一个mailbox。

    3. 使用特殊的X-Rps-CAT Access-Token可以模拟任意用户，此时就能执行powershell命令了。

    4. 利用New-MailboxExportRequest导出shell
        
                New-MailboxExportRequest –Mailbox orange@orange.local  
        –FilePath \\127.0.0.1\C$\path\to\shell.aspx  
        

New-MailboxExportRequest导出的是 Outlook Personal Folders (PST)
format,通过学习这个文件格式构造特殊的文件，在导出时就可以变成webshell了。

文章地址：https://blog.orange.tw/2021/08/proxylogon-a-new-attack-surface-on-ms-
exchange-part-1.html

视频地址：https://www.youtube.com/watch?v=5mqid-7zp8k&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=16

## SAST

DEF CON 29 - Rotem Bar - Abusing SAST tools When scanners do more than just
scanning

https://www.youtube.com/watch?v=Jl-
CU6G4Ofc&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=7

作者梳理了静态分析的一些手法，并且提出了一些sast扫描器的错误配置，可能造成执行命令，这个特性可能会被利用。

## Hacking IDE

https://www.youtube.com/watch?v=pzqu_qaoNuY&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=33

DEF CON 29 - David Dworken - Worming through IDEs

作者说在新冠期间本来有几个月的空闲可以去休假，但是他花了几个月用来发现一些IDE的漏洞。使用`strace`命令可以监控程序运行时发出的系统调用，通过这可以找到IDE启动时调用的命令。

作者介绍了VSCode、VS、IntelliJ、Vim、甚至记事本都可能存在的命令执行。

不止本地ide，还有在线ide，通过设定一些特殊的配置，就可以在远程IDE中执行命令。

作者提出了一个构想，如果某个开发人员代码中被包含了这些漏洞，所有打开这个开发者代码的人会再次中毒，以此进行蠕虫传播并自动感染！

作者说要把poc上传到github，但是搜了下没看到 - =

## 用Golang编写恶意软件

DEF CON 29 - Ben Kurtz - Offensive Golang Bonanza: Writing Golang Malware

https://www.youtube.com/watch?v=3RQb05ITSyk&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=39

介绍了golang的一些作为红队使用语言的优势，一些用golang的库(这些库就像是为恶意软件量身定制)，可以作为学习go的开始。

像py2exe和jar2exe，因为没有流行的软件，它们生成的工具很容易被杀毒针对，而golang编写的软件像docker等，让杀软无法直接查杀golang语言本身的特征，这更方便编写恶意软件。

议题主要介绍的是题主自己用golang实现的一系列红队工具，外加一些其他的补充。

PE/ELF/Mach-O 文件解析

  * https://github.com/binject/debug

任意平台进行任意系统调用

  * https://github.com/awgh/cppgo

WMI 远程执行

  * https://github.com/c-sto/goWMIExec

钓鱼工具包

  * https://github.com/gophish/gophish

爆破

  * https://github.com/OJ/gobuster

    * 爆破子域名 vhost 目录/文件

Go混淆

  * https://github.com/burrowers/garble

帮助通过UDP、TLS、HTTPS、DNS、S3带出数据

  * https://github.com/awgh/ratnet

Go写的VPN

  * https://github.com/WireGuard

Go实现的虚拟文件系统

  * https://github.com/capnspacehook/pandorasbox

任意系统中加载库文件

  * https://github.com/Binject/universal

  * Windows用的反射加载

  * 库加载是通过「内存」的

Donut

  * https://github.com/TheWover/donut

  * 任意文件(exe dll .net vbs js)转shellcode

  * donut Go包装器

    * https://github.com/Binject/go-donut

payload 免杀框架

  * https://github.com/optiv/ScareCrow

地狱之门Go实现

  * https://github.com/C-Sto/BananaPhone

  * Windows 原生调用syscall

天堂之门Go实现

  * https://github.com/aus/gopherheaven

  * 从32位代码中调用64位代码

多平台的stager dropper实现

  * https://github.com/gen0cide/gscript

## Chrome扩展的攻击和持久化

DEF CON 29 - Barak Sternberg - Extension Land: Exploits and Rootkits in Your
Browser Extensions

作者讲的很详细，总结了几种chrome扩展和网页通信的api的攻击面，根据一些实例讲述了一些存在漏洞的扩展以及在应用中注入恶意的js达到持久化。

  * 一种学术翻译扩展，它会从本地127.0.0.1的一个端口上获取js内容，所以可以根据另一个只有tcp权限的app控制这个app。

  * 一种和谷歌学术交互的扩展，从chrome.storage获取链接，而chrome.storage可以被控制

  * vimium：作者演示了vimium漏洞，通过对vimium攻击，可以注入js到其他任意网站。

chrome扩展持久化

chorme对扩展有严格的控制，会通过计算hash校验，但是如果开启了开发者模式，就不会有这些验证，并且开启开发者模式后，chrome也不会有任何提示。

通过后缀`--load-extensiton`即可加载任意扩展。

https://www.youtube.com/watch?v=PpSftQuCEDw&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=40

## DNS as Service 测试

对云厂商的DNS as Service服务进行测试，在dns server中添加dns
server的记录到自己的ip，结果意外发现有大量流量经过，最后发现这是Windows的一种“特性”。

作者还提到了通过这些数据，进行的一些有趣的探索。

DEF CON 29 - Shir Tamari, Ami Luttwak - New class of DNS Vulns Affecting DNS-
as-Service Platforms

https://www.youtube.com/watch?v=72uzIZPyVjI&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=4

## UFO

DEF CON 29 - Richard Thieme AKA neuralcowboy - UFOs: Misinformation, Disinfo,
and the Basic Truth

关于ufo，视频讲了一些事件，以及告诉我们要了解的话就要去阅读文件，和当事者对话以及列举了一些人的语录等等。但也都是听他再说。

https://www.youtube.com/watch?v=mExktWB0qz4&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=8

## 逃避Linux系统调用监控

DEF CON 29 - Rex Guo, Junyuan Zeng - Phantom Attack: Evading System Call
Monitoring

https://www.youtube.com/watch?v=yaAdM8pWKG8&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=11

作者提出了两种方法逃避系统调用监控，TOCTOU和语言混淆。

  * TOCTOU

    * 一般的系统调用监控有两个步骤`sys_enter`、`sys_exit`，这两个断点放置的位置不同，利用的方式也不同。

    * 在内核调用之前被hook，那么可以在之后修改缓冲区的文本绕过

    * sys_exit hook的位置错误，也能通过内核竞争或执行UserFaultfd syscall去提前结束

  * 语义混淆

    * 通过lnk文件绕过黑名单

## 用eBPF创建安全、稳定、隐蔽和可移植的内核 rootkit

DEF CON 29 - PatH - Warping Reality: Creating and Countering the Next
Generation of Linux Rootkits

https://www.youtube.com/watch?v=g6SKWT7sROQ&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI&index=14

eBPF是伯克利数据包过滤器，平时用于过滤和修改网络流量，内核输入状态等。因为eBPF是linux内核成员之一，通过eBPF编写内核rootkit代码十分稳定，因为只需要写一些eBPF代码，不容易崩溃，通过网络流量和程序输入状态之行，也十分隐蔽。

作者讲的eBPF利用十分详细，最后也给出了源代码：https://github.com/pathtofile/bad-bpf

视频推荐一看。

Ps: eBPF For Windows也在被官方发展中，虽然目前只有网络方面。

## iis 原生恶意程序剖析

作者总结了历史的iis后门，以及对iis模块进行的逆向分析，作者可能是想做一个通过http就访问的iis后门，但是从ppt中没看到它们是怎么做的。

https://www.blackhat.com/us-21/briefings/schedule/index.html#anatomy-of-
native-iis-malware-23395

## 免杀 - 通过 多进程ROP + NTFS事务免杀

将单个组件的代码分成多个块，由注入到多个进程中的模拟器运行。

https://www.blackhat.com/us-21/briefings/schedule/index.html#rope-bypassing-
behavioral-detection-of-malware-with-distributed-rop-driven-execution-23051

### 通过以下两个手法

代码复用：避免rwx内存空间，使用pwn手段中ROP gadgets的手段复用代码。

  * 有效编码payload

  * 绕过WDEG

NTFS事务

  * 不可检查的隐蔽通道

  * payload共享+通信

### loader过程

  1. 选择一个进程

  2. 创建ROP-TxF文件

  3. 清除，填充chains & metadata

  4. 注入启动(Bootstrap)代码

### Bootstrap代码

  1. 加载ROP-TxF文件

  2. 执行ROP代码

  3. 可以和其他进程配合(ROP代码可以从多个进程中读取)

### 优点

✓ 无需分配/修改可执行内存  
✓ AV/EDR 的内存检查更难（ROP 增加了间接性）  
✓ 代码和数据的单一共享介质  
✓ 符合 ACG 和 CIG  
✓ 无文件

## CVE-2021-1748：从客户端 XSS 到弹计算器

https://www.blackhat.com/us-21/briefings/schedule/index.html#hack-different-
pwning-ios--with-generation-z-bugz-23002

CVE-2021-1748：从客户端 XSS 到弹计算器  
[https://mp.weixin.qq.com/s/g1kmGkp2B5QyQEUhdXOG8Q](https://mp.weixin.qq.com/s?__biz=Mzk0NDE3MTkzNQ==&mid=2247483975&idx=1&sn=8ee4f4036bf7d1b314df3789cb918779&scene=21#wechat_redirect)

输入向量是`URL scheme`、 通过反编译找到url scheme找到处理逻辑，通过xss可以执行内置的js类，从而可以获取到很多信息

>   * iTunes.systemVersion() 获取系统版本号
>
>   * iTunes.primaryAccount?.identifier 获取 App Store 账号的邮箱
>
>   * iTunes.primaryiCloudAccount?.identifier 获取 iCloud 账号的邮箱
>
>   * iTunes.diskSpaceAvailable() 存储可用空间
>
>   * iTunes.telephony 电话号码、运营商等信息
>
>   * iTunes.installedSoftwareApplications 所有已安装的 app 信息
>
>

## Chrome漏洞挖掘

>
> 错误很少是唯一的。系统规模不断扩大的软件通常涉及多个团队，负责开发众多功能。考虑到代码库的复杂性，在整个代码库中的许多地方可能存在与类似代码模式共享的错误的可能性很高。
>
> 在本次演讲中，我们以 Chrome 为例，介绍如何根据历史漏洞发现新的漏洞。我们将介绍几种在 Chrome
> 中容易受到攻击的代码模式，从浅到深。对于每一个模式，我们都会从一些经典的bug中进行总结，进行详细的描述，不仅介绍了寻找相似bug的基本工作流程，还介绍了调整和细化模式以发现与原始bug不同的新bug的方法。通过这些模式，我们最终在
> Chrome 中找到了 24 个漏洞并获得了 11 个 CVE。最后，我们将详细介绍如何利用我们在 2020 天府杯网络安全大赛中使用的其中一个逃出
> Chrome 沙箱，这是自 2015 年以来首次在公开赛中以沙箱逃生 Chrome 类别获胜。

从一个漏洞寻找更多相似漏洞，先看历史漏洞，然后基于历史规则用Codeql寻找更多漏洞

https://www.blackhat.com/us-21/briefings/schedule/index.html#put-in-one-bug-
and-pop-out-more-an-effective-way-of-bug-hunting-in-chrome-22855

## 其他

以上列举的都是我看过的一些议题的小结，还有很多有趣并且内容充实的议题没有列举，例如一些二进制漏洞的挖掘，bootloader的探索等等，有兴趣可以自行去官网看看。

blackhat 议题：https://www.blackhat.com/us-21/briefings/schedule/index.html

defcon：https://www.youtube.com/watch?v=VxNi5pVDZU0&list=PL9fPq3eQfaaBUD1zVxJWJmX86A6d0isBI

  

## 广告

插播一个广告，我也开始做知识星球了，侧重点在于对各类黑客工具的原理以及关于一些自动化扫描的研究，创立知识星球的目的是想促使我继续研究以及将研究转换成文档。知识星球也会不定时发布我平时研究比较好玩的东西当作作业。

加入后三天内可以全额退款，可以先加入看看呢。

公众号回复 知识星球 即可获得获得进入的二维码和8折优惠券(前100名有效)。

![]()

w8ay

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

对blackhat和defcon一些议题的小结

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

