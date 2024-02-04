#  数据库注入工具 SqlmapXPlus ！增强MSSQL后渗透利用方式！

[ 天擎攻防实验室 ](javascript:void\(0\);)

**天擎攻防实验室** ![]()

微信号 gh_2fb077348503

功能介绍 天擎攻防实验室，致力于分享最新漏洞情报以及复现!

____

___发表于_

以下文章来源于赛博大作战 ，作者coolcat

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Qxc8fCfTYXUKNleEa4MalgQjYLrj8Wiba2PORelrnPjw/0)
**赛博大作战** .

致力于打造友好网络安全交流圈子，包括漏洞情报跟踪、漏洞分析复现、审计安全开发学习、资源分享...

赛博大作战中的技术文章仅供参考，此文所提供的信息只为网络安全人员对自己所负责的网站、服务器等（包括但不限于）进行检测或维护参考，未经授权请勿利用文章中的技术资料对任何计算机系统进行入侵操作。利用此文所提供的信息而造成的直接或间接后果和损失，均由使用者本人负责。本文所提供的工具仅用于学习，禁止用于其他！！

1基本介绍  

 **SqlmapXPlus 基于 S qlmap**，对经典的数据库漏洞利用工具进行二开，参照各种解决方法，增加MSSQL数据库注入的利用方式。

目前已完成部分二开， **包括**
**ole、xpcmdshell两种文件上传、内存马上传、clr安装功能，能够实现mssql注入场景下的自动化注入内存马、自动化提权、自动化添加后门用户、自动化远程文件下载、自动化shellcode加载功能。**

![]()

先说场景，在众多的地区性攻防演练中，SQL
Server数据库堆叠注入仍有较高的爆洞频率，但因为一些常见的演练场景限制，如不出网、低权限、站库分离、终端防护、上线困难、权限维持繁琐等，公开的漏洞利用工具难满足我们的需求。

场景举例：默认的--os-shell依赖于xp_cmdshell的存储过程，执行命令的权限默认为数据库的权限

![]()

权限低受不了？试试system权限！  

![]()

以及 mssql站库不分离，找路径、for循环写webshell？curl上线cs？其实都可以，怎么好用怎么来，或者试试这个

https://github.com/co01cat/SqlmapXPlus

![]()

新增功能介绍：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #  开启 clr 功能--enable-clr#  关闭 clr 功能--disable-clr# 通过 xp_cmdshell 实现的文件上传功能 ，作用为将本地文件上传到远程服务器--xp-upload localfile --file-dest remotefile# 通过 ole 实现的文件上传功能 ，作用为将本地文件上传到远程服务器，默认将文件上传至c:\windows\tasks\目录下--ole-upload#  通过 xp_cmdshell 实现的clr安装方式--install-clr1#  通过 ole 实现的clr安装方式--install-clr2#  进入clr-shell命令交互模式--clr-shell#  通过 xp_cmdshell 实现的HttpListener内存马上传方式--sharpshell-upload1#  通过 ole 实现的HttpListener内存马上传方式--sharpshell-upload

2安装CLR和上传内存马

 **1.关于
clr利用功能和内存马的功能实现：**clr由当前目录下的clrdatabase.dll实现，内存马由当前目录下的listen.tmp.txt实现，如果需要使用自己实现的clr和listen内存马可以通过替换该目录下的两个文件实现  

![]()

 **2.安装clr：** 进入clr-shell模式前必须先安装clr的存储过程，提供了两种安装的方式

 **\--install-clr1** 适合在xp_cmdshell没有被拦截的情况下使用，即使用--os-shell可以成功执行命令并获取回显。  

 **- -install-clr2**适合在xp_cmdshell被拦截的情况下使用，即使用--os-shell无法执行命令。

完整命令：python sqlmap.py -u/-r xxxx **\--install-clr1** 或  **- -install-clr2 **

![]()

无论采用哪种方式，在安装流程结束后都会显示Install clr successful!

 **3.上传HttpListener内存马：** 如果需要使用内存马注入功能，就必须先上传内存马文件到目标机器 ****

 **\--sharpshell-upload1  **适合在xp_cmdshell没有被拦截的情况下使用，即使用--os-
