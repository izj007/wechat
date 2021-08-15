![title](https://leanote.com/api/file/getImage?fileId=5d9ed4f3ab64415a49000b1b)

Netcat 简称为nc，中文名网猫。网络里的一只猫，可见其灵活性。

# ❤ Netcat 是什么？

nc 是一个【**命令行**】工具，通过nc，可以很灵活地操纵【传输层协议】（TCP&UDP）。

OSI 7层模型：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed51eab64415c47000a6b)


`划重点！` 协议非常重要。比如，在实战中，我们可能遇到一种情况：走 TCP 无法出网、走 UDP 也无法出网，只有 DNS 才能出网。那么 nc 支持这种场景吗？NC 是支持 DNS 的。因为DNS协议位于应用层，而 netcat 操纵的 TCP 和 UDP 是传输层，只要位于传输层之上的协议， nc 就都是支持的。
要做到**根据支持的协议划分工具**。这样在实战中才能以不变应对万变。

# ❤ nc 命令行简介
nc 所有的功能，都以命令行的方式呈现。

### nc 命令行的常规形式
``` bash
nc 命令选项 主机 端口
```
- 命令选项
这部分可能包含 0~N 个选项。
- 主机
这部分可能没有，可能是IP，也可能是域名。
- 端口
这部分可能没有，可能是单个端口，也可能是端口范围。
对于【端口范围】，以两个数字分别表示“开始和结束”，中间用【半角减号/连号】相连。举例： `1-1024`。

### 命令行选项
nc 提供了很多【命令行选项】，分别对应它提供的功能。每个选项都是【单字母】。有些选项需要带选项值，有些则不需要。
选项要放在 nc 这个命令之后，每个选项前面要有一个【半角减号】，选项之间以空格分开。

![title](https://leanote.com/api/file/getImage?fileId=5d9ed53fab64415a49000b1d)


一些常见的 nc 命令行选项：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed55bab64415a49000b1f)

`注：` nc 有很多【变种】。不同的变种，会在原有 nc 的基础上增加一些新功能。比较流行的变种之一是OpenBSD 社区的变种（也叫“OpenBSD netcat”或“netcat-openbsd”），这是由 OpenBSD 社区重写的 netcat，主要增加了对“IPv6、proxy、Unix sockets”等功能的支持。很多主流 Linux 发行版的官方软件仓库已包含这个变种（比如说：Debian 家族、Arch 家族、openSUSE 家族、Gentoo 家族......）。
　
在`nc - h`的输出中，如果第一行包含 OpenBSD 这个单词，就说明当前 nc 是 OpenBSD 变种。

Ubuntu 系统使用了 OpenBSD netcat：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed579ab64415c47000a73)

### 命令行选项的【合写】形式
``` bash
nc -l -p 12345 -v
nc -l -p -v 12345
nc -lp 12345 -v
nc -lv -p 12345
nc -lvp 12345
```
所有上面这些命令都是【等价】的。只要注意，`-p`参数是一个需要带选项值的命令行选项，而`-v`、`-l`参数则不需要带选项值。所以端口值一定要写在`-p`后面。

### 如何强行终止 nc?
在命令行环境下，可以用【Ctrl C】这个组合键来强行终止当前运行的进程。
对于nc，也是一样的，可以使用【Ctrl C】来终止。


-----------

参考链接：
[1] [扫盲 netcat（网猫）的 N 种用法——从“网络诊断”到“系统入侵”](https://program-think.blogspot.com/2019/09/Netcat-Tricks.html)，Program Thinking，2019年9月18日
