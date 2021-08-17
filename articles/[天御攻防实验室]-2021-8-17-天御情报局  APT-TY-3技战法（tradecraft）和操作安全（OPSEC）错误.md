##  天御情报局 | APT-TY-3技战法（tradecraft）和操作安全（OPSEC）错误

Zer0d0y  [ 天御攻防实验室 ](javascript:void\(0\);)

**天御攻防实验室** ![]()

微信号 TianyuLab

功能介绍 专注威胁感知、威胁猎杀、高级威胁检测，Adversary Simulation、Adversary Detection、Adversary
Resilience

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20210817185435.png)

技战法（tradecraft）：

  

1.没有两个主机与同一个C2通信，具有相同的恶意软件哈希，具有相同的后门文件名。

  

2.他们通过.eml文件中的后门传输恶意软件并使用Outlook Express提取，然后以DVD的块大小将exfil发送到受害者组织所在国家/地区的服务器

  

3.攻击者的后门总是在命令行中使用“密码（通常是4个字符）”来执行（因此阻止了在沙盒中的执行和没有经验的恶意软件分析师），并将配置存储在注册表Key中 -
或复制和重命名一个可执行文件（通常是ping.exe、mtxex.dll），配置在文件的随机位置。从二进制文件外部存储配置的另一个好处 -
如果文件上传到VT - 也无法找到C2服务器

  

OPSEC错误：

1.奇怪的是 - APT-
TY-3在TTPs中犯的错误是在配置注册表键的名称上（他们只用了3-4个），后门程序在UA字符串上有一个错误，而且他们的后门程序有足够常见的模式来编写一个snort规则。

  

2.对第二阶段C2使用相同的自签名SSL证书 # 大约5年也不是他们最伟大的技术

往期精选

  

围观

[威胁猎杀实战（六）：横向移动攻击检测](http://mp.weixin.qq.com/s?__biz=MzU0MzgyMzM2Nw==&mid=2247483843&idx=1&sn=b3c26b8593f0cbe2b02c896df7b0f7f9&chksm=fb04c2abcc734bbdaebb1dcae8697d8a5e66e7ea1f458e2695d3da065211c3958f97b8c12586&scene=21#wechat_redirect)  

  

热文

[全球“三大”入侵分析模型](http://mp.weixin.qq.com/s?__biz=MzU0MzgyMzM2Nw==&mid=2247483996&idx=1&sn=cd167b205384bf9b1e45a6f7aca0f17a&chksm=fb04c134cc7348226746d9514ad9b7bae3d0654ac4b0103cbe8dc590f96c3423005d60892540&scene=21#wechat_redirect)  

  

热文

[实战化ATT&CK：威胁情报](http://mp.weixin.qq.com/s?__biz=MzU0MzgyMzM2Nw==&mid=2247483921&idx=1&sn=1e778a6ec60df19a148f1a44cb8c2e45&chksm=fb04c179cc73486f818f1bb96ea58fee96a243f77538a9b164738f78c2ccdb37756ffb5f1659&scene=21#wechat_redirect)  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210817185446.png)

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

天御情报局 | APT-TY-3技战法（tradecraft）和操作安全（OPSEC）错误

最多200字，当前共字

__

发送中

写下你的留言

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

