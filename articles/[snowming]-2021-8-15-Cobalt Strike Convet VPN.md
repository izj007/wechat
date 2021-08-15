因为网上似乎没有关于这个功能的文章，至少中文真的没找到。所以为了应应急，我先记录下操作，原理这周末再慢慢分析。

对了，这是哪个功能呢？对应 [CS 手册](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)中第九章 `9.5 隐蔽 VPN`：

![title](https://leanote.com/api/file/getImage?fileId=5ebe34ceab64411c3300b8fa)

# 实验环境


失陷机器：

![title](https://leanote.com/api/file/getImage?fileId=5ebe2b93ab64411c3300a7bb)

目标网络：

![title](https://leanote.com/api/file/getImage?fileId=5ebe2b71ab64411e3000a75f)

# 操作过程


1、 部署 VPN 

![title](https://leanote.com/api/file/getImage?fileId=5ebe294cab64411c3300a419)

![title](https://leanote.com/api/file/getImage?fileId=5ebe2c70ab64411c3300a925)


`Launch` 之后，在 VPN Interfaces 这里可以看到刚刚部署的 VPN 接口：


![title](https://leanote.com/api/file/getImage?fileId=5ebe2cd4ab64411e3000a997)

Beacon Shell 里面也可以看到：

![title](https://leanote.com/api/file/getImage?fileId=5ebe2f4dab64411c3300ae2b)

2、 使用 VPN 


在团队服务器中，配置刚刚的 VPN 接口：


先连接到刚刚的 VPN 接口，能找到此设备:

![title](https://leanote.com/api/file/getImage?fileId=5ebe2db7ab64411e3000ab0b)


然后手动给他配置一个`目标网络`中的内网地址，如:


```
sudo ifconfig phear0 192.168.3.145/24
```

![title](https://leanote.com/api/file/getImage?fileId=5ebe2e53ab64411c3300ac30)


然后就可以通目标内网了哦：

![title](https://leanote.com/api/file/getImage?fileId=5ebe2e68ab64411e3000ac45)



回到 CS 的 VPN Interfaces 这里可以看到数据走 VPN 在收发：

![title](https://leanote.com/api/file/getImage?fileId=5ebe2ed3ab64411e3000ad19)


# 常见错误


之前我试过 `UDP`（理论上 VPN 部署最好的 tunnel），但是失败了。也试过了 `TCP(Bind)` 也失败了。如果一种协议不通，那么如下图。`client` 那里会显示 `not connected`，`tx` 和 `rx` 也没有收发的数据字节数。

![title](https://leanote.com/api/file/getImage?fileId=5ebe0a45ab64411e30006b8f)


虽然网卡设备在 CS 团队服务器主机上能找到，但是实际并没有进目标内网：


![title](https://leanote.com/api/file/getImage?fileId=5ebe3039ab64411e3000afce)


所以要选择可通的协议来部署隧道。


Channel 包括:

- UDP
- HTTP
- ICMP
- TCP(Bind)
- TCP(Reverse)


根据实际情况选择合适你的场景的。



# What's More

当你的 CS 团队服务器获取了一个目标内网的网卡之后，然后呢？

然后你可以，比如：

- msf 攻击，LHOST 设置为此 VPN 接口的 IP 即可，在上面的例子中就是 `192.168.3.145`。
- 通过【SSH 会话】的方式继续对一些 UNIX 目标进行后渗透。比如，在目标外网你无法 SSH 上去的情况，现在就有办法了（参考 [使用 Cobalt Strike 对 Linux 主机进行后渗透](http://blog.leanote.com/post/snowming/c34f9defe00c)）。
- ......


这几天很忙，原理周末有时间再补上。

-----------------------------------

# 参考文档：



1. [VPN Pivoting with Cobalt Strike](https://www.youtube.com/watch?v=YLwJORPJ5OA&app=desktop)，YOUTUBE，Raphael Mudge，2014年9月19日
2. [Covert VPN - Cobalt Strike](https://www.youtube.com/watch?v=jrcjJm0COI0&app=desktop#menu)，YOUTUBE，Raphael Mudge，2012年9月4日
3. [VPN Pivoting over ICMP](https://www.youtube.com/watch?v=RGlw6us2aG0&app=desktop)，YOUTUBE，Raphael Mudge，2014年11月19日
4. 原理：[Covert VPN – Layer 2 Pivoting for Cobalt Strike](https://blog.cobaltstrike.com/2012/09/05/covert-vpn-layer-2-pivoting-for-cobalt-strike/)，CS 官网，Raphael Mudge，2012年9月5日
5. 原理：[How VPN Pivoting Works (with Source Code)](https://blog.cobaltstrike.com/2014/10/14/how-vpn-pivoting-works-with-source-code/)，CS 官网，Raphael Mudge，2014年10月14日
6. [CobaltStrike4.0用户手册_中文翻译](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)，QAX-A TEAM

