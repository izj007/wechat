#  大力出奇迹—从目录爆破到getshell

[ 黑战士 ](javascript:void\(0\);)

**黑战士** ![]()

微信号 heizhanshi1

功能介绍 关注网络安全，为网络安全而战

____

___发表于_

收录于合集

某日在做一个渗透测试项目，但是没有任何收获，这怎么能给领导交差呢？于是只能加班加点进行测试，终于在我大力出奇迹的干法下，拿到了一个shell。

# 0x01 获取备份文件

1、对目标站点进行目录扫描没有什么收获，只有一些403，

![]()

但是总感觉这里会有东西，于是我又重新fuzz了一下目录，把目标的公司名缩写加在了目录名中，果然大力出奇迹，获取到了一个备份文件。

![]()

2、在备份文件中获取到了许多敏感信息

![]()

# 0x02 通过钉钉KEY和SECRET获取敏感信息

1、env的文件中有微信小程序、公众号、QQ、钉钉等IM通讯软件的KEY和SECRET

![]()

2、微信的KEY和SECRET都尝试利用了，但都没能获取到token，可能是设置的有IP地址限制，但是钉钉的可以成功利用，利用官方的API
获取accessToken

https://open-
dev.dingtalk.com/apiExplorer#/?devType=org&api=oauth2\\_1.0%23GetAccessToken  
![]()

然后有了token，能够获取的数据就有很多了，这里只演示一下获取部门列表，根据官方API手册，获取部门列表

![]()

成功获取部门列表信息

![]()

# 0x03 微信支付宝支付接口信息泄露

1、在Web.config文件中获取到了微信和支付宝支付的接口信息

![]()

2、支付密钥泄漏，就有可能导致攻击者花1元购买了100元的商品。系统进行验证时，会发现签名正确，商户号正确，订单号支付成功，
**若代码没有验证支付金额与订单是否匹配**
，将完成攻击者的订单。在许多网站或者App中，曾出现过只验证签名和订单id的情况，没有验证实付金额，因此可以通过这种金额篡改进行攻击。

![]()

3、并且文件中还泄漏了证书文件

![]()

有了证书就可以调用微信支付安全级别较高的接口（如：退款、企业红包、企业付款）

4、这里就没有进行利用（害怕ing）

# 0x04 接口文档泄露导致getshell

1、泄露的文件中还有一个接口文档，在其中查到了一个文件上传的接口

![]()

2、测试后发现该接口是未授权访问并且可以上传webshell

![]()

但是返回的链接直接拼接到url上并不是正确的shell路径，于是本着大力出奇迹的原则，开始爆破webshell的路径，可以先选择一些常用的上传文件的接口路径进行爆破

    
    
    file/  
    fileRealm/  
    file\_manager/  
    file\_upload/  
    fileadmin/  
    fileadmin/\_processed\_/  
    fileadmin/\_temp\_/  
    fileadmin/user\_upload/  
    upload/  
    filedump/  
    filemanager/  
    filerun/  
    fileupload/  
    files/  
    files/cache/  
    files/tmp/  
    logfile/  
    paket-files/  
    profile/  
    profiles/  
    

我们发现uploadFile这个路径和其它的不太一样

![]()

成功连接shell

![]()

# 0x05 总结：

1、本次能有这么多收获，都是从那个备份文件中获取到的信息，fuzz目录这个思路是从密码爆破中学来的，虽然好多公司都要求密码设置强密码，但是还是有一定的逻辑的

比如说

腾讯的系统  
tx@123！  
tc@123456！

可以自己收集一些特定密码，进行爆破，简单写了一个python脚本，还不太完善，大家可以加入一些自己的想法。

    
    
    #coding=utf-8  
    import sys  
    key = sys.argv\[1\]  
    f = open("%s.txt"%key,"w")  
    list1 = \[123,321,1234,4321,123456,654321,12345678,123456789,1234567890,888,8888,666,6666,163,521,1314,1,11,111,1111,2,222,3,333,5,555,9,999\]  
    list2 = \['#123','#1234','#123456','@123','@1234','@123456','@qq.com','qq.com','@123.com','123.com','@163.com','163.com','126.com','!@#','!@#$','!@#$%^','098'\]  
    for j1 in list1:  
        pwd1 =  key + str(j1) + '\\n'  
        f.write(pwd1)  
    for j2 in list2:  
        pwd2 =  key+str(j2)+'\\n'  
        f.write(pwd2)  
      
    for i in range(1000,2021):  
        #pwd1 = key + str(i) + '\\n'  
        pwd3 = '{}{}{}'.format(key,i,'\\n')  
        f.write(pwd3)  
      
    f.close()  
    print (key+' password ok')

2、对于密钥的利用，需注意要区分是企业内部应用还是第三方应用，关于微信密钥的利用可以看下这位大佬的文章：https://xz.aliyun.com/t/11092

3、文件上传接口那里，也是花了很长时间，慢慢尝试才成功上传了的。

  

内容来源：https://forum.butian.net/share/1466  

往期推荐

[

关于自学\跳槽\转行做网络安全行业的一些建议

](https://mp.weixin.qq.com/s?__biz=MzUxMzQ2NTM2Nw==&mid=2247489608&idx=1&sn=e0127ab135708b6588c1a51e63d4c6bd&chksm=f9559530ce221c26b6ec8787707b29043bcd6cff0b52f322e2d6e9e4e7fe1ab20965800d8a16&scene=21#wechat_redirect)[

google hacker语句

](https://mp.weixin.qq.com/s?__biz=MzUxMzQ2NTM2Nw==&mid=2247489588&idx=1&sn=d7f359cfcaba347680b78b3db57440da&chksm=f955954cce221c5a8a213f25f27cbe0bc40daea8c129ea784f622e2e877221710e407e69b8a7&scene=21#wechat_redirect)[

为什么说网络安全是风口行业

](https://mp.weixin.qq.com/s?__biz=MzUxMzQ2NTM2Nw==&mid=2247489569&idx=1&sn=ef5cdc4b0bef6f79a185f55ac6bf28ee&chksm=f9559559ce221c4f1affaaa1165fc020749a3013bf48d5338da1ed7ea44fda998fd42c2cb350&scene=21#wechat_redirect)[

杂七杂八的网络安全知识

](https://mp.weixin.qq.com/s?__biz=MzUxMzQ2NTM2Nw==&mid=2247489525&idx=1&sn=209b1d6b10b6c90450eb873738d8516e&chksm=f9559a8dce22139b32ac3d60793159cbedace497d018f4bd690e1c9af0694fe9934bf4f1e46e&scene=21#wechat_redirect)[

如何快乐地检测SQL注入

](https://mp.weixin.qq.com/s?__biz=MzUxMzQ2NTM2Nw==&mid=2247489479&idx=1&sn=38fa68a391f3af1fde242e3891bcc859&chksm=f9559abfce2213a985877ffb3d56e9fc0a7d336c2c2103159439f9a9183f9843d14d50ecc0f2&scene=21#wechat_redirect)[

浅谈微信小程序渗透

](https://mp.weixin.qq.com/s?__biz=MzUxMzQ2NTM2Nw==&mid=2247489456&idx=1&sn=39436aedbb3ab01246206269a8dd990e&chksm=f9559ac8ce2213ded794921de9023499743477cb0e9711f9e34a5caecca3bf779a260500b1f0&scene=21#wechat_redirect)

  

  

  

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

