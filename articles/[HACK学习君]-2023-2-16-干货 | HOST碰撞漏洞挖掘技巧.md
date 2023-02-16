#  干货 | HOST碰撞漏洞挖掘技巧

HACK学习君  [ HACK学习君 ](javascript:void\(0\);)

**HACK学习君** ![]()

微信号 XHacker1961

功能介绍
HACK学习，专注于网络安全攻防与黑客精神，分享技术干货，代码审计，安全工具开发，实战渗透，漏洞挖掘，网络安全资源分享，为广大网络安全爱好者和从业人员提供一个交流学习分享的平台

____

___发表于_

收录于合集

#HOST碰撞 1 个

#漏洞挖掘 58 个

#渗透测试 72 个

**文章来源： **转载来源** 语雀文档，非** **P喵呜** **作者本人投稿**

 **如有侵权，请联系删除**

 **0x01 前言**

 **  
**在实战中,我们总会对一个企业进行资产收集,在这个过程中会收集到许多资产,有域名有 ip  
但有的时候打开的域名指向的是一个内网 ip 非常无奈 :(  
而打开的 ip 状态码更是直接显示 400,403,404 禁止我们访问  
还有一种状态码显示 200 但是输入啥都没啥变化的  
同时对它们进行目录扫描,也是常常没有结果  
那么这种情况下, HOST 碰撞技术就可以尝试使用了

 **0x02 host 碰撞原理**

 **  
**当数据包的 host 头替换为某个域名时在访问该反代服务器的 ip, 如果 nginx/Apache 的反向代理的 host
配置没删除,就会把请求转发到内网对应的 host 业务服务器上, 接着返回该业务的信息, 实现本该隐藏的业务访问

简单点就是: 当数据包的 host 头替换为某个域名时在访问时该反代服务器的 ip, 如果页面发生了变化,返回了对应的资源, 即可判断为存在 host 碰撞

 **0x03 host 碰撞什么时候存在?**

 **  
**1业务通过 DNS 解析到外网，后面删除了 A 记录(但是 nginx/Apache 的反向代理还没删除)  
2测试业务(不对外开放的业务,只流传于开发或是测试使用)

 **0x04 什么样的 ip 能进行 host 碰撞?**

 **  
**这里我看网上很多人写文章都是写 ip 状态码为 40X 的时候,在进行 host 碰撞  
但是我想说,这不一定正确!!!  
实际上,我认为应该改为任何一个 ip 都有 host 碰撞的价值!!!  
这个点的问题在于,现在很多较大的公司比较流行,资产统一把控,也就是自己所有的资产全部收缩进内网  
然后整个 nginx 或是 Apache 服务器,想对外网开放某个资产的时候就通过这个反代服务器新添加个配置映射出去  
这就导致了一个问题, 那就是如果配置不当了, 忘记删除这台 nginx 或是 Apache 服务器的域名指向了  
那么我们通过修改 host 就可以重新访问这些以前在外网后面被收缩进内网的资产了

    
    
    例如说:  
    现在外网有个 ip: 47.10.10.1(虚构的)  
    它的域名为: testmiao.com  
      
    现在它对映射规则配置不当了  
      
    然后打开状态码显示 200,出现的是一个站点,返回的数据为一段 json  
      
    对外映射的: a.testmiao.com  
    对外映射的: b.testmiao.com  
    内部 nginx/Apache 还映射的: oa.testmiao.com  
      
    那么这种情况下如果我们进行爆破式 host 碰撞  
    撞了一个 oa.testmiao.com 进去  
    那么 nginx 或是 Apache 服务器接收到这个 host: oa.testmiao.com   
    直接去请求了这个所谓的被收缩进内网的资源,然后返回  
    

这种情况, 我对大公司进行测试时, 已经发现不下于三次  
因此我认为只要是个 ip 能够访问,那么它就有进行 host 碰撞的价值  
当然这个是我自己个人实战经验,读者们看个乐乎就好了 :)

 **0x05 host 碰撞检测方法-思路**

 **  
