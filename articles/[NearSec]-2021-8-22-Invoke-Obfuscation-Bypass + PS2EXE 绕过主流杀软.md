##  Invoke-Obfuscation-Bypass + PS2EXE 绕过主流杀软

原创 nearsec [ NearSec ](javascript:void\(0\);)

**NearSec** ![]()

微信号 nearsec

功能介绍 专注Web安全领域，分享渗透测试、漏洞挖掘实战经验，面向广大信息安全爱好者。 NearSec - 更接近安全

____

__

收录于话题

#免杀 1

#cobaltstrike 1

#bypass 1

  

本文通过混淆器对cs生成的powershell文件进行混淆，并打包成exe的方式，来绕过主流杀软，提供一个简单思路  

  

项目地址：

https://github.com/AdminTest0/Invoke-Obfuscation-Bypass

  

原混淆器运行后，某60云查杀会报毒11个文件

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075252.png)

  

某绒会报毒4个文件  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075253.png)

  

将原项目中的以下文件混淆后进行测试  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    Invoke-Obfuscation.ps1Out-EncodedAsciiCommand.ps1Out-EncodedBinaryCommand.ps1Out-EncodedHexCommand.ps1Out-CompressedCommand.ps1Out-EncodedBXORCommand.ps1Out-EncodedOctalCommand.ps1Out-ObfuscatedStringCommand.ps1Out-PowerShellLauncher.ps1Out-SecureStringCommand.ps1Out-EncodedWhitespaceCommand.ps1

  

再次运行不会被拦截

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075254.png)

  

将Invoke-Obfuscation-Bypass打包成exe，绕过defender

  

![]()

  

下面通过Invoke-Obfuscation-Bypass对powershell文件进行混淆

  

用cs生成powershell的payload

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075255.png)

  

用powershell启动Invoke-Obfuscation-Bypass.exe，或者双击打开，会在同目录生成混淆后的Invoke-
Obfuscation

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075256.png)

  

依次输入选项进行混淆（其他混淆方式自测）  

  *   *   *   * 

    
    
    set scriptpath C:\Users\用户名\Desktop\payload.ps1tokenall1

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075257.png)

  

导出混淆后的payload  

  * 

    
    
    out ..\payload1.ps1

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075258.png)

  

使用ps2exe将混淆后的ps1文件转为exe  

  * 

    
    
    .\ps2exe.ps1 -inputFile 'payload1.ps1' -outputFile 'Test.exe' -runtime40 -lcid '' -MTA -noConsole -supportOS

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075259.png)

  

绕过defender动态  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075300.png)

  

参考链接：

https://github.com/danielbohannon/Invoke-Obfuscation

https://github.com/MScholtes/TechNet-Gallery/blob/master/PS2EXE-GUI/ps2exe.ps1

https://github.com/cseroad/bypassAV

  

  

小广告时间，团队小伙伴创建的知识星球，干货多多~ 欢迎各位师傅加入

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075302.png)

![]()

nearsec

奥利给～

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

奥利给～

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

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

Invoke-Obfuscation-Bypass + PS2EXE 绕过主流杀软

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

