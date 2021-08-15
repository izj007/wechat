# 0x01 什么是外部 C2？
C2 就是 Command & Control Server 的简称，也就是命令与控制服务器。

如下图是 C2 的大致通信模型。实现 C2 通信有时候很难，因为出口防火墙规则限制或进程限制。

![title](https://leanote.com/api/file/getImage?fileId=5e258af4ab64411ab6007467) 


Cobalt Strike 的团队服务器其实也是一种 C2 服务器。CS 的客户端相当于上面的「客户端」，CS 的团队服务器相当于上面的「攻击者的C2」。另外 CS 中，团队服务器会将要执行 的任何动作都以计划任务列表的形式进行管理，也就是说，在不考虑通信协议的前提下，CS 中的 C2 通信流程大概为：

1. 通过 CS 客户端发送到团队服务器的任何动作都会被弄成计划任务的形式依次排队（这也就是 beacon 内置的那个 job 命令存在的实际意义）；
2. 而后等着目标机器上的负载（payload）来下载这些计划任务列表中的具体指令去目标机器上执行；
3. 随后再依次把执行完的结果回传给 CS 团队服务器，团队服务器再回显至 CS 客户端。
 

现在的 Cobalt Strike （4.0版本之前）中，比较常用的 C2 通信方式是使用反向 shell 和反向 HTTP C2 通道。然而随着时间和防御水平的提高，这种「传统方法」势必会越来越难以生效。


![title](https://leanote.com/api/file/getImage?fileId=5e2597eeab64411ab600790c)

在这种情况下，势必需要一些实现 C2 通信的替代方法，<u>外部 C2</u> 就是这样的一种方法。

在 Cobalt Strike 4.0 中，对监听器类型做了扩充，直接加入了外部 C2 的 Payload 选项。

![title](https://leanote.com/api/file/getImage?fileId=5e256632ab64411ab600671d)

但其实，外部 C2 并非 CS 4.0 才有。之前的 Cobalt Strike 版本中（外部 C2 是自 Cobalt Strike 3.6 引入的功能），也一直有提供外部 C2 接口、允许第三方程序充当 Cobalt Strike 与其 Beacon payload 之间的通信层。只是没有直接在监听器这里加入这种 Payload 选项。

外部 C2 也不仅仅是 Cobalt Strike 才有，一些框架都会提供外部 C2 的方法，比如 MSF。


总之，我们看到了未来的方向：基于但不限于框架、使用多种方法、拓展协议实现 C2 通信。

# 0x02 外部 C2 的通信模型

外部 C2 系统是用于实现 C2 通信的一种规范。那么外部 C2 是如何实现 C2 通信的呢？

上面已经谈到，CS 的原始通信架构是这样的：

![title](https://leanote.com/api/file/getImage?fileId=5e2570f9ab64411ab6006b33)

Cobalt Strike 的外部 C2 接口允许第三方程序充当 Cobalt Strike 与其 Beacon payload 之间的通信层。那么加上外部 C2，CS 的 C2 通信架构就变成了这样：

![title](https://leanote.com/api/file/getImage?fileId=5e257187ab64411ab6006b69)

其实，接口就是一个函数方法。CS 允许用户在遵循外部 C2 规范的前提下，使用此接口定义自己的外部 C2 协议。

那么这个外部 C2 本身的实现是怎么样的呢？CS 的外部 C2 规范文档规定：

- 外部 C2 系统由`第三方控制器`、`第三方客户端`和由 Cobalt Strike 提供的`外部 C2 服务`三部分组成。
- `第三方客户端`和`第三方控制服务器`是独立于 Cobalt Strike 外部的组件。第三方可以用他们选择的语言来开发这两个组件。
- `外部 C2 服务`是第三方程序与你的 CS 团队服务器之间交互的地方。

所以我们可以得出信息：外部 C2 也是 C/S 架构的。

总结来说，外部 C2 本身的具体实现如下：

![title](https://leanote.com/api/file/getImage?fileId=5e2577a3ab64411ab6006d99)

在 CS 4.0 中，


![title](https://leanote.com/api/file/getImage?fileId=5e256632ab64411ab600671d)


这个 `External C2` payload，其实是在定义 CS 团队服务器的外部 C2 服务接口，这个选项只是方便建立接口本身，规定在哪一个端口上开 CS 的`外部 C2 服务器`。


于是把外部 C2 系统与 CS 通信架构结合起来，最终的使用外部 C2 的 CS 通信架构图就是这样的：

![title](https://leanote.com/api/file/getImage?fileId=5e257c57ab64411ab6006f44)

# 0x03 拓展知识：SMB Beacon 与命名管道

注意到上图中 `第三方客户端` 与 Beacon 直接通过 `name pipe` 通信。下图中的名称管道（`name pipe`）有的地方翻译为 `命名管道`。

![title](https://leanote.com/api/file/getImage?fileId=5e25b55cab64411ab6008376)



既然是通过 `命名管道` 通信，那么一定是 SMB Beacon。

> 下面这一部分参考自 @Rcoil 大佬的文章：[【知识回顾】命名管道](https://rcoil.me/2019/11/%E3%80%90%E7%9F%A5%E8%AF%86%E5%9B%9E%E9%A1%BE%E3%80%91%E5%91%BD%E5%90%8D%E7%AE%A1%E9%81%93/)

SMB Beacon 命名管道 payload 适用于什么样的通信场景呢？

假设这样一个场景：

> 在 Windows 环境中，无管理员权限的情况下，对已获取权限的机器上使用 ncat 反弹一个 shell，但是遭到防火墙或反病毒程序的阻拦。
那么实际上就是 `reverse_tcp` 被防火墙拦了。那么肯定 `reverse_http`、`reverse_https` 都走不通了。<br/>
之所以被拦是因为：
在 Windows 中，当尝试使用 Bind() 绑定一个 TCP Socket 时，Defender 会弹窗提示是否允许此程序进行网络连接，只有用户点击允许访问才可放行。当然，如果我们拥有管理员权限，可以将此程序添加到白名单中，允许连接。但我们这里使用的普通用户权限，是无法添加修改防火墙规则的。所以当无权限进行修改时，注定会弹窗提示，也意味着我们的此攻击操作注定失败。
也就是说实际上，目标机器上发生了这个弹窗：
![title](https://leanote.com/api/file/getImage?fileId=5e25c518ab64411ab60088ef)

在这种情况下，如何回连 C2 服务器呢？

这就是 SMB 协议的适用场景了。

在 Windows 中，防火墙通常默认允许 SMB 协议出入站，因此，如果有什么功能或机制可以用于与外部机器进行通信的，SMB 协议无疑是一种很好的选择。而命名管道 就是基于 SMB 协议进行通信的，所以我们可以基于命名管道与外部机器进行通信，从而建立控制通道。


`命名管道`（也就是传说中的 `IPC`）是一种简单基于 SMB 协议的进程间通信（Internet Process Connection - IPC）机制。在计算机编程里，命名管道可在同一台计算机的不同进程之间或在跨越一个网络的不同计算机的不同进程之间，支持可靠的、单向或双向的数据通信传输。

和一般的管道不同，命名管道可以被不同进程以不同的方式方法调用（可以跨语言、跨平台）。只要程序知道命名管道的名字，任何进程都可以通过该名字打开管道的另一端，根据给定的权限和服务器进程通信。

默认情况下，我们无法使用命名管道来控制计算机通信，但是微软提供了很多种 Windows API 函数，例如：

- 用于实例化命名管道的服务器端函数是 CreateNamedPipe
- 接受连接的服务器端功能是 ConnectNamedPipe
- 客户端进程通过使用 CreateFile 或 CallNamedPipe 函数连接到命名管道

在本文中，只探讨原理，所以不会去拓展到如何具体实现通过 Windows API 编写程序来实现两台机器直接的数据传输。

要注意的是：

下图是 @Rcoll 自己编写的命令执行的SMB命名管道通道的C/S两端连接的截图：


![title](https://leanote.com/api/file/getImage?fileId=5e25c707ab64411ab6008998)

可以看到，第一次从本地主机运行客户端尝试连接到远程主机的服务端，因为没有登录，所以连接失败。第二次先使用 `net use 账号` 登录之后，才连接上。

在 Cobalt Strike 中如何使用 SMB Beacon 的 payload 呢？

Windows 将命名管道通信封装在 SMB 协议中。因此得名 SMB Beacon。SMB Beacon 使用命名管道通过父 Beacon 进行通信。这种点对点的通信对于在同一台主机上的 Beacon 生效。它也可以在整个网络上运行。

![title](https://leanote.com/api/file/getImage?fileId=5e25bda1ab64411ab6008664)

Cobalt Strike 的监听器提供 SMB Beacon 命名管道 payload。

CS 3.14 中的 SMB Beacon：

![title](https://leanote.com/api/file/getImage?fileId=5e25be99ab64411ab60086b9)

CS 4.0 中的 SMB Beacon：

![title](https://leanote.com/api/file/getImage?fileId=5e25bec0ab64411ab60086c7)


从 Beacon 控制台，可以使用 `link [host] [pipe]` 来把当前的 Beacon 链接到一个等待连接的 SMB Beacon。当当前的 Beacon 登入，它的链接的点也会登入。

为了与正常流量融合，链接的 Beacon 使用 Windows 命名管道进行通信。这个流量封装在 SMB 协议中。此方法有一些限制：

1. 具有 SMB Beacon 的主机必须在端口445上接受连接。
2. 只能链接由同一个 Cobalt Strike 实例管理的 Beacon。

就如上面说到的 SMB 命名管道连接需要先用账号密码进行用户登录，因为它也是需要权限的。要连接 smb beacon，也是看当前操作的 beacon 的权限。

如果在尝试去连接到一个 Beacon 之后得到一个 error 5（权限拒绝），这就是权限不够的表现。解决方法是窃取域用户的 token （令牌）或使用 `make_token DOMAIN\user password` 来使用对于目标的有效凭据来填充当前 token（令牌）。然后再次尝试去 link 到 Beacon。

make_token 实际上使用了 `LogonUserA()` 函数，相当于赋予了当前程序这个 token 权限。
`注：`LogonUserA()函数参考：[【渗透技巧】SCshell 技术细节](https://rcoil.me/2019/12/%E3%80%90%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7%E3%80%91SCshell%20%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82/)

上面主要是介绍了命名管道作为 C2 信道，通讯执行命令的功能。

进一步拓展的话，通过 SMB 协议绕过防火墙建立 C2 信道可以作为本地权限提升漏洞的利用链中的一步。

进一步可以利用命名管道的模拟客户端功能来获取 `system` 权限，MSF 的 `getsystem` 功能就是通过此方法实现的。

把话题稍微拉回来，`SMB 命名管道`与`外部 C2` 的关系是：在加了外部 C2 的 CS 通信架构中，目标机器上的 Beacon payload 与外部 C2 系统中的第三方客户端之间是通过 SMB 命名管道来建立 C2 信道的。这也是我们整个 C2 通信中的一环：

![title](https://leanote.com/api/file/getImage?fileId=5e25d576ab64411ab6008ea9)


# 0x04 为什么需要外部 C2？

本身，自定义 C2 通道可以拓展协议，这样自然可以解决一些 C2 流量出口的问题（绕过防火墙或者进程限制）。

使用 SMB Beacon，当 payload 成功执行之后，由第三方客户端完全接管与 Beacon 的交互。所有与 Beacon 后续的交互，最终均是对命名管道的读写。命名管道可以直接作为文件来读写，多数脚本语言都支持该功能。这样就有很多发挥的空间。

另外，还有一个功能是使得 C2 流量充分融合进正常流量。要理解这一点可以从 CS 的 `Malleable C2 Profile` 功能说起。

熟悉 CS 的人都知道 CS 的 C2 profile 是可拓展的。

也就是说，通过加载自定义 C2 profile，我们可以伪装流量，让通讯更加隐蔽和控制其行为的一种方式。

```
./teamserver [团队服务器的IP] [password] [c2 profile]
```

原始通信报文：

![title](https://leanote.com/api/file/getImage?fileId=5e25aa61ab64411ab6007f9c)

使用 C2 profile 伪装的通信报文：

![title](https://leanote.com/api/file/getImage?fileId=5e25aa84ab64411ab6007faa)


一些可拓展 C2 profile 项目让我们可以把流量伪装成 APT 组织、恶意软件或其他的普通应用的流量：

![title](https://leanote.com/api/file/getImage?fileId=5e25aa06ab64411ab6007f6e)


在下图中，可以使用 C2 profile 把流量伪装成 gmail、onedrive 等应用的流量。别的项目中也见过可以伪装为 Office 365 的。

![title](https://leanote.com/api/file/getImage?fileId=5e25b0faab64411ab60081e4)

这样可以让 C2 的流量融合进正常应用流量中。

外部 C2 也可以达到让 C2 的通信流量融合进正常应用流量中这个结果，但是不是伪装，因为它是真正实现了自己的通信渠道。

通过实现自己的通信渠道，**可以拓展通信协议、提供工具集本身不支持的通信方法。**在 Cobalt Strike 的特定场景中 —— Beacon 可以通过 HTTP/S 和 DNS 协议出口 C2 流量；并通过 SMB 的命名管道和 TCP socket 链接到其他的 Beacon 。

外部 C2 规范允许将 Beacon 的通信（<u>回连团队服务器或连接到其他 Beacon</u>）方式拓展到几乎任何需要的方式。

任何需要的方式包括：

- 通过 Office 365 进行 C2 通信
- 通过文件分享进行 C2 通信
- 通过活动目录进行 C2 通信
- 通过 RDP、WinRM 或 WMI 进行 C2 通信
- ......

这样自定义 C2 通信渠道有什么好处呢？

通过合法服务（例如 `Office 365`、`Google云端硬盘`、`Dropbox`、`Slack`或其他任何服务）出口C2 流量，可以使得 C2 流量充分融合到正常流量，尤其是如果你的目标使用这些服务作为其日常业务的一部分时。在某些情况下，你甚至可以通过目标自己的应用传输 C2 流量，例如通过他们自己的 OneDrive 传输 C2 数据。

通过文件共享、活动目录或任何可以被写入/读取的应用进行的内网 C2 通信也很难被检测到，这也有助于绕过不同类型的防火墙限制。

比起来，可拓展 C2 的好处就是配置简单，外部 C2 的好处是隐蔽性更好、绕防火墙等限制的效果更好，可以拓展协议，但是实现起来比较难。



-------------------

## 参考文档：

**SMB & 命名管道部分：**

- [【知识回顾】命名管道](https://rcoil.me/2019/11/%E3%80%90%E7%9F%A5%E8%AF%86%E5%9B%9E%E9%A1%BE%E3%80%91%E5%91%BD%E5%90%8D%E7%AE%A1%E9%81%93/)
- [Windows 命名管道研究初探](https://www.anquanke.com/post/id/190207#h2-0)
- [【知识回顾】深入了解 PsExec](https://rcoil.me/2019/08/%E3%80%90%E7%9F%A5%E8%AF%86%E5%9B%9E%E9%A1%BE%E3%80%91%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3%20PsExec/)
- [【渗透技巧】SCshell 技术细节](https://rcoil.me/2019/12/%E3%80%90%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7%E3%80%91SCshell%20%E6%8A%80%E6%9C%AF%E7%BB%86%E8%8A%82/) 

@Rcoil 大佬的文章真的写的非常好，另外也感谢 @Rcoil 大佬的耐心指导，不胜感激~

**外部 C2 部分：**

- [Cobalt	Strike	External Command	and	Control	Specification](https://www.cobaltstrike.com/downloads/externalc2spec.pdf)
- [利用External C2让内网机器在Cobalt Strike中上线](https://blog.hl0rey.com/2019/09/06/%E5%88%A9%E7%94%A8External-C2%E8%AE%A9%E5%86%85%E7%BD%91%E6%9C%BA%E5%99%A8%E5%9C%A8Cobalt-Strike%E4%B8%AD%E4%B8%8A%E7%BA%BF/)
- [External C2](https://wbglil.gitbooks.io/cobalt-strike/cobalt-strikekuo-zhan/external-c2.html)
- [External C2 (Third-party Command and Control)](https://www.cobaltstrike.com/help-externalc2)
- [Rvn0xsy/Cooolis-ms](https://github.com/Rvn0xsy/Cooolis-ms)
- 当然还有必不可少的 @Klion 的 Cobalt Strike 系列（非公开故不列文章名了）