**网上大佬是遇到 40X,或是收集到了内网域名在进行 host 碰撞  
这里我的检测方法对比网上那些大佬的比较泛~~

    
    
    我的检测方法是:  
    第一步:   
        收集目标域名  
        PS: 内外网的域名都要  
    第二步:   
        收集目标 ip 段  
    第三步:   
        将外网域名保存为一个 hostList.txt 备用  
    第四步:   
        将外网域名全部 ping 一下获取一下 ip,并将收集到的目标 ip 段加外网域名 ip 段保存为一个 ipList.txt 备用  
        PS: 只要外网可访问的 ip 哦  
    第五步:  
        将收集到的 ipList.txt 与 hostList.txt 进行 host 碰撞检测  
    第六步:  
        将可以互相解析的 ip 提取出来  
        例如:  
            域名: aa.testmiao.com 解析的 ip: 42.169.88.55  
            域名: bb.testmiao.com 解析的 ip: 42.142.165.49  
      
            ip:42.169.88.55 修改 host 为:bb.testmiao.com  
            然后打开: bb.testmiao.com 显示的还是 bb.testmiao.com 的内容  
            这就说明有价值了 :)  
    第七步:  
        重点测试提取的 ip 进行 host 碰撞爆破  
        例如:  
            域名: aa.testmiao.com 解析的 ip: 42.169.88.55  
            自己构造常见的内网重要的域名  
            如:  
                oa.testmiao.com  
                user.testmiao.com  
                mail.testmiao.com  
                sso.testmiao.com  
                portal.testmiao.com  
    

 **0x06 host 碰撞检测方法-实际**

 **  
**测试方法的话,我这里提供两种比较高效的测试方法

 **0x06.1 测试数据**

    
    
    测试数据:  
      
    // 拿来做碰撞的 ip  
    域名: https://sso.testmiao.com 解析的 ip: 42.xxx.xxx.xxx  
      
    // 拿来做碰撞的 host  
    域名: vms.testmiao.com 解析的 ip: 10.xxx.xxx.xxx  
    域名: a.testmiao.com 解析的 ip: 无法解析(猜的内网可能有这个域名)  
    域名: b.testmiao.com 解析的 ip: 无法解析(猜的内网可能有这个域名)  
    域名: c.testmiao.com 解析的 ip: 无法解析(猜的内网可能有这个域名)  
    域名: d.testmiao.com 解析的 ip: 无法解析(猜的内网可能有这个域名)  
    域名: scm.testmiao.com 解析的 ip: 118.xx.xxx.xxx  
    

 **0x06.2 方法一 - 使用工具 HostCollision**

    
    
    下载地址: https://github.com/pmiaowu/HostCollision  
      
    下载完毕以后  
      
    第一步:   
        打开 HostCollision/dataSource 目录  
        将: ipList.txt 与 hostList.txt 分别填写进对应的数据(一行一个)  
    第二步:  
        打开 HostCollision 目录  
        执行命令: java -jar HostCollision.jar  
      
        执行完毕以后会在根目录生成一个 年-月-日_8 位随机数 csv/txt 文件  
        里面会保存碰撞成功的结果  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230216182933.png)

  
