#  挖洞随记-自动化域名收集获得JS中钉钉token利用

原创 Soufaker  [ 夜安团队SEC ](javascript:void\(0\);)

**夜安团队SEC** ![]()

微信号 Night-Sec

功能介绍 主要发布渗透测试,代码审计,红队攻防,漏洞挖掘等实战文章,欢迎各位大佬萌新关注,互相学习。

____

___发表于_

收录于合集

#挖洞随记 11 个

#自动化 3 个

* * *

声明：该公众号大部分文章来自作者日常学习笔记，也有部分文章是经过作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

* * *

# 0X01.前言

    本次测试目标为国内某公益项目的一个专属,漏洞已经修复,运气不算好,获得的token权限不够高,要不肯定是高危跑不脱,这次是通过一个自动化监测收集的域名获取的域名(ps名字越长越怪的域名多看两眼)中找的JS文件看到的泄露企业微信token,然后利用企业微信的开发者接口文档利用获取敏感数据。

# 0X02.挖掘利用过程

1.  在收集到一堆新增加的域名里,我依次点开了一些,发现一个域名虽然打开是空白的,但会加载一些JS文件,这让我来了兴趣,直接ctrl+shift+p全局搜索敏感字段,这不,就来了个获取企业微信的接口https://xxx-xxxx-inc.com/cgi-bin/gettoken?corpid=wxb3xxxxxxxxx&corpsecret=xxxxxxxxxxxxxxx8xfOJxxx

![](https://gitee.com/fuli009/images/raw/master/public/20230321231608.png)

  

2\.  访问URL：https://xxx-xxxx-inc.com/cgi-
bin/gettoken?corpid=wxb3xxxxxxxxx&corpsecret=xxxxxxxxxxxxxxx8xfOJxxx

![](https://gitee.com/fuli009/images/raw/master/public/20230321231636.png)

  

  

3.  这里调用企业微信查询为有效token具有哪些权限,https://open.work.weixin.qq.com/devtool/query?e=48002

![](https://gitee.com/fuli009/images/raw/master/public/20230321231637.png)

  

  

4. 可以通过企业微信开发者工具接口文档https://developer.work.weixin.qq.com/document/path/91022进行操作一些敏感操作。

![](https://gitee.com/fuli009/images/raw/master/public/20230321231639.png)

5\. 如获取部门成员信息

https://xxxx-inc.com/cgi-
bin/user/list?access_token=xxxxxxxxxxxxxxxx&department_id=上面查询获取到的ID

![](https://gitee.com/fuli009/images/raw/master/public/20230321231640.png)

  

6.其他操作我尝试了,如增加成员或者删除成员或者创建一个企业微信的邀请链接,但是权限不够,可惜了,要是够就是高位了,最后浅浅给个中。

![](https://gitee.com/fuli009/images/raw/master/public/20230321231642.png)

# 0X03.总结

因为之前没写过这种漏洞,虽然比较常见,但是我最想分享的还是我们一些奇怪的域名,不要看他打开是空白的还是没啥数据就不管了,多看看JS里的东西,有意外惊喜。

  

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

