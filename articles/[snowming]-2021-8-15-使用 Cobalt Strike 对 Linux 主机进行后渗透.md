因为 Cobalt Strike 只能生成针对 Windows 的有效载荷，并不是全平台的。所以使用 Cobalt Strike 对 Linux 主机进行后渗透常常被人忽略。但是其实是可以做到的。

主要是为了对目标网络形成控制链。长话短说，有两种方法可以在 Cobalt Strike 中让 Linux 主机上线：

# 0x01 方法一：SSH 会话

## 1、原理篇

【SSH 会话】是针对 UNIX 目标进行后渗透的 Cobalt Strike 工具。

使用 【SSH 会话】 作为 agent 有两个前提：

1. 你已经有1个 Windows Beacon Shell 了。因为这个 Linux 的 Beacon 需要从一个 Windows Beacon 来初始化；
2. 需要知道 Linux 主机的可登录的 SSH 凭据。

那么你可能会问了，那如果有了 SSH 凭据，为什么我不自己登上去看，还非要上个 CS 干什么，毕竟 CS 又不是稳控。

个人认为这主要是为了在后渗透的网络拓扑中把目标网络的主机们串起来，便于横向。因为 SSH 会话生成的 Beacon 还具有连接到 TCP Beacon 的功能。这样可以形成一个 `Win` → `Linux` → `Win` 的拓扑链。


那么为什么使用 【SSH 会话】 作为在目标机器上的 agent？

- 功能上：
    - 可以上传、下载、执行命令和作为跳板
    - 支持加密通讯
    - 在多种操作系统和架构的环境中生效
- 目标上自带。大多数 UNIX 目标中已经提供了 SSH 程序。



功能上已经实现了 Beacon 的基本功能了。如果要重新设计创建具有以上这些特性和功能的一个 agent，并且让此 agent 在多种操作系统和架构的主机环境中生效是非常困难的。而且 SSH 会话还是绑定代理（`bind agent`），因此从概念上讲，它非常适合 Cobalt Strike 的模型，可以用于进一步连接到 TCP Beacon。


`注：`如果想要自定义配置 agent 可以使用 `dropbear SSH` 软件。它是一个小型 SSH 服务器，你可以在编译时将自定义配置嵌入到其中，与这个特定的客户端功能搭配使用，这可能是一个很好的后门。


## 2、操作篇

**<u>Beacon 初始化：</u>**

- 使用账号密码启动 SSH 会话

```
ssh [目标:端口] [用户名] [密码]
```
- 使用密钥启动 SSH 会话
```
ssh-key [目标:端口] [用户名] [/path/key]

```

**<u>Beacon 中的命令：</u>**

- 运行命令

```
shell [命令] [参数]
```

- 使用 sudo 运行命令（此命令不一定成功，这是 CS 的 bug）

```
sudo [密码] [命令] [参数]
```

- 改变文件夹

```
cd /路径/
```

- 上传/下载文件

```
upload [/本地路径/文件]
download [文件]
```

SSH 会话基本上就是一个简化版本的 Beacon。

**<u>跳板功能：</u>**


- 启动 SOCKS 跳板（`pivoting`）
```
socks 1234
```

- 反向端口转发

```
rportfwd [监听端口] [转发的主机] [转发的端口]
```
`rportfwd` 命令要求 SSH 守护进程的 `GatewayPorts` 选项被设置为 `yes` 或者是 `ClientSpecified`；其中如果此选项被设置为 `ClientSpecified` 则要求反向端口转发绑定为 localhost 之外的地址。

注：用 `dropbear SSH` 就不会有问题，但如果只是使用 凭据验证至 SSH 守护进程就要记住这个问题。

**<u>重定向器功能：</u>**

还可以进行一些跨会话的跳板（`pivoting`）操作。

SSH 会话可以控制 TCP beacon，通过 `connect [host] [port]` 命令连接到一个等待连接的 TCP Beacon。


- 连接到一个 TCP Beacon

```
connect [主机] [端口] 
```

