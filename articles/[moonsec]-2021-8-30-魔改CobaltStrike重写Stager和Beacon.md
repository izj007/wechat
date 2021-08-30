#  魔改CobaltStrike重写Stager和Beacon

原创 快乐鸡哥  [ moonsec ](javascript:void\(0\);)

**moonsec** ![]()

微信号 moon_sec

功能介绍 暗月博客

____

__

收录于话题 #渗透测试 免杀 红队 ,23个

  

  

**魔改CobaltStrike重写Stager和Beacon**![](https://gitee.com/fuli009/images/raw/master/public/20210830125945.png)
** _ **1**_** **一、概述**

这次我们一起用C#来重写stager及其Beacon中的大部分常用功能，帖子主要介绍该项目的运行原理（LolBinsStagerBeacon）及相应的功能介绍及展示。LolBins部分是由GadgetToJs使Stager转换为js、vba、hta文件后，再结合相应的csript、mshta等程序来运行；Stager功能包括从网络中拉取Beacon的程序集并在内存中加载及AMSIBypass；Beacon部分主要有包括正常上线、文件管理、进程管理、令牌管理、结合SysCall进行注入、原生端口转发、关ETW等一系列功能。

项目地址：https://github.com/mai1zhi2/SharpBeacon

项目基于.net4.0，暂支持cs4.1（更高版本待测试），感谢M大、BGW师傅、SharpSploit、Geason的分享。另因最近出去广州找工作没时间弄，就暂时写到这里，开发进度比较赶致使封装不是很好、设计模式也没有用，但每个实现功能点都有较详细注释，等后续工作安定后会进行重构及完善更多功能。若有错误之处还请师傅指出，谢谢大家。

  

 ** _ **2**_** **二、LolBins**

#  

LOLBins，全称“Living-Off-the-LandBinaries”，直白翻译为“生活在陆地上的二进“，我大概将其分为两大类：

1、带有Microsoft签名的二进制文件，可以是Microsoft系统目录中二进制文件。

2、第三方认证签名程序。

LolBins的程序除了正常的功能外，还可以做其他意想不到的行为。在APT或红队渗透常用，常见于海莲花等APT组织所使用。下图是较常见的LolBins，还有很多就不一一列出了：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125946.png)

而GadgetToJS项目则可以把源码cs文件动态编译再base64编码后，保存在js、vba、vbs、hta文件，而在其相关文件中文件利用了当BinaryFormatter属性在进行反序列化时，可以触发对Activator.CreateInstance()
的调用，，从而实现.NET 程序集加载/执行。

![](https://gitee.com/fuli009/images/raw/master/public/20210830125947.png)

但这需要在.net程序集中把相应的功能写在默认/公共构造函数，这样才能触发.NET 程序集执行。下面以实例程序为例：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125948.png)

在相应文件夹下执行如下命令：

.\GadgetToJScript.exe -w js -c Program.cs -d  System.Windows.Forms.dll -b -o
gg

其中命令参数解析如下：

-wjs表示所生成的是js文件，可以生成其他形式的文件

-cProgram.cs是所选择的cs文件

-d System.Windows.Forms.dll  cs文件所用到的dll

-b 会在js文件中的引入第一个stager，因为当在.NET4.8+的版本中引入了旁路类型检查控件，默认值为false，如果所生成的脚本要在.NET4.8+的环境中运行，则设置为true（\--Bypass/-b）。生成的stager1就是bypass这个检查的。

-ogg生成文件名

生成js、hta、vbs等文件后默认是会被杀的：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125949.png)

而我们只需要简单修改下单引号为/就行了:

![](https://gitee.com/fuli009/images/raw/master/public/20210830125950.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830125951.png)

最后执行所生成的js或hta：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125952.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830125953.png)

  

 ** _ **3**_** **三、Stager**

#  

Stager部分的功能可以包括下图几项：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125954.png)

我主要实现了从网络中拉取Beacon的程序集并在内存中加载及AMSIBypass，沙箱及虚拟机检测的方式有挺多方式的，师傅可以自行添加。

拉取程序集及内存加载这个较为简单，就不细说了：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125955.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830125956.png)

下面说说bypassAMSI，这里一开始找的不是AmsiScanBuffer，而是找DllCanUnloadNow的地址：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125957.png)

