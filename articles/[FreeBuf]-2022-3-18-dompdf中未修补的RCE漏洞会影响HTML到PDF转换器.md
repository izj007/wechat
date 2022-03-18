#  dompdf中未修补的RCE漏洞会影响HTML到PDF转换器

LouisJack  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 国内网络安全行业门户

____

__

收录于话题

![](https://gitee.com/fuli009/images/raw/master/public/20220318193528.png)

研究人员在“dompdf”（一种基于php的HTML到PDF的转换器）中发现了一个未修补的安全漏洞，如果该漏洞被成功利用，可能会导致某些配置中的远程代码被执行。

“通过将 CSS 注入到 dompdf 处理的数据中，它可以存储在一个.php缓存文件扩展名的恶意字段中，之后可以通过访问web以执行”，Positive
security的研究人员Maximilian Kirchmeier 和Fabian Bräunlein在其发布的报告中如此写道。

换而言之，该漏洞允许恶意方将扩展名为.php的字段文件上传到web服务器，然后利用XSS漏洞将HTML注入到web页面中，最后将其呈现为PDF。这就意味着攻击者可能会导航到上传的php脚本，从而有效地使得远程代码在服务器上执行。

![](https://gitee.com/fuli009/images/raw/master/public/20220318193540.png)

对于那些需要根据用户提供的数据（如票务购买和其他收据）在服务器端生成pdf的网站来说，这可能会导致 **严重后果**
，特别是当输入接口没有充分扫描杀毒以减少XSS缺陷的时候，或者是当程序库安装在公共可访问的目录中的时候。

根据GitHub上的统计数据，dompdf在将近59250个存储库中使用，这使得它成为在php编程语言中生成pdf的流行数据库。

1.2.0及其更早版本的dompdf位于web可访问目录中，并启用了“$isRemoteEnabled”设置，这显然是非常容易遭到攻击的。但是，即使将此选项设置为false，
**该数据库的0.8.5及之前版本也会受到影响** 。

尽管早在2021年10月5日开源项目维护者就收到了该漏洞的报告，但对于预计何时修复却仍然毫无头绪。“安全漏洞通常是由于设计决策基于对底层或互联组件的错误假设而产生的”，研究人员解释道。“如果可能的话，将dompdf
更新到最新版本并关闭$isRemoteEnabled就可以免于其扰。  

 **参考来源**

https://thehackernews.com/2022/03/unpatched-rce-bug-in-dompdf-project.html

![](https://gitee.com/fuli009/images/raw/master/public/20220318193541.png)  
  

精彩推荐

  
  
  
  
  
 ** **![]()****  
  
[![](https://gitee.com/fuli009/images/raw/master/public/20220318193542.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247487250&idx=1&sn=ba8fd6364620305f829b909d8465b7b2&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20220318193543.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247487226&idx=1&sn=ae9abde9c0e9ebb6ec8f1cd74fc71a86&scene=21#wechat_redirect)  
[![](https://gitee.com/fuli009/images/raw/master/public/20220318193544.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247487156&idx=1&sn=f4346a01c147c034245d3d29ba003696&scene=21#wechat_redirect)
** ** ** ** ** **
**![](https://gitee.com/fuli009/images/raw/master/public/20220318193545.png)**************

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

阅读原文

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment5c9a6b.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

dompdf中未修补的RCE漏洞会影响HTML到PDF转换器

最多200字，当前共字

__

发送中

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