- 创建一个反向 TCP Beacon 监听器
```
[会话] → Pivoting → Listener 
```
![title](https://leanote.com/api/file/getImage?fileId=5e402838ab6441047f05fc05)



> 注：<br>

> - `Pivot Listener` 要求 SSH 守护进程的 `GatewayPorts` 选项被设置为 `yes` 或是 `ClientSpecified`。
- 如果 `GatewayPorts` 选项未被设置为 `yes` 或是 `ClientSpecified`，那么你的跳板监听器（`pivot listener`） 会被绑定到 localhost，这将不会有效。
- 在此选项设置没有问题的情况下，你可以将一个受害的 UNIX 目标主机转换为用于反向TCP Beacon 会话的重定向器。

## 3、实例

首先通过已有的 Windows Beacon Shell 初始化一个 SSH 会话 Beacon：

```
sleep 0 
ssh [目标主机ip:端口] [用户名] [密码]
```

![title](https://leanote.com/api/file/getImage?fileId=5e40369cab6441047f063db6)


然后就上线了一个 Linux Beacon Shell：

![title](https://leanote.com/api/file/getImage?fileId=5e40378aab6441047f0640b8)

![title](https://leanote.com/api/file/getImage?fileId=5e4037aaab6441026a0641dd)

实际测试中，这个 SSH 会话 Beacon Shell 老掉线，于是就没进行进一步的功能测试。

# 0x02 方法二：Cross C2 项目


----------------------------

注：截止今日（2020/5/17），该项目暂不支持 CS 4.0。因为协议有所改变，所以在 CS 4.0 上使用此项目上线的 Beacon 会存在通信问题，如「服务端解密出错」。

该项目支持的版本截止到 CS 3.14 最后一个版本（bug fixs 版本）。


---------------------


[Cross C2](https://github.com/gloxec/CrossC2) 项目是一个可以生成 Linux/Mac OS 的 CS payload 的跨平台项目。

该项目的中文文档已经说的非常清楚了，作者也做了更新，修复了原本生成 Payload 时候的 bug。并且优点在于实操发现生成的 Beacon Shell 很稳。

操作过程中要注意如下几点：


- 要在 Linux/Mac OS 系统下起 CS 客户端，Windows 下不可以。
- 使用 `windows/beacon_https/reverse_https` 监听器。
- 要把团队服务器下的隐藏文件 `.cobaltstrike.beacon_keys` 复制到本地 CS 目录下。
- 文件都丢到 CS 客户端根目录下，别搞二级目录。
- 生成的 payload 是一个 Linux 下的<u>可执行文件</u>，`ip` 和端口对应那个 `windows/beacon_https/reverse_https` 监听器。


把生成的可执行文件丢到目标 Linux 机器下执行，即可上线：

![title](https://leanote.com/api/file/getImage?fileId=5e403ab4ab6441026a065118)

![title](https://leanote.com/api/file/getImage?fileId=5e403a37ab6441026a064f14)

![title](https://leanote.com/api/file/getImage?fileId=5e403ad2ab6441047f0650fa)

![title](https://leanote.com/api/file/getImage?fileId=5e4039f1ab6441047f064d2e)

![title](https://leanote.com/api/file/getImage?fileId=5e403a1dab6441026a064ea9)

当然，上线之后，你在 Windows 下面起的 CS 客户端，也可以去操作这个 Linux Beacon Shell。

# 0x03 总结

经过两种方法上线了 Linux Beacon Shell 之后的拓扑图如下：

![title](https://leanote.com/api/file/getImage?fileId=5e403b50ab6441047f0652b7)

总之，通过 SSH 会话上线 Linux Beacon Shell，有两个前提：

1. 你已经有至少1个 Windows Beacon Shell 了。因为这个 Linux 的 Beacon 需要从一个 Windows Beacon 来初始化；
2. 需要知道 Linux 主机的可登录的 SSH 凭据。

实测不是很稳。但是功能多，可以作为重定向器继续连接到 TCP Beacon，方便横向。



通过 Cross C2 项目生成可以控 Linux 的 payload 上线的 Linux Beacon Shell，简单稳定。


# 0x04 更新


[Geacon](https://github.com/darkr4y/geacon) 这个项目支持 Linux 和 Mac OS 机器 CS 上线。并且支持 CS 4.0。

我还没测试，有空来补坑。


-------------------

参考文档：


[1] [Youtube 视频 - 【字幕版】Red Team Ops with Cobalt Strike 9 of 9 Pivoting](https://www.youtube.com/watch?v=T6JOzXmpmZw&list=PLC0e6z60AM2Gqimz0kFiLBp5cCnCQGoud&index=9)，Youtube，Raphael Mudge
[2] [Cross C2](https://github.com/gloxec/CrossC2/blob/master/README_zh.md)，Github，gloxec
