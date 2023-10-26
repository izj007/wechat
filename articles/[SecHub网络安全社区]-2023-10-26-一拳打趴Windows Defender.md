#  一拳打趴Windows Defender

Mr.x  [ SecHub网络安全社区 ](javascript:void\(0\);)

**SecHub网络安全社区** ![]()

微信号 secevery0x01

功能介绍
隶属于SecHub网络安全社区，致力于研究渗透测试、红蓝对抗、应急响应、实战案例、钓鱼社工、和漏洞分析，定期分享实战案例、技术教程、安全工具等前沿网络安全资源。

____

___发表于_

收录于合集 #渗透工具 53个

**点击蓝字 关注我们**

![]()

 **免责声明**

本文发布的工具和脚本，仅用作测试和学习研究，禁止用于商业用途，不能保证其合法性，准确性，完整性和有效性，请根据情况自行判断。

如果任何单位或个人认为该项目的脚本可能涉嫌侵犯其权利，则应及时通知并提供身份证明，所有权证明，我们将在收到认证文件后删除相关内容。

文中所涉及的技术、思路及工具等相关知识仅供安全为目的的学习使用，任何人不得将其应用于非法用途及盈利等目的，间接使用文章中的任何工具、思路及技术，我方对于由此引起的法律后果概不负责。

 **添加星标不迷路  
**

由于公众号推送规则改变，微信头条公众号信息会被折叠，为了避免错过公众号推送，请大家动动手指设置“星标”，设置之后就可以和从前一样收到推送啦![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/2_04.png)

  

## 关于

一个工具，用于删除Windows Defender在Windows 8.x，Windows 10（每个版本）和Windows 11。

汗流浃背了吧？傻小子

![]()

###  **app是做什么的？**

此应用程序删除/禁用Windows Defender，包括Windows安全应用程序、基于Windows虚拟化的安全性（VBS）、Windows
SmartScreen、Windows安全服务、Windows Web威胁服务、Windows文件虚拟化（UAC）、Microsoft
Defender应用程序防护、Microsoft驱动程序阻止列表、系统缓解措施以及Windows 10或更高版本上设置应用程序中的Windows
Defender页面。

###  **系统要求**

  * Windows `8.x`、`10`和`11`（所有版本）。

###  **使用说明**

注意

建议在运行脚本之前使用系统还原点。

  1. 从Releases下载打包的脚本

  2. 以管理员身份运行“.exe”

  3. 按照显示的说明进行操作

![]()

如果您遇到任何问题，请提交问题。

###  **📃脚本的自动化**

您可以使用参数禁用或启用Windows Defender。

#### 启用/禁用Windows Defender和安全组件

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    # ENABLEDefender.Remover.exe /r <# or /R #>  
    # DISABLEDefender.Remover.exe /n <# or /N #>仅启用/禁用Windows Defender Antivirus# ENABLEDefender.Remover.exe /e <# or /E #>  
    # DISABLEDefender.Remover.exe /m <# or /M #>

###  **禁用或删除Windows Defender _应用程序防护策略_ （高级）**

如果您在打开应用程序时遇到任何问题（非常罕见）并收到消息“应用程序无法运行，因为Device Guard”或“Windows Defender
Application Guard阻止了此应用程序”，则必须从不同位置删除4个同名文件。

  * 

  *   *   *   *   *   *   *   * 

    
    
    在EFI分区中Remove-Item -LiteralPath "$((Get-Partition | ? IsSystem).AccessPaths[0])Microsoft\Boot\WiSiPolicy.p7b"在代码完整性文件夹中Remove-Item -LiteralPath "$env:windir\System32\CodeIntegrity\WiSiPolicy.p7b"在Windows文件夹中Remove-Item -LiteralPath "$env:windir\Boot\EFI\wisipolicy.p7b"在WinSxS文件夹中Remove-Item -Path "$env:windir\WinSxS" -Include *winsipolicy.p7b* -Recurse

####  **常见问题**

####  **为什么下载的可执行文件被标记为病毒？**

这是假阳性。

由于“.exe”文件的创建方式，某些安全应用程序会将此应用程序标记为病毒。

####  **为什么更新Windows时补丁不起作用？**

Windows Update包含`Intelligence Update`，它阻止某些操作并修改Windows
Defender/Security策略。如果该脚本对您不起作用，请检查您是否安装了Windows安全智能更新。如果这样做，请禁用篡改保护，然后重新运行脚本。

#### 如何在不从发行版下载可执行文件的情况下使用软件包移除程序？

使用PowerRun从cmd运行所需的“.bat”文件（通过拖动到可执行文件）。您必须重新启动以使更改生效。

 **项目地址** ****  

  * 

    
    
    https: //github.com/ionuttbara/windows-defender-remover

  

  

欢迎关注SecHub网络安全社区，SecHub网络安全社区目前邀请式注册，邀请码获取见公众号菜单【邀请码】

![]()

 **联系方式**

电话｜010-86460828

官网｜http://www.secevery.com

![]()

 **关注我们**

![]()![]()![]()

 **公众号：** sechub安全

 **哔哩号：** SecHub官方账号

  

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