然后再通过相关的硬编码找到AmsiScanBuffer后，再进行相应的patch：

![](https://gitee.com/fuli009/images/raw/master/public/20210830125958.png)

  

 ** _ **4**_** **四、Beacon**

#  

Beacon部分主要有包括正常上线、文件管理、进程管理、令牌管理、结合SysCall进行注入、原生端口转发、关ETW等一系列功能。

![](https://gitee.com/fuli009/images/raw/master/public/20210830125959.png)

  

##  **4.1 文件管理**

先从文件管理部分说，包含了cp、mv、upload、download、filebrowse、rm、mkdir上述这七个功能点：

Cp:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130000.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830130001.png)

Mv:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130002.png)

Upload:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130004.png)

Download:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130005.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830130006.png)

Filebrowse:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130007.png)

rm:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130008.png)

mkdir

![](https://gitee.com/fuli009/images/raw/master/public/20210830130009.png)

  

##  **4.2 进程部分**

进程部分，已完成的有run、shell、execute、runas、kill，未完成的有runu:

Run:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130010.png)

shell:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130011.png)

execute:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130012.png)

runas:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130013.png)

ps:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130014.png)

kill:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130015.png)

##  **4.3 令牌权限**

令牌权限部分，已完成的有getprivs、make_token、steal_token、rev2self：

Getprivs:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130016.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830130017.png)

make_token：测试时在make_token后执行了

 **cmd.exe /C dir \\\10.10.10.165\C$**

![](https://gitee.com/fuli009/images/raw/master/public/20210830130018.png)

steal_token：测试时在steal_token后执行了whoami

![](https://gitee.com/fuli009/images/raw/master/public/20210830130020.png)

rev2self：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130021.png)

##  **4.4 端口转发**

端口转发部分，已完成的有rportfwd、rportfwdstop：

Rportfwd，注意这里端口转发teamserver只返回了本地需要绑定的端口，没有返回需转发的ip和port。

在192.168.202.180:22222上新建msf监听：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130022.png)

在本机地址192.168.202.1的23456端口转发到上述msf的监听

![](https://gitee.com/fuli009/images/raw/master/public/20210830130023.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830130024.png)

本地访问23456端口：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130025.png)

另一个网段访问23456端口：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130026.png)

rportfwd stop：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130027.png)

## 4.5注入部分

注入部分，cs的shinject、dllinject、inject都用来远程线程注入，我个人机器是win10x64
1909,shellcode是用cs的64位c#shellcode，被注入的程序是64位的calc.exe，程序返回的NTSTSTUS均为SUCCESS，且shellcode均已注入在相应的程序中，并新建出线程进行执行，但最后calc.exe都崩了，有点奇怪呀：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130029.png)

申请rwx内存空间存放shellcode后并在所执行shellcode下断:

![](https://gitee.com/fuli009/images/raw/master/public/20210830130030.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210830130031.png)

执行NtCreateThreadEx(),被注入的calc.exe新建线程执行此shellcode：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130033.png)

最后跑起来报的c05，但分配的内存属性是rwx的：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130034.png)

##  **4.6 杂项部分**

杂项部分，已完成有sleep、pwd、exit、setenv、drives、cd：

Sleep：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130035.png)

exit：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130036.png)

setenv：

![]()

  

drives：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130037.png)

Pwd：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130038.png)

cd：

![](https://gitee.com/fuli009/images/raw/master/public/20210830130039.png)

  

 ** _ **5**_** **五、完善及改进**

后续需要改进的地方还有很多，有如下几点：

1、该封装好就封装好，该用设计模式就用

2、目前rsa密钥是pem方式就用了BouncyCastle库，要用回Exponent和 Modulus

3、更多的注入方式，APC、傀儡进程等

4、更多的通信协议，如DNS、ICMP

5、支持spawn**，因为当执行spawn和job后，teamserver端会回传相应的dll，要改ts端

6、更多的功能，如mimi、keylogger、portscan、加载pe等

最后谢谢大家观看。

 ** _ **6**_** **关注  
**

 **关注本公众号 不定期更新文章和视频**

 **欢迎前来关注**  

![](https://gitee.com/fuli009/images/raw/master/public/20210830130040.png)

  

  

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

魔改CobaltStrike重写Stager和Beacon

最多200字，当前共字

__

发送中

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

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

