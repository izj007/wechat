#  深入研究IIS短文件漏洞

原创 酒零 [ NOVASEC ](javascript:void\(0\);)

**NOVASEC** ![]()

微信号 NOVA_SEC

功能介绍 NOVA SEC 新星安全 萌新启蒙之路 愿大家都能成为最闪耀的星。

____

___发表于_

收录于合集 #基础安全技术 17个

![]()

  

  

  

**_△△△点击上方“蓝字”关注我_** ** _们了解更多精彩_**  
  
  
  
 **0x00 前言** **  
** **本文篇幅较长 **,需要耐心 **阅读****.**  
 ****  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    目录 研究目标  1IIS介绍及内置版本总结  1DOS下的8.3命名规则  28.3规则由来  2查看短文件名  2基本命名格式  3基本转换规则  3扩展转换规则  3深入符号转换  3不适用的规则  4短文件名测试实例  4前缀处理  4后缀处理  4前缀后缀同时处理  4空格处理  5点号处理  5加号处理  5相同性质的类似符号  6不处理的字符  6IIS短文件名漏洞  6IIS短文件名修复方案  7IIS短文件名检测工具  7IIS异常短文件命名问题  8同名数量过多导致异常  8文件名过短异常  9不同IIS版本短文件漏洞测试  9测试文件列表  9测试方法命令  10测试Win2008 IIS7.0 ASP.NET关闭  10测试Win2008 IIS7.0 ASP.NET开启  11测试Win10 IIS10.0 ASP.NET状态未知  13测试Win2003 IIS6.0 ASP.NET未安装  14不同版本IIS测试结论  16其他不确定的结论  16衍生 不同服务器的漏洞检测  16衍生 短文件名直接访问测试  16通配符在短文件名猜测中的扩展  17通配符在短文件名猜解中的实际利用  18IIS短文件名推测长文件名  20中文短文件名的猜测问题  20部分后缀名文件无法访问  21参考  22  
      
      
    

  
  
  
 **0x01 研究目标**

  *   *   *   * 

    
    
     1 了解IIS短文件名漏洞原理2 不同场景下的短文件检测问题3 短文件名检测工具4 根据短文件名猜解实际文件名

  

注意：本文测试环境为Windows10 IIS10，故可能存在部分旧文章描述的规则不可用的问题。

  
  
  
  
 **0x0** **2 IIS介绍及内置版本总结**

  

IIS起初用于Windows NT系列，随后内置在Windows 2000、Windows XP Professional、Windows Server
2003和后续版本一起发行。

  

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    IIS 1.0，Windows NT 3.51  IIS 3.0，Windows NT 4.0 Service Pack 2  IIS 4.0，Windows NT 4.0选项包 IIS 5.0，Windows 2000  IIS 5.1，Windows XP Professional和Windows XP Media Center Edition  IIS 6.0，Windows Server 2003和Windows XP Professional x64 Edition  IIS 7.0，Windows Server 2008和Windows Vista  IIS 7.5，Windows 7（远程启用<customErrors>或没有web.config）IIS 7.5，Windows 2008（经典管道模式） IIS 8.0，Windows 8, Windows Server 2012IIS 8.5，Windows 8.1,Windows Server 2012 R2IIS 10.0，Windows 10, Windows Server 2016  
    

  
  
  
  
 **0x03 DOS下的8.3命名规则** **  
**

 **8.3规则由来**

  

Windows下的短文件名是dos+fat12/fat16时代的产物。由于FAT中的文件目录登记项只为文件名保留了8个字节，为扩展名保留了3个字节，所以MS-
DOS和Windows的用户为文件起名字时要受到8.3格式（也称8dot3命名法）的限制。

  

  

8是指文件名或目录名的主体部分小于等于8个字符，

3是指文件名或目录名的扩展部分小于等于3个字符，

中间以 . 作为分割在FAT16文件系统中。

  

最终生成类似于【PROGRA~1】（目录）或者【新建文~1.txt】（文件）这样的名称。

  

从Win95开始，采用fat32已经支持长文件名，但为了保持兼容低版本的程序，系统还是会为每个长文件名文件创建了对应的短文件名。使这个文件既可以用长文件名寻址，也可以用短文件名寻址。

  

  

 **查看短文件名**

  

在cmd下输入dir /x即可看到短文件名的效果。

