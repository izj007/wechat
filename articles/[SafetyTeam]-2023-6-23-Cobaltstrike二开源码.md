#  Cobaltstrike二开源码

原创 luckone  [ SafetyTeam ](javascript:void\(0\);)

**SafetyTeam** ![]()

微信号 luck_sec

功能介绍 安全开发，漏洞研究，漏洞挖掘

____

___发表于_

收录于合集

> 微信公众号： **SafetyTeam**  
>  为国之安全而奋斗，为信息安全而发声！  
> 如有问题或建议，请在公众号后台留言  
> 如果你觉得本文对你有帮助，欢迎关注本公众号，后期会不定时公布二开细节  
>  **如果你觉得对你有帮助，欢迎赞赏 [1]**

### 声明

>
> 声明：本公众号所分享内容仅用于网安爱好者之间的技术讨论，`禁止用于违法途径，所有渗透都需获取授权`！否则需`自行承担`，`本公众号及原作者不承担相应的后果`.

### 本项目的宗旨

在大家的日常渗透过程中，Cobaltstrike是攻击队常用的团队协作工具之一  
但人怕出名猪怕壮，有了攻也必有防，攻防对抗是一个过程。Cobaltstrike在历史的发展中，一些通用指纹和漏洞被防守方发掘了出来，便很容易被拦截、溯源甚至反制

`本项目，旨在建立一个长期维护的Cobaltstrike魔改项目，为各位红队师傅减小因CS的“历史问题”上被暴露、反制以及溯源的可能性`

### 本项目的魔改内容

    
    
    修改监听不返回任何数据  
    添加teamserver认证白名单限制登陆IP  
    去除checksum8特征  
    默认证书已替换  
    防止stage被扫  
    去除listenerConfig特征字符串  
    修改配置加解密的xorkey  
    去除beaconeye特征  
    修复CVE-2022-39197  
    修复错误路径泄漏stage  
    增加谷歌动态密码(身份验证器)  
    修复CS4.5的foreign派生bug  
    添加中文汉化翻译  
    防止CS反制，修改.aggressor.prop文件  
    

### 项目获取方式

cs4.5二开领取：

关注`luck_sec`公众号，在后台回复关键词`(cs)`即可领取

cs4.7源码领取：

关注`luck_sec`公众号，在后台回复关键词`(4.7)`即可领取  

### 反馈和交流

欢迎各位师傅进群交流，我们也会长期维护

添加微信回复`加群`

  * 

    
    
    微信号：zero-b0y

`如果感觉不错，欢迎给项目点个Star或者分享给其他师傅`

  

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