shell可以成功执行命令并获取回显。  

 **\--sharpshell-upload2  **适合在xp_cmdshell被拦截的情况下使用，即使用--os-shell无法执行命令。

完整命令：python sqlmap.py -u/-r xxxx **\--sharpshell-upload1** 或  **\--sharpshell-
upload2**

![]()

内存马的上传流程结束后会显示Uploaded successfully!的字样。

3关于 **clr-shell的命令执行模式**

在前面步骤都完成的情况下，输入 python sqlmap.py -u/-r xxxx  **\--clr-shell  **进入clr命令执行模式
****

clr-shell模式下的内置功能介绍：  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     clr_rdp # 开启RDPclr_adduser # 添加系统用户clr_exec # 命令执行clr_efspotato # 提权模块clr_memshell # 内存马clr_download # 远程文件下载clr_rm # 指定文件删除clr_cd # 切换目录clr_ping # pingclr_scloader # 直接shellcode加载clr_scloader1 # 落地的shellcode加载clr_scloader2 # 落地的shellcode加载

 **1\. 命令执行目前仅clr_exec和clr_efspotato执行是有回显的 ：**需要注意的是，因为获取命令执行后回显的权限问题，在进入clr-
shell模式 后 应该先执行 clr_exec whoami获取回显，再执行其他命令，否则可能导致回显无法正常获取

![]()

 **2\. 内存马注入：** 内存马注入需要system权限，利用efspotato的方式注入，如果执行clr_efspotato
whoami可以回显system权限就不会有问题

在clr-shell的命令执行模式中，输入  **clr_memshell** 即可实现注入！

![]()

此时尝试访问url，内存马默认上在对应服务器80端口，f12会存在inject success的字样

![]()

![]()

https://github.com/AntSwordProject/antSword/tree/v2.2.x 使用该版本的蚁剑来进行连接内存马路径  

![]()

![]()

我们成功获取了一个webshell！

 **3.远程文件下载** ：clr_download 远程路径 目标本地路径

![]()

 **4.shellcode加载功能说明：**  

首先，使用cobaltstrike生成payload.bin文件  

![]()

![]()

然后使用Encrypt.py文件对payload.bin进行处理，-f 指定cs生成的payload.bin，-k 自定义的密钥

最终会输出一个base64的payload，我们可以将这个payload写在一个txt文件中，存储在sqlmap的目录下如123.txt

![]()

![]()

 **上线方式一** ，适合POST请求情况下的堆叠注入： **clr_scloader base64的payload 自定义密钥**  

![]()

直接加载，成功上线CS ****  

 **上线方式二** ，加载落地的任意后缀payload文件上线，适合POST、GET情况下的注入： **clr_scloader1
目标机器上payload文件路径  自定义密钥**

先将payload上传到目标机器

 **![]()**

运行加载命令，成功上线CS ****

![]()

 **上线方式三** ，加载落地的原始payload.bin文件上线，适合POST、GET情况下的注入： **clr_scloader2
目标机器上payload文件路径**

先将payload上传到目标机器

![]()

![]()

运行加载命令，成功上线CS！  

3  
最后

趁着假期前的小空闲改写的工具，如果有好的建议欢迎加入交流群大家一起交流技术，2024年，希望是个好年，希望大家都能过得更好！

![]()

 **主要参考内容 ：                                                      **
https://github.com/sqlmapproject/sqlmap
https://github.com/uknowsec/SharpSQLTools
https://github.com/Anion3r/MSSQLProxy
https://mp.weixin.qq.com/s/X0cI85DdB17Wve2qzCRDbg
https://yzddmr6.com/posts/asp-net-memory-shell-httplistener/  ......

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 数据库注入工具 SqlmapXPlus ！增强MSSQL后渗透利用方式！

[ 天擎攻防实验室 ](javascript:void\(0\);)

轻触阅读原文

![]()

天擎攻防实验室

赞 分享 在看

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