![]()

  

  

 **基本命名格式**

  * 

    
    
     【前6位文件名前缀】【~】【前缀出现次数】【.】【前3位后缀名】

 **基本转换规则**

  *   *   *   * 

    
    
     1、所有小写字母均转换成大写字母2、前缀超过8位字符必定生成短文件名，3、后缀超过3位字符必定生成短文件名4、文件夹名与文件名适用相同的规则, 即使文件夹名中的后缀没有实际意义。

  

 **扩展转换规则**

  *   * 

    
    
     1、如果文件名包含某些特殊字符【如空格】，不论长度都会生成短文件2、命名规则适用与中文文件及目录情景，一个中文代表两个字符

  

 **深入符号转换**

  *   *   *   *   *   *   *   *   * 

    
    
     A、包含[' ', '+', ',', '.', ';', '=', '[', ']'] 等特殊字符的名称必定生成短文件名，无任何长度要求。  
    B、短文件名称中的特殊字符[' ','.']会被剔除，再计算前六位字符  
    C、短文件名称中的特殊字符['+', ',', '.', ';', '=', '[', ']']会被替换为[_]后，再计算前六位字符。  
    D、对使用多个"．"隔开的长文件名，取最后的[ . ]号后为扩展名,前为文件名。

  

  

 **不适用的规则**

  *   *   *   *   *   *   *   *   *   *   * 

    
    
     1、  删除空格后的长文件名，若长度大于8个字符，则取前6个字符，后两个字符以"~#"代替，若个数超过10个则取前5个字符，后三个字符以"~##"代替，若个数大于100也依此规则替换。其中"#"为数字，是相同短文件名的出现次数顺延。【大概不适用，实际场景下多个同名文件会使用数字计算生成。】  
    2、对使用多个"．"隔开的长文件名，取最左端一段转换为短文件名，取最右一段前三个字符为扩展名。【应该不适用，实际场景下，文件名中的[.]号会被直接去除。】

  

  
  
 **0x04 短文件名测试实例**  

 **前缀处理**

  *   *   *   * 

    
    
     1234567.txt          无短文件名12345678.txt          无短文件名123456789.txt        123456~1.TXT123456789          123456~1

总结：前缀长度需要大于8才会生成短文件名

  

  

 **后缀处理**

  *   * 

    
    
     abcdef.txtsss          ABCDEF~1.TXTabcdef.txt          无短文件名

总结：后缀长度大于3就会生成短文件名。

  

  

 **前缀后缀同时处理**

  *   *   *   * 

    
    
     abc12345678.txt           ABC123~1.TXTabc12345678.txtt        ABC123~1.TXTabc12345678.tx         ABC123~1.TX  abc123456789.tx       ABC123~2.TX

总结：不同长度后缀的文件名，可能生成相同的短文件名。

  

  

 **空格处理**

  *   *   *   *   * 

    
    
     abc .txt                  ABC~1.TXTabc        .txt              ABC~2.TXTabc 12345678.txt                   ABC123~1.TXTabc 123 456.txt              ABC123~2.TXT 、abc         xxxxxxxxxxxxxxxxxxxxxx1.txt    ABCXXX~2.TXT

  

总结：

1、不管文件名多长，包含空格就会生成短文件名

2、计算短文件名时，会先把空格去除。

  

  

 **点号处理**

  *   * 

    
    
     1.a.s.p.x.aspx      1ASPX~1.ASP  1.a.s.p.x.a         1ASPX~1.A    



总结：

1 计算短文件名时，会先把.号去除。

2 带点号的目录,在IIS7.0下测试发现访问目录及目录下文件是404，正常目录访问时403。

  

  

 **加号处理**

  *   *   * 

    
    
     abc+.txt          ABC_~1.TXT   abc+123+456.txt      ABC_12~1.TXTabc++++12345678.txt  ABC___~1.TXT

  

1、不管文件名多长，包含+号就会生成短文件名

2、计算短文件名时，会先把+号转换为下划线_。

  

  

 **相同性质的类似符号**

  *   *   *   *   *   *   *   *   *   * 

    
    
     123 .txt    123~1.TXT123..txt    123~1.TXT  
      
    123+.txt    123_~1.TXT123,.txt    123_~1.TXT123;.txt    123_~1.TXT123=.txt    123_~1.TXT123[.txt    123_~1.TXT123].txt    123_~1.TXT

  

  

 **不处理的字符**

  *   *   * 

    
    
     abc@.txt          无短文件名abc-.txt          无短文件名abc#.txt          无短文件名

  

  
  
  
 **0x05  IIS短文件名漏洞**  

Microsoft IIS 短文件/文件夹名称信息泄漏漏洞（中危）最开始在2010年8月1日由Vulnerability Research
Team的Soroush Dalili发现，并于2012年6月29日公开披露, 是世界网络范围内最常见的中等风险漏洞。

  

