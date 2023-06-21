#  新一代子域嗅探工具 -- SubAps

Magic-Weapon  [ LemonSec ](javascript:void\(0\);)

**LemonSec** ![]()

微信号 lemon-sec

功能介绍 每日发布安全资讯～

____

___发表于_

收录于合集 #网络安全 166个

**0x01 工具介绍**

SubAps是新一代子域嗅探工具，可以全网爬虫获取子域，以数据库作为字典进行计次，提高准确率和效率。![](https://gitee.com/fuli009/images/raw/master/public/20230621085627.png)  

 **0x02 安装与使用**

  
1、主动扫描指的是通过从数据库读取Topxx（指定）数量的字域名前缀进行拼接，然后进行域名A记录的识别，对成功识别的进行站点访问，如果可以访问则提交至被动扫描进行爬虫嗅探并且将子域前缀入库，如果数据库存在则Conut次数+1。2、被动扫描指的是对可访问的站点进行访问获取页面源码，通过页面源码提取Url/域名进行关键词模糊匹配（关键词是自动提取的），如果模糊匹配到关键词则进行访问获取页面源代码并且将子域前缀入库，如果数据库存在则Conut次数+1，重复操作，一直到无法提取到新的域名为止（关键词模糊匹配不到）。![](https://gitee.com/fuli009/images/raw/master/public/20230621085633.png)

0x03 项目链接下载

  
https://github.com/Security-Magic-Weapon/SubAps  
 **侵权请私聊公众号删文**  

  **热文推荐  ** ** **

  * [蓝队应急响应姿势之Linux](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523380&idx=1&sn=27acf248b4bbce96e2e40e193b32f0c9&chksm=f9e3f36fce947a79b416e30442009c3de226d98422bd0fb8cbcc54a66c303ab99b4d3f9bbb05&scene=21#wechat_redirect)  

  * [通过DNSLOG回显验证漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523485&idx=1&sn=2825827e55c1c9264041744a00688caf&chksm=f9e3f3c6ce947ad0c129566e5952ac23c990cf0428704df1a51526d8db6adbc47f998ee96eb4&scene=21#wechat_redirect)  

  * [记一次服务器被种挖矿溯源](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523441&idx=2&sn=94c6fae1f131c991d82263cb6a8c820b&chksm=f9e3f32ace947a3cdae52cf4cdfc9169ecf2b801f6b0fc2312801d73846d28b36d4ba47cb671&scene=21#wechat_redirect)  

  * [内网渗透初探 | 小白简单学习内网渗透](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523346&idx=1&sn=4bf01626aa7457c9f9255dc088a738b4&chksm=f9e3f349ce947a5f934329a78177b9ce85e625a36039008eead2fe35cbad5e96a991569d0b80&scene=21#wechat_redirect)  

  * [实战|通过恶意 pdf 执行 xss 漏洞](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523274&idx=1&sn=89290e2b7a8e408ff62a657ef71c8594&chksm=f9e3f491ce947d8702eda190e8d4f7ea2e3721549c27a2f768c3256de170f1fd0c99e817e0fb&scene=21#wechat_redirect)  

  * [免杀技术有一套（免杀方法大集结）(Anti-AntiVirus)](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523189&idx=1&sn=44ea2c9a59a07847e1efb1da01583883&chksm=f9e3f42ece947d3890eb74e4d5fc60364710b83bd4669344a74c630ac78f689b1248a2208082&scene=21#wechat_redirect)  

  * [内网渗透之内网信息查看常用命令](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522979&idx=1&sn=894ac98a85ae7e23312b0188b8784278&chksm=f9e3f5f8ce947cee823a62ae4db34270510cc64772ed8314febf177a7660de08c36bedab6267&scene=21#wechat_redirect)  

  * [关于漏洞的基础知识](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247523083&idx=2&sn=0b162aba30063a4073bad24269a8dc0e&chksm=f9e3f450ce947d4699dfebf0a60a2dade481d8baf5f782350c2125ad6a320f91a2854d027e85&scene=21#wechat_redirect)  

  * [任意账号密码重置的6种方法](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522927&idx=1&sn=075ccdb91ae67b7ad2a771aa1d6b43f3&chksm=f9e3f534ce947c220664a938bc42926bee3ca8d07c6e3129795d7c8977948f060b08c0f89739&scene=21#wechat_redirect)  

  * [干货 | 横向移动与域控权限维持方法总汇](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522810&idx=2&sn=ed65a8c60c45f9af598178ed20c89896&chksm=f9e3f6a1ce947fb710ff77d8fbd721220b16673953b30eba6b10ad6e86924f6b4b9b2a983e74&scene=21#wechat_redirect)  

  * [手把手教你Linux提权](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247522500&idx=2&sn=ec74a21ef0a872f7486ccac6772e0b9a&chksm=f9e3f79fce947e89eac9d9077eee8ce74f3ab35a345b1c2194d11b77d5b522be3b269b326ebf&scene=21#wechat_redirect)

  
 ** **欢迎关注LemonSec**** **觉得不错点个 **“赞”** 、“在看“**

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

