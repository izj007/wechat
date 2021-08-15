探测连通性指的是机器能否上外网或者是内网机器之间的互通性。

# 0x01 Windows


```
# 探测 ICMP 协议出网
ping ip or domain 

# 第一个参数是需要解析的域名，第二个参数是目标 DNS 服务器的 IP，可判断目标 IP 的53端口是否正常工作
nslookup baidu.com 10.10.10.128 # 第二个参数指定 DNS 服务器

arp -a  # 查看 ARP 缓存内容
net view # 显示当前域中的计算机列表
```

不依赖第三方工具探测 TCP/UDP 是否出网：

在攻击机器开一个 python HTTP Server，然后在失陷 Windows 机器上使用 Powershell 去下东西。如果不出网会报错：


```
powershell -nop -Exec Bypass -Command (New-Object System.Net.WebClient).DownloadFile('http://47.244.96.168:8888/readme.txt','readme.txt')
```


![title](https://leanote.com/api/file/getImage?fileId=5ed65aaaab644127c2001b3c)



# 0x02 Linux


```
#Linux bash
ping ip or domain 
```

```
nc -zv ip  port  # z选项：没有输入和输出的模式,v选项：显示详细信息
```
经我测试直接这样很慢，使用下面的参数选项快很多！

```
nc -znv -w 3 ip port
```

- -w 设置超时秒数。在测试链接的时候，如果你【没】使用 -w 这个超时选项，默认情况下 nc 会等待很久，然后才告诉你连接失败。如果你所处的网络环境稳定且高速（比如：局域网内），那么，你可以追加 `-w 选项`，设置一个比较小的超时值。在上面的例子中，超时值设为3秒。
- -n：由于测试的是【IP 地址】，用该选项告诉 nc，【无须】进行域名（DNS）解析；反之，如果你要测试的主机是基于【域名】，就【不能】用`选项 -n`。加上 -n 省略了 DNS 解析的步骤，速度快很多。
- 补充说明：UDP 端口测试。通常情况下，要测试的端口都是 TCP 协议的端口；如果你碰到特殊情况，需要测试某个 UDP 的端口是否可达。nc 同样能胜任。只需要追加 -u 选项。



![title](https://leanote.com/api/file/getImage?fileId=5ecfb019ab644131af001808)



```
curl ip:port
#默认为80端口，提示信息为拒绝连接，如果是其他错误，则可判断目标端口已开启
```
![title](https://leanote.com/api/file/getImage?fileId=5ecfb1aaab644133bf001930)




```
nslookup domain ip # domain 是需要解析的域名，IP是目标 DNS 服务器的 IP，可判断目标 IP 的53端口是否正常工作
如：nslookup 0day.org 127.0.0.1 


dig @ip domain #ip 部分为 DNS 服务器，domain 为待解析的域名，可判断目标 IP 的53端口是否正常工作
如：dig @127.0.0.1 0day.org
```



另外的一些技巧，某些场景中需要内网需要全部经过代理服务器才能出网，常见的判断方法：

```
# linux
ps -ef |grep ip #查看是否存在和其他机器端口的连接
#查看内网主机是否包含字符"proxy"的机器
#查看IE浏览器的直接代理
#根据pac文件查看代理机器

curl baidu.com # 不挂代理无法正常出网
curl -x 127.0.0.1:1080 baidu.com  #注意代理的目标，不一定是本机，挂代理时通，不挂代理时不通说明存在代理机器
```
注：`ps -aux` 和 `ps -ef` 的区别在于 `ps -aux` 更详细，`ps -ef` 少一些结果列。



# 参考文档：

1. [隐藏通信隧道技术](https://wuhash.com/2020-05-27.html?from=timeline#more)，在学安全的路上，伍默，2020年5月28日
2. [如何使用PAC文件“科学上网”](https://exp-team.github.io/blog/2017/01/13/tool/using-pac/)
