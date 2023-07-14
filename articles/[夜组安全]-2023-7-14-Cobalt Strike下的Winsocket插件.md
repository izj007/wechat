#  Cobalt Strike下的Winsocket插件

[ 夜组安全 ](javascript:void\(0\);)

**夜组安全** ![]()

微信号 NightCrawler_Team

功能介绍 "恐惧就是貌似真实的伪证" NightCrawler Team(简称:夜组)主攻WEB安全 | 内网渗透 | 红蓝对抗 | 代码审计 |
APT攻击，致力于将每一位藏在暗处的白帽子聚集在一起，在夜空中划出一道绚丽的光线！

____

___发表于_

收录于合集 #安全工具 53个

免责声明

由于传播、利用本公众号夜组安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号夜组安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

朋友们现在只对常读和星标的公众号才展示大图推送，建议大家把 **夜组安全** “ **设为星标** ”，否则可能就看不到了啦！

![](https://gitee.com/fuli009/images/raw/master/public/20230714180139.png)![](https://gitee.com/fuli009/images/raw/master/public/20230714180139.png)

 **Cobalt Strike**

![](https://gitee.com/fuli009/images/raw/master/public/20230714180139.png)![](https://gitee.com/fuli009/images/raw/master/public/20230714180139.png)

 **01**

 **插件介绍**

Cobalt Strike的Winsocket插件，用于使用Winsocket与受害者进行通信，而不是传统的方式。

 **02**

 **插件使用**

 **Client.c**

  

这是一个客户端，从服务器接收命令，以子进程执行命令，解析其输出并将其发送回服务器。打开解决方案（.sln）文件，使用Visual Studio编译代码。

  

 **Server.c**  

将BOF脚本加载到Cobalt Strike，它连接到客户端的Winsocket服务器，发送命令并接收返回的响应。

  

To compile it, use `make`:

  * 

    
    
    cd Server && make

Then load `socket.cna` to Cobalt Strike. To use it, run the following command:

  * 

    
    
    socky <command>

 **注意：带有空格的命令（例如：whoami /all）必须用引号括起来。**  

 **03**

 **使用教程**

 **04**

 **工具下载**

 **点击关注下方名片** **进入公众号**

 **回复关键字【 230711** **】获取** **下载链接**

  

 **05**

 **往期精彩**

[ ![]()

值得加入的安全WIKI，涉及30个方向的资料

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487281&idx=1&sn=e5cedf2a72956941381803471b4edb5f&chksm=c3684bc9f41fc2dfbdd38dbd1944c86e191a1d257befaa894e824b168c1dc149c8980c673080&scene=21#wechat_redirect)  
[ ![](https://gitee.com/fuli009/images/raw/master/public/20230714180142.png)

一个终身免费的安全武器库

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487269&idx=1&sn=58a73b9c68cc3e50db835b23c7a27e1a&chksm=c3684bddf41fc2cb8e9355095e181e797522178ae96b99ebd81c6b998406ff7a1bda9d1db334&scene=21#wechat_redirect)  
[ ![](https://gitee.com/fuli009/images/raw/master/public/20230714180144.png)

康威视综合安防管理平台任意文件上传，一键getshell

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487396&idx=1&sn=8562fe48d4c0101a363eb2da7fdc650b&chksm=c3684b5cf41fc24aee672ba32a1cd7ff7a66148f56fbf6776ed098bcdd70d3c8c14c57a0f03d&scene=21#wechat_redirect)
![](https://gitee.com/fuli009/images/raw/master/public/20230714180145.png)  

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

