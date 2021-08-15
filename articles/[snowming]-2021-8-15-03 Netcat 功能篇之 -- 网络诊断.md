`netcat`

# ❤ 功能1：【网络诊断】测试某个远程主机的【监听】端口是否可达
## 使用场景：
已经有某个网络软件开启了监听端口，然后用 nc 测试端口是否可达。
需求：要判断某个主机的监听端口是否能连上。
导致监听端口无法连上，通常有两种原因：
1. 这个监听端口根本就【没开启】；
2. 监听端口虽然开启，但是被防火墙阻拦了。
对于第1个原因，（如果我们能在该主机上运行命令）可以直接用`netstat`这个命令来查看监听端口是否开启：

使用 netstat 查看正在监听的端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed772ab64415c47000a7c)

当前主机 tcp 开放了哪些端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed798ab64415c47000a84)

当前主机 udp 开放了哪些端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed7b5ab64415a49000b39)


但是对于第2个原因（防火墙拦截了监听端口），`netstat`就用不上了，这时候可以用nc来搞定。
## 方法
``` bash
nc -nv x.x.x.x xx
(nc -nv IP port)
```
使用这个命令可以测试某个IP地址（`x.x.x.x`）上的某个监听端口是否开启。

**选项 -v**
`-v` 选项——通过更详细的输出，能帮你搞明白状况。

**选项 -n**
由于测试的是【IP 地址】，用该选项告诉 nc，【无须】进行域名（DNS）解析；
反之，如果要测试的主机是基于【域名】，就【不能】用选项 `-n`。

## 补充说明：超时设置
在测试链接的时候，如果你【没】使用 `-w` 这个【超时选项】，默认情况下 nc 会等待很久，然后才告诉你连接失败。
如果你所处的网络环境稳定且高速（比如：局域网内），那么，你可以追加`-w `选项，设置一个比较小的超时值。在下面的例子中，超时值设为3秒。
``` bash
nc -nv  -w 3 x.x.x.x xx
```
## 补充说明：UDP —— `-u` 选项
通常情况下，要测试的端口都是【TCP】协议的端口；如果你碰到特殊情况，需要测试某个【UDP】的端口是否可达。nc 同样能胜任。只需要追加` -u` 选项。

## 实验
实验环境：

主机1： 本地 Windows10 主机
![title](https://leanote.com/api/file/getImage?fileId=5d9ed839ab64415a49000b3b)

主机2：WSL —— Linux 子系统（Ubuntu 18.04）

1. nc 测试 CLOSE_WAIT 的端口
Windows 主机上 6666 端口处于 CLOSE_WAIT 状态：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed85aab64415a49000b3e)
WSL 上使用 nc 测试 6666端口不可达：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed885ab64415a49000b3f)


2. nc 测试未开启端口
Windows 主机上 49628 端口未开启：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed8a4ab64415c47000a8a)
WSL 上使用 nc 测试 49628端口不可达：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed8bbab64415a49000b48)

3. nc 测试 ESTABLISHED 的端口
Windows 主机上 62035 端口处于 ESTABLISHED状态：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed8dfab64415c47000a8e)
WSL 上使用 nc 测试 62035 端口不可达：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed8f8ab64415c47000a90)

4. nc 测试 LISTENING 的端口
Windows 主机上 135 端口处于 LISTENING 状态：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed911ab64415c47000a91)
WSL 上使用 nc 测试该端口可达：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed924ab64415c47000a92)

5. nc 加上超时选项
超时值设为1：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed93bab64415a49000b4f)

因为是在局域网内，网速非常快而稳定，所以超时值设为1秒也可以得到正确回显——对135端口连接成功。

6. nc 测试 UDP 端口
Windows 主机上 123 端口为 UDP 服务端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed950ab64415c47000a95)
只有当指定了 -u 参数时候才能连接成功此 UDP 端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed966ab64415a49000b50)


注意：
上面的三行命令非常有意思，第`1`条命令，指定了 `-w` 为1，因为超时时间设定的太短而连接超时。
第`2`条命令，nc 默认使用 `TCP` 协议测试端口，因而连接失败。
第`3`条命令，指定了 `-u` 参数，使用 `UDP` 协议测试端口是否可达，因而连接成功。
这告诉我们一个道理：
**在测试UDP端口的可达性时，一定要指定【-u】参数！！！**


