##  记录：已下线的某QP站点的渗透测试

原创 Snaker  [ Snaker独行者 ](javascript:void\(0\);)

**Snaker独行者** ![]()

微信号 gh_e5407d383892

功能介绍 记录本人网络安全知识/经验/笔记，致敬未来的我。

____

__

收录于话题

#渗透测试 7

#经验总结 9

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091614.png)

  

当刚看到这个站点的时候，其实从登录页面上是得不到任何与BC、QP有关的信息的，所以从前台上也无法判断其就属于BC、QP类的资产，但是开局就送了个弱口令，进到后台，发现其属于QP类站点。此次的渗透比较顺利，思路比较简单，若有不足之处，望师傅们斧正。

  

进入正题：

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091615.png)

  

这里开局就送个弱口令 admin/123456单从这点我就已经觉得站点大概率已经荒废了

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091617.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091618.png)

  

大量的机器人用户，但是却又发现近期存在充值记录，近半年仅有7条充值记录，说明这站点还是大概率荒废了。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091619.png)

  

看到到这里，我只想简单渗透一下此站点，当练练手记录记录吧，因为在后台处找不到任何一处上传点，所以想通过文件上传Getshell是没办法了，所以回到登录框测试。
这种仅有登录功能的站点，优先测username、password这俩参数是否存在注入。抓登录请求包，发现userName参数存在注入，上sqlmap

  * 

    
    
    python2 sqlmap.py -r post.txt --random-agent --batch

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091620.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091621.png)

  

发现是堆叠注入，MSSQL数据库，看看它的数据库

  * 

    
    
    python2 sqlmap.py -r post.txt --random-agent --batch --dbs

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091622.png)

  

从这里的数据库名称就可以判断，其中存在代理、平台、会员等数据库。接下来先不爆数据，先想办法拿shell。开始尝试利用os-shell
Getshell，在这之前先判断最近用户是否为DBA。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091623.png)

  

是DBA就直接os-shell，如果不是DBA，就没必要os-shell了。这里可以直接os-shell，成功执行命令。

  *   * 

    
    
    python2 sqlmap.py -r post.txt --random-agent --batch --is-dbapython2 sqlmap.py -r post.txt --random-agent --batch --os-shell

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091624.png)

  

那么接下来就是利用certutil远程下载CS马，执行上线（这里最好就查看一下进程，看看有没有存在杀软，但是因为接收回显信息比较慢，所以为了高效，我这里就直接下载执行了，然后根据回显结果去判断，如果上线不成功，那有两种情况，要么杀软拦截，要么不能出网）

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091625.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091626.png)

  

当前不是system权限，所以先进行提权，利用甜土豆进行提权并反弹到CS上。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091627.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091628.png)

  

拿到最高权限后，我这才开始收集系统相关信息

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091629.png)

  

![]()

  

Windows Server 2008 R2、阿里云服务器、没安装杀软。那么接下来很好办，我要做的事如下：1. 读取Administrator明文密码2.
查找敏感信息文件，提取其中的数据库账号密码、网站后台账号密码等信息3. 权限维持  
事情一件件做，先利用mimikatz读密码

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091630.png)

  

得到Administrator管理员密码，查看一下远程桌面连接是否开启

  * 

    
    
    REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091631.png)

  

0x0代表开启，0x1代表关闭再查看一下主机远程桌面连接的端口

  * 

    
    
    REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-TCP" /v PortNumber

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091632.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091633.png)

  

0xd3d即3389，说明端口并没有被刻意修改，那就立马远程登陆，登陆之前看看管理员是否在线。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091634.png)

  

处于断开状态，那直接利用管理员账号密码远程登陆。

  

![]()

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091635.png)

  

刚进入就看到其开着在线游戏服务控制端，同时还发现其数据库正在连接着，因为我可以通过注入查看这些数据，所以不着急现在分析其数据库。（后面得到数据库账号密码，更是可以直接连接进行分析）

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091636.png)

  

先看看近期打开过的文件以及它的桌面、浏览器记录等。首先我是在谷歌浏览器的历史记录中发现其对应的站点。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091637.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091638.png)

  

![]()

这提示说明站点下线了或者更名了，但是上图却显示其游戏控制端还在启动状态，这很大可能是它的棋牌APP还没下线，这网站或许是其导流站。还发现存在两个隐藏用户，看来已经被入侵过了。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091639.png)

  

在C盘找到网站源码，经过初步筛选，其中真正有用的只有两个源码文档，如下：

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091640.png)

  

将其打包并下载到本地进行分析

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091641.png)

  

利用VScode的全局搜索定位敏感信息，查找"password"、"passwd"、"database"等关键词

  

![]()

  

找到数据库账号密码，用Nmap扫描过其端口，1433端口是开放着的，那尝试在公网直接连接其数据库。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091642.png)

  

连接成功，暂不深入查看其数据，因为在其phpstudy_pro中发现一个正在运营的站点。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091643.png)

  

随便点击后跳转到APK下载页面

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091645.png)

  

下载后运行看看

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091646.png)

  

初步查看了一下，并无异常的地方，没有发现更多有价值的信息了，接下来进行IFEO映像劫持，以达到权限维持的目的。

  *   * 

    
    
    reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\Utilman.exe" /freg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\Utilman.exe" /v Debugger /t REG_SZ /d "c:\Windows\system32\cmd.exe " /f

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091647.png)

  

再多留一个Telemetry后门

  

![]()

  

![](https://gitee.com/fuli009/images/raw/master/public/20210822091648.png)

  

到此结束，简单的总结下整个流程： 前台登录框注入 -> 通过注入上线CS -> 甜土豆提权 -> 获得system权限 ->
mimikatz获取管理员账号密码 -> 内网信息收集 -> 网站源码文件 -> 数据库配置文件 -> 数据库账号信息 -> 获得数据库管理员账号密码 ->
IFEO映像劫持 -> Telemetry权限维持

  

  

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

记录：已下线的某QP站点的渗透测试

最多200字，当前共字

__

发送中

写下你的留言

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