Microsoft IIS在处理带有波浪号（~）代字符的HTTP请求时，波浪号(~)会触发windows服务器的8.3名称约定。

  

Microsoft
IIS服务器会根据文件名的短文件名是否存在返回的响应存在差异，从而使得攻击者能够暴力枚举出服务器中存在的短文件名，再根据短文件名猜测出最终的文件名称信息。

  

该漏洞适用于IIS 1.0 - IIS 10.0 等全部版本，但不同版本之间可能存在不同的触发方法差异。

  

  
  
  
 **0x06 IIS短文件名修复方案**

  

Microsoft表示将不会修补此安全问题，并且NTFS 8.3文件格式支持功能默认是开启的, 需要手动进行功能关闭。

  

 **方案一：关闭NTFS 8.3文件格式的支持**

 **  
**

 **1）从CMD命令关闭NTFS 8.3文件格式的支持**

  

Windows Server 2003：(1代表关闭，0代表开启）

  * 

    
    
    关闭该功能：fsutil behavior set disable8dot3 1

  

Windows Server 2008 R2：

  *   * 

    
    
    功能是否开启：fsutil 8dot3name query关闭该功能：fsutil 8dot3name set 1

  

修改完成后，需要重启系统生效。

  

 **2）从修改注册表关闭NTFS 8.3文件格式的支持**

注册表路径：

  *   * 

    
    
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem，修改NtfsDisable8dot3NameCreation项值为 1。

  

修改完成后，需要重启系统生效。

  

注意：该方法只能禁止NTFS8.3格式文件名创建,对于已经存在的文件的短文件名无法移除，需要重新复制才会消失。

  

示例：将web文件夹的内容拷贝到另一个位置，如c:\www到c:\ww,然后删除原文件夹，再重命名c:\ww到c:\www。

  

  

 **方案二：升级.net framework到新版本。**

【该修复方案比较模糊】

  
  
  
  
 **0x07 IIS短文件名检测工具**  

 **irsdl/IIS-ShortName-Scanner:  **

  * 

    
    
    https://github.com/irsdl/IIS-ShortName-Scanner

提示：该项目没有发布各版本的release，只是把编译好的jar包放在release文件夹下。当前release文件夹下发布的版本是jdk17版本，旧版本需要在github的commit历史中去下载。

  

  

 **bitquark/shortscan:  **

  * 

    
    
    https://github.com/bitquark/shortscan

提示：shortscan是最新的IIS短文件名扫描工具，使用Golang进行开发，并且支持单词爆破。

  

  

 **VMsec/iisShortNameScaner:**

  * 

    
    
     https://github.com/VMsec/iisShortNameScaner

多线程批量检测IIS短文件名漏洞+漏洞利用

  

  

 **注意：本文后续的扫描测试都使用 irsdl IIS-ShortName-Scanner工具进行尝试。**

  
  
  
 **0x08 IIS异常短文件命名问题**  

 **同名数量过多导致异常**

 **  
**

测试过程中发现本应该命名为FILE_1~*.ASP的文件被命名为FI4388~1.ASP

![]()

  

查询资料发现，如果文件名中"~"后面的数字达到
5，则短文件名只使用长文件名的前两个字母，并通过数学方法计算长文件名的剩余字母生成短文件名的后四个字母，然后加后缀"~1"直到最后。

  

注：此处没有查出来具体的数学计算方法是什么。

  

  

 **文件名过短异常**

  

1.aspx 猜测生成为1~1.ASP,但是实际短文件名为1AD5D~1.ASP  

b.aspx 猜测生成为b~1.ASP,但是实际短文件名为B1ECF~1.ASP  

![]()

推测: 如果文件后缀大于3，但文件名长度小于3，此时会通过数学方法计算生成四个字母短文件名后，然后加后缀"~1"直到最后。

  

 **数学方法计算的特征：都是无规律的十六进制字符。**

  
  
  
  
 **0x09  ** **不同IIS版本短文件漏洞测试**  

目标：使用DEBUG、OPTIONS、GET、POST、HEAD、TRACE方法分别对IIS6.0、IIS7.0
、IIS10.0进行短文件探测测试，确认可用的探测方法。

  

查看当前IIS版本：打开IIS——查看帮助信息——关于

查询ASP.NET版本：直接查看网站目录下的ASP.NET文件夹

或查看本机所有ASP.NET版本:  

  *   * 

    
    
    reg query "HKLM\Software\Microsoft\NET FrameworkSetup\NDP" /s /v version | findstr /i version | sort /+26 /r

  

 **测试文件列表**

 **  
**

![]()

  

  

 **测试Win2008 IIS7.0 ASP.NET关闭**

  

  *   *   * 

    
    
     IIS版本: 7.0.0.0  IIS路径: C:\inetpub\wwwroot  ASP.NET状态：已关闭

  

  

 **config.ZhCN.POST.xml检测失败**

 **config.ZhCN.HEAD.xml **检测** 失败**

 **config.ZhCN.GET.xml **检测** 失败**

 **config.ZhCN.DEBUG.xml **检测** 失败**

![]()

  

 **config.ZhCN.OPTIONS.xml 检测成功**

![]()

  

 **config.ZhCN.TRACE.xml **检测** 成功**

![]()

  

  

 **测试结论：**

 **1、IIS7 可使用OPTIONS和TRACE方法探测。**

  

  

 **测试Win2008 IIS7.0 ASP.NET开启**



config.ZhCN.POST.xml  失败

config.ZhCN.HEAD.xml  失败

config.ZhCN.GET.xml  失败

config.ZhCN.DEBUG.xml  失败

![]()

 **config.ZhCN.OPTIONS.xml   成功**

![]()

 **config.ZhCN.TRACE.xml   成功**

![]()



  

 **测试结论：**

 **1、开启ASP.NET可能修复无效（未重启或当前ASP.NET版本不够高）。**

 **2、IIS7 可使用OPTIONS和TRACE方法探测。**

 **  
**

 **  
**

  

 **测试Win10 IIS10.0 ASP.NET状态未知**

  

config.ZhCN.POST.xml 失败

config.ZhCN.HEAD.xml失败

config.ZhCN.GET.xml失败

config.ZhCN.DEBUG.xml失败

![]()

  

config.ZhCN.OPTIONS.xml 成功

![]()

config.ZhCN.TRACE.xml成功

![]()

  

  

 **测试结论：**

 **1、IIS10可使用OPTIONS和TRACE方法探测。**

  

  

  

 **测试Win2003 IIS6.0 ASP.NET未安装**

  

config.ZhCN.POST.xml 失败  

config.ZhCN.HEAD.xml  失败

config.ZhCN.GET.xml  失败

![]()

config.ZhCN.DEBUG.xml  成功

![]()

config.ZhCN.OPTIONS.xml  成功

![]()

config.ZhCN.TRACE.xml  成功

![]()

  

 **测试结论：**

 **1、IIS6 可使用DEBUG 、OPTIONS和TRACE方法探测。**

 **2、IIS6 使用DEBUG 、OPTIONS方法时存在ASA误报。**

  

  
  
  
 **0x10  不同版本IIS测试结论** **  
**

IIS6.0 无ASP.NET环境 可用 DEBUG 、OPTIONS和TRACE方法  

  

IIS7.0、IIS10开启及关闭ASP.NET环境 可用 OPTIONS TRACE  

  

总结：在IIS所有常见版本，通过OPTIONS和TRACE方法都可以猜解成功。

  

  

其他不确定的结论:

  

1、IIS1.0 - IIS6.0有ASP.NET环境，可用GET方法探测  【未验证】

2、IIS使用.Net4时不受影响（IIS7.0版本默认有.NET4） 【不适用】

  

  
  
 **0x11 衍生 不同服务器的漏洞检测**  

  

  *   *   * 

    
    
     IIS服务器   存在Apche服务器  不存在tomcat服务器  不存在

  

  

结论：短文件名漏洞仅存在与IIS服务器情况

  

  
  
  
 **0x12  衍生 短文件名直接访问测试**

  

  *   *   * 

    
    
    IIS服务 不支持    (ASP.NET开启、关闭相同)Apche服务器  支持Tomcat服务器  不支持

  

结论：Apache下可通过短文件名直接访问服务器文件

  
  
  
 **0x13  ** **通配符在短文件名猜测中的扩展**

  

测试环境：IIS7.0

FILE_1~1.TXT  存在的短文件名

FILE_N~1.TXT  不存在的短文件名

  

![]()

IIS7环境DEBUG方法不支持通配符探测

  

![]()

  

IIS7环境GET、HEAD方法不支持通配符探测

![]()

 **IIS7环境 OPTIONS方法支持通配符  ** **猜测成功返回404，猜测失败返回200**

  

![]()

 **Trace方法支持通配符猜测猜测成功返回404，猜测失败返回的是501**

  

  

 **通配符扫描结论：**

 **1、在IIS猜测短文件名的过程中，可以使用通配符"*"和"?"。**

  

  
  
 **0x14 通配符在短文件名猜解中的实际利用**  

攻击者结合使用通配符“*”和“?”辅助发送TRACE和OPTIONS等请求到IIS服务器，能够快速检测出存在的短文件名。

  

猜测思路：

  

  *   *   *   * 

    
    
    1、猜测/*~1*/是否存在2、遍历猜测/[A-Z 1-9 符号]*~1*/是否存在3、猜测/[A-Z 1-9 符号][ 1-5个?]~1*/。4、再逐步猜测每个字符....

  

![]()

  

IIS短文件猜解过程(参考)

  

 **注意：经过测试，通配符使用过程中，路径不加[/.aspx]后缀不影响扫描**

  

![]() ****

 **配置无/.aspx后缀扫描  
**

  
 **![]()**

 **有无/.aspx后缀不影响探测** ****

  
  
  
 **0x15 IIS短文件名推测长文件名**

 **  
**

IIS由于安全原因不支持短文件名访问，需要继续猜测出长文件名才可以在IIS上进行访问。

  

 **基本情况推断方案：（不建议）**

方案一：对未猜测完毕的字符继续进行Fuzz

使用dirbuster、burpsuite等工具即可

  

 **方案一：通过字典进行对比：（建议|未 **完全** 实现）**

1建立一个常见的目录文件单词表。

2判断字典中是否有以短文件作为子串的单词路径,使用高度匹配的单词路径进行访问测试。

  
  
 **  
**  
  
 **0x16 中文短文件名的猜测问题**  

1个中文字符相当于2个英文字符，故超过4个中文字会产生短文件名。

现有工具不支持爆破中文文件和中文文件夹的短文件名。

  
![]()  
  
  

 **字符对应URL编码:**

  *   *   *   *   * 

    
    
     中文中文中文.txt ==> %E4%B8%AD%E6%96%87%E4%B8%AD%E6%96%87%E4%B8%AD%E6%96%87.txt%E4%B8%AD  ==> 中%E6%96%87 ==> 文%3f ==> ?%7e ==> ~

 **  
** **![]()**

 **中文文件访问** ****

  

![]()

 **中文文件名可猜测,但是更复杂** ****

 **  
**

 **结论：**

 **每个中文的是一个整体，需要当作一个字符看待，爆破也只能一个个中文的爆破。  ** **因此爆破难度比爆破字母数字的文件更复杂.**

  
  
 **0x17  部分后缀名文件无法访问** **  
**

不可访问下载的后缀名：.conf、.sql、.config. 【未全面测试】

  

 **** **![]()** **  
** **![]()**  
  
  
  
 **0x18 Summary  总结**  
本文主要是深入学习研究了IIS短文件名漏洞的原理|挖掘|利用, 其中最大的不足是, 没有实现一个更完成的漏洞深入利用工具. 最新的 shortscan
工具,好像实现了该功能,但截至本文完成时,我没有对该工具进行实际测试.  

  

参考文章:

  *   *   *   * 

    
    
    浅析Windows短文件名漏洞https://mp.weixin.qq.com/s/RLWXOHh_8cuudGM5UD8SEA  
    由于本文时间跨度较大, 更多参考文章一时无法找齐, 未详细描述.

  
  
  
 **0x19  免责声明**  

  

在学习本文技术或工具使用前，请您务必审慎阅读、充分理解各条款内容。

  

1、本团队分享的任何类型技术、工具文章等文章仅面向合法授权的企业安全建设行为与个人学习行为，严禁任何组织或个人使用本团队技术或工具进行非法活动。

  

2、在使用本文相关工具及技术进行测试时，您应确保该行为符合当地的法律法规，并且已经取得了足够的授权。如您仅需要测试技术或工具的可行性，建议请自行搭建靶机环境，请勿对非授权目标进行扫描。

  

3、如您在使用本工具的过程中存在任何非法行为，您需自行承担相应后果，我们将不承担任何法律及连带责任。  

  

4、本团队目前未发起任何对外公开培训项目和其他对外收费项目，严禁任何组织或个人使用本团队名义进行非法盈利。

  

5、本团队所有分享工具及技术文章，严禁不经过授权的公开分享。

  

如果发现上述禁止行为，我们将保留追究您法律责任的权利，并由您自身承担由禁止行为造成的任何后果。

  

  

END

  
  

如您有任何投稿、问题、建议、需求、合作、请后台留言NOVASEC公众号！

![]()

或添加NOVASEC-余生 以便于及时回复。  

![]()

  

感谢大哥们的对NOVASEC的支持点赞和关注

加入我们与萌新一起成长吧！

  

 **本团队任何技术及文件仅用于学习分享，请勿用于任何违法活动，感谢大家的支持！ ！**

  
  

  

#  

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

