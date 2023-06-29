#  通达OA漏洞检测工具-TongdaScan_go

Fu5r0dah  [ 夜组安全 ](javascript:void\(0\);)

**夜组安全** ![]()

微信号 NightCrawler_Team

功能介绍 "恐惧就是貌似真实的伪证" NightCrawler Team(简称:夜组)主攻WEB安全 | 内网渗透 | 红蓝对抗 | 代码审计 |
APT攻击，致力于将每一位藏在暗处的白帽子聚集在一起，在夜空中划出一道绚丽的光线！

____

___发表于_

收录于合集 #安全工具 44个

免责声明

由于传播、利用本公众号夜组安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号夜组安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

朋友们现在只对常读和星标的公众号才展示大图推送，建议大家把 **夜组安全** “ **设为星标** ”，否则可能就看不到了啦！

![](https://gitee.com/fuli009/images/raw/master/public/20230629083331.png)![](https://gitee.com/fuli009/images/raw/master/public/20230629083331.png)

 **安全工具**

![](https://gitee.com/fuli009/images/raw/master/public/20230629083331.png)![](https://gitee.com/fuli009/images/raw/master/public/20230629083331.png)

 **01**

 **工具介绍** ****

通达OA漏洞检测工具![](https://gitee.com/fuli009/images/raw/master/public/20230629083334.png)

  

 **02**

 **支持检测漏洞**

  *   *   *   *   *   *   *   *   *   *   * 

    
    
       通达OA v2014 get_contactlist.php 敏感信息泄漏   通达OA v2017 video_file.php 任意文件下载   通达OA v2017 action_upload.php 任意文件上传   通达OA v2017 login_code.php 任意用户登录   通达OA v11 login_code.php 任意用户登录   通达OA v11.5 swfupload_new.php SQL注入   通达OA v11.6 report_bi.func.php SQL注入   通达OA v11.8 api.ali.php 任意文件上传漏洞   通达OA v11.8 gateway.php 远程文件包含漏洞   通达OA v11.6 print.php未授权删除auth.inc.php导致RCE   通达OA v11.10 getdata 任意文件上传

 **03**

 **漏洞检测**

使用scan参数时，不指定任何vulnID，则自动检测所有漏洞

  * 

    
    
    TongdaScan_go scan -u http://1.1.1.1

![](https://gitee.com/fuli009/images/raw/master/public/20230629083335.png)

指定漏洞ID

  * 

    
    
    TongdaScan_go scan -u http://1.1.1.1 -i Td03

![](https://gitee.com/fuli009/images/raw/master/public/20230629083336.png)

 **04**

 **漏洞利用**

  * 

    
    
     TongdaScan_go exp -u http://1.1.1.1 -i Td03

![](https://gitee.com/fuli009/images/raw/master/public/20230629083337.png)

代理

  * 

    
    
    TongdaScan_go scan -u http://1.1.1.1 -i Td01 -s http://127.0.0.1:8080

 **05**

 **工具下载**

 **点击关注下方名片** **进入公众号**

 **回复关键字【 230629** **】获取** **下载链接**

  

 **06**

 **往期精彩**

[ ![](https://gitee.com/fuli009/images/raw/master/public/20230629083338.png)

一个终身免费的安全 _武器库_

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247485483&idx=1&sn=636d5c32271c9d5b6faa861508d32086&chksm=c3684cd3f41fc5c543a47a019cba9614fade2e06b34dcf237f2877b4ddefe80a0b3cb416697f&scene=21#wechat_redirect)  
[ ![](https://gitee.com/fuli009/images/raw/master/public/20230629083339.png)

Burpsuite验证码爆破插件，更新了captcha-killer

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487235&idx=1&sn=3fd962db522337b89430672de283850f&chksm=c3684bfbf41fc2ed7697f54637296f81061b25e3502acaa062534a73223e5d7855c510bc3dd0&scene=21#wechat_redirect)  
[ ![](https://gitee.com/fuli009/images/raw/master/public/20230629083340.png)

LSTAR - CobaltStrike 综合后渗透插件

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487228&idx=1&sn=5150924c2600d0e66297f4acb9384688&chksm=c3684a04f41fc312bcbf6342c590933a7489442eed42b2ae9d165a224c8fe0c6c30b5d9769e3&scene=21#wechat_redirect)

![](https://gitee.com/fuli009/images/raw/master/public/20230629083341.png)

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

