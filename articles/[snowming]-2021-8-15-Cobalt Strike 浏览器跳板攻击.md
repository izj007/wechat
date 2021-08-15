# 0x01 概念介绍

浏览器跳板攻击（`Browser Pivoting`）是一个应用层的跳板技术。

设想一个场景：

攻击者获取了目标机器的 Beacon shell，然后通过 Cobalt Strike 的 `screenshot` 工具进行截屏，看到受害机上的终端用户正在与 web 应用程序进行交互，比如登陆了在线邮箱，正在邮箱应用的网页版客户端查看邮件。这种应用对于实现后渗透目标具有很高的价值。

如何去利用这些 web 应用呢？

【浏览器跳板攻击】就是适用于这种场景的一种攻击方式。

简单来说，浏览器跳板攻击可以让攻击者以受害主机上的终端用户的身份来访问浏览器上开着的应用。攻击者可以继承目标用户对于网站的访问权限，相当于直接跳过了对于浏览器上的应用程序的身份验证。


![title](https://leanote.com/api/file/getImage?fileId=5e369b88ab64417712006329)


【浏览器跳板攻击】使攻击者可以用自己的浏览器通过目标的浏览器中继请求。这使攻击者可以以目标用户的身份与应用网站进行静默交互、实现后渗透目标。

<u>但是，前提是终端用户必须使用 `Internet Explorer` 浏览器（`iexplore.exe`），也就是说，只可以以目标用户的身份访问目标用户开在 `Internet Explorer` 浏览器中的那些应用（区别于 `explorer.exe`），无法访问终端用户开在 Edge、Chrome 等浏览器上的那些应用。</u>


# 0x02 实现原理

下面介绍【浏览器跳板攻击】的实现原理。

如果使用 socks 跳板/代理跳板来访问受害机终端用户打开的那些 web 应用，就无法通过身份认证：

![title](https://leanote.com/api/file/getImage?fileId=5e36a80bab64417712006557)

那为什么浏览器跳板攻击与 socks 跳板不同，可以通过身份认证呢？

![title](https://leanote.com/api/file/getImage?fileId=5e36bf5cab6441771200698f)


关键点在于 `WinINet` 这个库。工作原理是：

1. 进程注入。浏览器跳板技术将一个 agent（代理）注入到 IE 浏览器进程中；
2. 在团队服务器上创建一个 HTTP 代理服务器。到时候攻击者通过请求此代理服务器的 IP 和端口，进而变成了 agent 的一个请求任务；
3. 当攻击者从自己的浏览器请求 web 应用时，IE 中的 agent （代理）将此请求转化为对 `WinINet` 库的 API 调用；
4. 恰好， WinINet 也是 IE 浏览器用于 web 通信和管理身份认证的库。Internet Explorer 将其所有通信委托给 WinINet 库。并且使用 WinINet 这个库来管理其用户的 cookies、SSL 会话和服务器身份验证；
5. 基于相同的进程上下文，使用此库来进行一个 web 请求可以引发免费的透明再验证。攻击者的 web 请求于是获取了终端用户的cookies、SSL 会话和服务器身份验证；
6. 最终，攻击者的 web 请求就成为了当前开着的 IE 浏览器的进行的一个新的请求。

![title](https://leanote.com/api/file/getImage?fileId=5e36ef48ab644175150072a0)

# 0x03 具体操作

**背景:** 通过 Cobalt Strike 的 `screenshot` 工具看到目标用户使用 IE 浏览器通过身份验证登陆了 processon 网站，想通过浏览器跳板攻击查看目标用户在此网站上的内容。

![title](https://leanote.com/api/file/getImage?fileId=5e36ec85ab6441771200725d)

**第一步：设置浏览器跳板**

```
beacon> sleep 0
```

先把 beacon 设为交互模式。因为浏览器跳板是通过 beacon 会话来隧道通信传输数据的，所以 beacon 连接到团队服务器的频率会影响浏览器跳板的同步性。所以要把 beacon 会话设为交互模式来实现最好的效果。

然后设置浏览器跳板代理（`agent`）。这一步实际上会完成两个任务：

- 将 agent 程序注入受害机器的 IE 浏览器进程
- 在团队服务器的一个端口上开启一个 HTTP 代理服务器

`[Beacon]` → `Explore` → `Browser Pivot`：

![title](https://leanote.com/api/file/getImage?fileId=5e36f292ab6441751500733b)

![title](https://leanote.com/api/file/getImage?fileId=5e36f342ab6441771200738b)

可以看到全部可注入的浏览器进程，包括 `explorer.exe`、`iexplore.exe`。

经本人实验，选择 `x86` 的 `iexplore.exe` 进程进行注入效果比较好。也就是打勾的这些进程，这些打勾的进程表示是 IE 浏览器进程的子标签页。

我选择了 `pid` 为 21260 的进程进行注入：选中之后按 `Launch`。（注：可以在 `Proxy Server Port` 字段选择 HTTP 代理服务器在团队服务器上开在哪个端口）

那么如下图，就对 21260 这个 pid 的 IE 浏览器进程注入了浏览器跳板 DLL，并且在团队服务器的 47855 端口启动了一个 HTTP 代理服务器：

![title](https://leanote.com/api/file/getImage?fileId=5e36f552ab644177120073e5)

实际上，这个过程也可以通过 `browserpivot` 命令来实现。效果是等同的。

顺便说一句，终止浏览器跳板会话使用 `browserpivot stop` 命令：

![title](https://leanote.com/api/file/getImage?fileId=5e37024dab6441771200763f)


**第二步：通过 chromium 浏览器访问 web 应用**

使用 chromium 浏览器的好处是：

chromium 浏览器有一个命令行参数 `ignore-certificate-errors`，加上该参数可以忽略证书错误。这允许我们浏览一些基于 SSL 的网站而不必被提示错误，在一些情况下我们很难绕过提示。所以搭配 chromium 浏览器的 `ignore-certificate-errors` 选项使得 Cobalt Strike 的浏览器跳板功能更好使用。

```
chromium --no-sandbox --ignore-certificate-errors --proxy-server=144.*.*.70:47855
```

> 注意这个 `proxy-server` 参数的值就是 HTTP 代理服务器的值：
![title](https://leanote.com/api/file/getImage?fileId=5e36f552ab644177120073e5)

在 Kali 虚拟机中输入上面的命令（Kali 内置了 chromium 浏览器），然后会自动打开浏览器页面。

然后输入 processon 的网址：https://www.processon.com/diagrams

然后就可以看到登录了目标机终端用户之后的网站页面了！

![title](https://leanote.com/api/file/getImage?fileId=5e36fa96ab644175150074a0)

用石墨网站做实验也是一样的：

访问 https://shimo.im/dashboard/used 可以看到：

![title](https://leanote.com/api/file/getImage?fileId=5e36fadbab644177120074ed)


值得注意的是，在这些 HTTPS 网站上都有证书错误，但是 chromium 浏览器的 `ignore-certificate-errors` 参数帮助我们绕过了错误提示，正常的访问到了网站。

![title](https://leanote.com/api/file/getImage?fileId=5e36fb30ab644177120074f9)


# 0x04 两个坑点

<u>1、 必须要先把 Beacon 设为交互式通信模式</u>

```
sleep 0
```

因为浏览器跳板是通过 beacon 会话来隧道通信传输数据的，所以 beacon 连接到团队服务器的频率会影响浏览器跳板的同步性。

如果处于异步通信模式下，会导致通过浏览器跳板访问到的 web 页面出现迟缓，出现我上面的这样的页面：

![title](https://leanote.com/api/file/getImage?fileId=5e36fa96ab644175150074a0)


![title](https://leanote.com/api/file/getImage?fileId=5e36fadbab644177120074ed)

<u>2、 必须要注入 x86 的 IE 浏览器进程</u>

如果一不小心注入了 `explorer.exe` 进程，就会出现如下效果：

![title](https://leanote.com/api/file/getImage?fileId=5e36fcebab6441751500750c)

![title](https://leanote.com/api/file/getImage?fileId=5e36fd37ab6441751500751a)

原因已经讲得很清楚，只有 IE 浏览器的 web 通信和管理身份认证使用了 `WinINet` 库，Explorer 浏览器并没有使用这个库。

另外必须要使用 x86 架构的 IE 浏览器**子进程**来注入浏览器跳板 DLL，因为只有注入了与打开的 IE 选项卡关联的进程才能继承会话状态（通过身份认证）。

但具体是哪个标签页进程无关紧要，因为子选项卡共享会话状态。Cobalt Strike 将在它认为你可以注入的进程旁边显示一个勾选框。

![title](https://leanote.com/api/file/getImage?fileId=5e368473ab64417515005efb)


总结一下：

要注入打勾勾的 x86 架构的 `iexplore.exe` 进程。

-------------------

参考文档：

[1] [Youtube 视频 - 【字幕版】Red Team Ops with Cobalt Strike 9 of 9 Pivoting](https://www.youtube.com/watch?v=T6JOzXmpmZw&list=PLC0e6z60AM2Gqimz0kFiLBp5cCnCQGoud&index=9)，Youtube，Raphael Mudge
[2] [Cobalt Strike mannual 4.0](https://www.cobaltstrike.com/downloads/csmanual40.pdf)，Cobalt Strike 官网