![](https://gitee.com/fuli009/images/raw/master/public/20230216182935.png)  
 **0x06.3 方法二 - 使用 burp  
** 但是这个方法,只能一个 ip 一个 ip 的测试,无法批量 host 爆破  
第一步: 找一个你认为有漏洞 ip 我拿的测试数据的 42.xxx.xxx.xxx  
第二步: 将找到的 host 保存成一个 hostList.txt 分别填写进对应的数据(一行一个)  
第三步: 构造数据包如下图  
![](https://gitee.com/fuli009/images/raw/master/public/20230216182937.png)  
有了这个包以后,发送到测试器里面进行 host 爆破

![](https://gitee.com/fuli009/images/raw/master/public/20230216182938.png)

  
![](https://gitee.com/fuli009/images/raw/master/public/20230216182940.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20230216182941.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20230216182943.png)  

![](https://gitee.com/fuli009/images/raw/master/public/20230216182944.png)  

![](https://gitee.com/fuli009/images/raw/master/public/20230216182945.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230216182947.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230216182949.png)

  
 **0x06.4 浏览器访问的方法**

  

 **0x06.4.1 方法一 - 系统 hosts 文件修改  
** 这里就教如何利用 windows 访问对应的站点

    
    
    例如:  
        ip: 42.xxx.xxx.xxx  
        host: vms.testmiao.com  
    

打开文件: C:\Windows\System32\Drivers\etc\hosts  
![](https://gitee.com/fuli009/images/raw/master/public/20230216182950.png)  
![](https://gitee.com/fuli009/images/raw/master/public/20230216182952.png)  
 **0x06.4.1 方法二 - 使用 burp**  

![](https://gitee.com/fuli009/images/raw/master/public/20230216182953.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230216182955.png)

  
如果还看不明白,这里也有中文版的可以看一眼

![](https://gitee.com/fuli009/images/raw/master/public/20230216182957.png)

  
 **0x07 总结**

  

    
    
     1. 收集一些内网常见的重要域名  
    例如:  
        oa.testmiao.com  
        user.testmiao.com  
        mail.testmiao.com  
        sso.testmiao.com  
        portal.testmiao.com  
    2. 尽量多可能的收集目标所有可对外访问的 ip  
    3. 尽量多可能的收集目标的域名  
    4. 下载 HostCollision  
    5. 等待老天爷的宠幸~~~~  
    

要是前面的文章还看不是很懂的话,可以看看这张网站访问流程的图会更加清晰  

![](https://gitee.com/fuli009/images/raw/master/public/20230216183000.png)

多试试总会成功的 :)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230216183001.png)  

  

 **推荐阅读**

  

 **[实战 |
记一次渗透拿下某儿童色情网站的经过](http://mp.weixin.qq.com/s?__biz=MzIzNzMxMDkxNw==&mid=2247488278&idx=1&sn=dc87c72cac0aa7c20b67b509339c909b&chksm=e8cbd5bcdfbc5caa9746a9f8e566cc4ab8e9243808dfed93bd0e9a92d5184f95a15ca16ac16e&scene=21#wechat_redirect)**

  

 **[实战 |
某某街一处XSS的绕过思路](http://mp.weixin.qq.com/s?__biz=MzIzNzMxMDkxNw==&mid=2247490434&idx=1&sn=9a41d7cc99b71b44f8b73ba9c7f7a0eb&chksm=e8cbdd28dfbc543eb5f6869ff95b820ca886e8c5abf70931358e7ffcbaa2163501fa56be597d&scene=21#wechat_redirect)  
**

  

[ **实战 |
记一次企业钓鱼演练**](http://mp.weixin.qq.com/s?__biz=MzIzNzMxMDkxNw==&mid=2247487781&idx=1&sn=2773c2aba8741bbf009ba3c0fde1c15a&chksm=e8cbd78fdfbc5e997cb1db69ea6f105b3f5f6d0f85a2134bc5c1a4d08be91be2511afee7da3d&scene=21#wechat_redirect)

  

[ **干货 |
2022年超全的安全知识库**](http://mp.weixin.qq.com/s?__biz=MzIzNzMxMDkxNw==&mid=2247488454&idx=1&sn=cb01427909964d0bb6bec6e00d7aa4ca&chksm=e8cbd56cdfbc5c7a411fa943ea0a89fe762827432c507a9633605ac049ed6a3cc01fbe987264&scene=21#wechat_redirect)  

  

[ **实战 |
实战一次完整的BC网站渗透测试**](http://mp.weixin.qq.com/s?__biz=MzIzNzMxMDkxNw==&mid=2247487922&idx=1&sn=e7b85887725d5304636bea1e8f869bc6&chksm=e8cbd718dfbc5e0e9395232682ccf79383d2f0f1a2b9da537381ad49dd047456a7344f0da32a&scene=21#wechat_redirect)

  

 **星球部分精华内容推荐**

 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20230216183002.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230216183003.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230216183005.png)

其他更多精彩内容，欢迎加入我们的星球  

![](https://gitee.com/fuli009/images/raw/master/public/20230216183006.png)

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

