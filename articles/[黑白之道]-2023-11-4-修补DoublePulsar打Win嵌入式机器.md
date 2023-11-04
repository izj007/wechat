#  修补DoublePulsar打Win嵌入式机器

[ 黑白之道 ](javascript:void\(0\);)

**黑白之道** ![]()

微信号 i77169

功能介绍 我们是网络世界的启明星，安全之路的垫脚石。

____

___发表于_

收录于合集

![]()

在我的一次工作中，我发现一些Windows设备受到 **MS17-010** 漏洞的影响。

其中一个设备引起了我的注意，因为它是我从未遇到过的东西 -  **Windows Embedded** 操作系统。![]()  

由于它容易受到MS17-010的影响，我立即尝试了相关的Metasploit模块。然而，它们都不起作用，我得到的只是一个错误，指出不支持目标操作系统。

![]()  

即使是新的MS17-010模块ms17_010_psexec也无法正常工作。

![]()  

这很奇怪，也许MSF的辅助模块给了我一个误报，或者漏洞利用模块的作者可能忘记包含对Windows Embedded的支持。

![]()  

为了验证目标是否确实容易受到攻击，我决定使用MS17-010的原始漏洞。因此，我启动了 **Fuzzbunch** ，然后使用了 **SMBTouch，**
结果表明，目标实际上是通过 **永恒之蓝** 受到攻击的。

![]()  

然后我很快使用了EternalBlue模块，结果成功了——后门成功安装在目标上。所以我猜测MSF漏洞利用模块的作者只是忘记添加对Windows
Embedded版本的支持。

![]()  

由于后门已经安装，因此完成利用并获得shell所需要做的最后一件事就是使用 **DoublePulsar** 。首先，我生成了DLL格式的shell。

![]()  

然后我使用 **DoublePulsar** 将生成的DLL注入到目标主机。但是，它失败并显示错误消息[-] ERROR unrecognized OS
string。

毕竟不支持Windows Embedded版本，我猜MSF模块是正确的。

![]()  

距离交战结束还剩几个小时，我决定深入挖掘并检查DoublePulsar。

首先，我搜索了尝试使用DoublePulsar时收到的错误消息，该字符串是在.text部分找到的0x0040376C。![]()  

为了更好地理解DoublePulsar如何最终出现该错误消息，我决定使用IDA的图形视图来跟踪程序的流程。

![]()  

从图中可以看出，如果目标机器运行的是Windows 7，它将走 **左边的路径** ，然后继续检测其架构是x86还是x64。如果目标不是Windows
7，它将采用 **正确的路径** 并执行其他操作系统检查。由于没有检查Windows Embedded，程序最终输出了错误消息[-] ERROR
unrecognized OS string。

![]()  

通过进一步分析“ **Windows 7 OS Check** ” ，我发现我可以通过修改指令来“强制”程序走 **左边的路径** 。jz short
loc_403641jnz short loc_403641

![]()  

为此，我转到  **Edit > Patch program > Change byte.**

![]()  

然后我将值74 (  **JZ** 的操作码)更改为75 (  **JNZ** 的操作码)。

![]()  

这是修改跳转指令后的样子。

![]()  

然后，我通过转到  **File > Produce file > Create DIF file… **创建了一个DIF文件。

![]()  

然后使用@stalkr的脚本来修补修改后的exe文件。

  * 

    
    
    https://stalkr.net/files/ida/idadif.py

![]()

  

然后将修改后的Doublepulsar-1.3.1.exe移回原来的位置。

![]()

  

使用修改后的DoublePulsar，我能够将生成的DLL负载注入到目标主机。  
![]()  

并获得了 **SYSTEM  shell**。

![]()

>  **文章来源：潇湘信安**

  

黑白之道发布、转载的文章中所涉及的技术、思路和工具仅供以安全为目的的学习交流使用，任何人不得将其用于非法用途及盈利等目的，否则后果自行承担！

如侵权请私聊我们删文  

  

 **END**

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