# ❤ 功能2：【网络诊断】判断防火墙是否“允许 or 禁止”某个端口
## 使用场景：
假设你正在配置防火墙规则，禁止 TCP 的 8080 端口对外监听。那么，你如何【验证】自己的配置是 OK 的？
更进一步说：如果当前【没有】任何软件开启 8080 这个监听端口，你如何判断：该端口号是否会被防火墙阻拦？
为了叙述方便，设想如下场景：
有两台主机——【主机C】充当客户端，【主机S】充当服务端。
然后要判断【主机S】上的防火墙是否会拦截其它主机对 8080 TCP 端口的连接。

## 方法：
在【主机S】上运行 nc，让它在8080端口，命令如下：
``` bash
nc -lv -p 8080
```
- 选项 `-l`　　
这个选项会让 nc 进入监听模式。

- 选项 `-p`
这个选项有“选项值”，也就是具体端口号。

在【主机C】上运行nc，测试【主机S】上的 8080 端口是否可达：
``` bash
nc -nv [SeverS's IP] 8080
```
## 实验：
**实验环境：**

主机1： 本地 Windows10 主机
![title](https://leanote.com/api/file/getImage?fileId=5d9ed987ab64415c47000a96)

主机2：WSL —— Linux 子系统（Ubuntu 18.04）

**实验：**

使用 Windows 10 主机作为服务器，nc 监听 8080 端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed9a9ab64415a49000b53)

Ubuntu WSL 上使用 nc 连接 Win 10 主机的 8080 端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed9c6ab64415a49000b54)

Windows 10 nc 显示了来自 WSL 的连接：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed9dfab64415a49000b56)

这样就可以通过 nc 检测某个端口**是否被防火墙允许**。

`注意：`本实验中是在内网里面的两台主机进行的实验，实际上并未通过防火墙。真实环境里面可以用一台公网机器和本地机器进行 nc 连接尝试。

## 补充说明：是否省略【-p】？

某些 nc 的变种，在开启监听模式时，可以省略 `-p`，上述命令变为如下：
``` bash
nc -lv 8080
```
但考虑到不同 `netcat` 之间的兼容性，最好总是写上 `-p` 选项。

## 补充说明：如何让 nc 的监听端口【持续开启】?

在默认情况下，nc 开启 listen 模式充当服务端，在接受【第一次】客户端连接之后，**就会把监听端口关闭**。
为啥会这样呢？因为当年设计 nc 更多的是作为某种**网络诊断/配置工具**，并【不是】真拿它当服务端软件来用的。

如果你想要让 nc 始终监听模式，使之能【重复】接受客户端发起的连接，可以追加 `-k` 选项。
Windows 环境下 nc 没有【-k】参数：
![title](https://leanote.com/api/file/getImage?fileId=5d9eda0bab64415a49000b57)


为什么会这样呢？
因为 Windows 环境下长连接不是使用 `-k` 命令行选项，而是使用 `L`。把普通监听的 `-l` 选项换成 `-L` 就是长连接了。

Linux 环境下使用【-k】参数开启长连接：
![title](https://leanote.com/api/file/getImage?fileId=5d9eda24ab64415a49000b58)

持久连接 VS 普通连接（Windows 作服务器）：
![title](https://leanote.com/api/file/getImage?fileId=5d9eda47ab64415c47000aa1)

持久连接 VS 普通连接（Linux 作服务器）：
![title](https://leanote.com/api/file/getImage?fileId=5d9eda5cab64415c47000aa3)


## 补充说明：UDP

上述举例是基于 TCP 协议。如果要测试 UDP 协议，要记得【两边】的 nc 都要追加 -u 选项。

服务端和客户端两边都得加上【-u】参数：
![title](https://leanote.com/api/file/getImage?fileId=5d9eda8fab64415c47000aa4)


-----------

参考链接：
[1] [扫盲 netcat（网猫）的 N 种用法——从“网络诊断”到“系统入侵”](https://program-think.blogspot.com/2019/09/Netcat-Tricks.html)，Program Thinking，2019年9月18日