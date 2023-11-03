#  实战｜记某次红蓝对抗(内附小工具 NaturalTeeth v1.2)

原创 ddwGeGe  [ 弱口令验证机器人 ](javascript:void\(0\);)

**弱口令验证机器人** ![]()

微信号 gh_4f1df697395d

功能介绍 脚本小子一个，擅长内网弱口令验证，以及文档大师，不定期分享漏洞挖掘，免杀等，感谢各位师傅关注

____

___发表于_

收录于合集

【本文仅为学习和交流，切勿用作非法攻击 ，参与非法攻击与笔者无关 ！！！】

最近接连参与两省某行业hvv，紧接着参与某企业红队测试，累趴![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_35@2x.png)，期间遇到很多有意思的点，故来分享一波，大佬勿喷![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Social.png)
，感谢队友持续输出，太顶了，只能说师傅牛皮![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png)

  * 

    
    
    末尾附 NaturalTeeth v1.2 下载地址，更新了10+ poc

 **打点一**

  * 

    
    
     云主机 -弱口令-文件上传-绕云WAF+getshell-bypass杀软-内网

目标IIS环境+某云WAF，尝试了空格、换行、asmx、asa、cer等等方式，基本都会拦截检测，对于我这种脚本小子来说太折磨了![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Hurt.png)

![]()

![]()

测试了一会儿，发现通过chunked 编码 + 头部添加一些参数+免杀shell
可成功上传，可连接没一会儿shell直接没了![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/2_05.png)，太绝望了，最终利用蚁剑成功getshell，但权限有点低

![]()

![]()

发现存在一堆杀软，想到之前被杀的shell，真滴绝望![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Hurt.png)，开始提权之路，常规的提权都没成功，最后利用PrintNotify
Potato成功提权到system权限![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png)，最后就是内网了。

![]()

![]()

 **打点二**  

  * 

    
    
     信息收集-默认账户密码-审计-绕WAF上传-内网-靶标

感谢 **@V1rtu0l** 师傅的现场审计，只能说牛，膜拜![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Social.png)。通过某搜索引擎获取PDF文件拿到账户及密码规则，登录了系统后台，发现可单点跳转多个系统，但都无法进行getshell，其中某系统队友发现有源码，故快速审计一波，跟着师傅的脚步也简单分析一下![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png)

  * 

    
    
    site:xxx.xx.xx filetype:pdf

![]()

![]()

可惜waf拦截的很死![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/2_05.png)，最终利用免杀shell成功绕过，在内网横向过程中，发现某mssql数据库sa弱口令，直接system权限，发现开放80端口，代理访问之后，发现与外网靶标系统一模一样，真滴是运气![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png)

![]()

![]()

在另一个目标一样的系统存在同样的漏洞，后缀和内容检测太严格，正常上传txt可成功上传，到结束都未突破，简单记录一下![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_45@2x.png)  

 **后缀突破** （个人分析 可能不太对![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Social.png)，waf应该是匹配filename=的全部文件名，若不在黑名单就直接放过，而后端是以.进行分割获取后缀
如上代码分析 直接进行拼接）

  *   *   *   *   *   *   *   *   *   * 

    
    
    "aaa.jsp" 拦截"aaa.jspx" 拦截"aaa.js p" 不拦截 但无法解析"aaa.jsp."  拦截"aaa.jsp空格" 拦截"aaa.jspx" 拦截"aaa.jsp/" 拦截"aaa.jsp 成功bypas 返回jsp文件ab"

![]()

 **内容突破**

这waf拦截的太严格，只要含有jsp相关敏感字符就直接封堵IP，代理池直接被用完了![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Hurt.png)，经过不断的测试，发现脏数据可以绕过。  

![]()

尝试jspx+各类编码 依然不行，最后一丢丢时间用 脏数据+jsp el表达式成功bypass获取了当前的web路径。

  * 

    
    
    ${pageContext.getSession().getServletContext().getClassLoader().getResource("")}

![]()

 **打点三**

  * 

    
    
     OA弱口令-后台1day-bypass杀软-内网

某OA弱口令爆破登录成功，后台1day文件上传成功getshell，system权限

![]()

![]()

开始上cs抓密码进行内网扫描，发现主机安装了某EDR，配合免杀的fscan+手动加白，发现扫描3分钟，该EDR直接进行隔，脑袋嗡嗡的响![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_35@2x.png)，发现没有卸载密码直接进行卸载，然后在开扫。暂时不放图，抱歉![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Social.png)

  

在内网横向过程在拿到某堡垒机权限，最终获取全部核心业务系统及服务器权限，真滴幸运![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_45@2x.png)。

![]()

刚好碰到某管理平台，故简单记录一下，iSecure文件上传获取root权限，通过配置文件获取管理中心的web端口，通过大佬的解密工具获取数据库密码，替换管理员密码进入到后台，最后在进行密码恢复。

![]()

![]()

![]()

![]()

 **打点四**

  * 

    
    
     邮箱弱口令-VPN口令复用-SQL注入-bypass杀软-内网

某企业红队测试，主要分享一下bypass，由于信息敏感，自己搭建环境演示一下![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_45@2x.png)，VPN进入之后发现存在HCM系统，经过探测存在SQL注入漏洞，且可以直接
--os-
shell获取system权限。由于本地靶机环境限制，是administrator权限，见谅![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png)

![]()

下一步想着直接写shell，然后用webshell管理工具进行管理更方便一些，采用echo直接写入默认的哥斯拉shell，发现直接被杀，发现存在某企业版杀软。

![]()

但生成免杀的shell又太长，涉及太多的转义。故思路变成 免杀shell-Base64编码-然后在echo写入，最后在certutil
-decode解码，中间发现Base64编码也很长，也会出现报错，可对Base64编码进行拆分，分多次写入，最终getshell。

  *   *   *   *   * 

    
    
    echo xxxxx >> c:\xxx\ccc\111.txtecho xxxxx >> c:\xxx\ccc\111.txtecho xxxxx >> c:\xxx\ccc\111.txt  
    certutil.exe -decode c:\xxx\111.txt c:\xxx\111.aspx

![]()

![]()

 **NaturalTeeth  v1.2 更新**

 **下载地址  
**

如果有一些帮助，也麻烦师傅点个小星星![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_14@2x.png) 感谢
~~，最近有收到小伙伴在地市hvv中有用部分漏洞成功getshell，能起到一点点帮助就行![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/Expression/Expression_45@2x.png) **  
**

  * 

    
    
    https: //github.com/ddwGeGe/NaturalTeeth/

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    新增漏洞 修复部分bug问题  
    1、H3C ER 敏感信息泄露2、大华智慧园区综合管理平台任意密码读取3、致远OA M1-Server RCE 暂时只支持检测4、JumpServer存在未授权访问漏洞5、大华智慧园区综合管理平台文件上传漏洞(POI) 6、大华智慧园区综合管理平台文件上传漏洞(video) 7、JeecgBootSQL注入8、jeecgboot任意代码执行漏洞9、泛微uploadify文件上传 10、广联达 Linkworks GetIMDictionary SQL注入11、海康威视isecure center 综合安防管理平台文件上传12、宏景HCM SQL注入漏洞13、致远 SeeyouWpsAssistServlet文件上传

 **部分漏洞测试**

 **某微uplo adify文件上传 **

![]()

![]()

 **某SQL注入**  

![]()

 **某文件上传**

![]()

##  **某boot任意代码执行漏洞**

 **  
**

![]()

##  **某平台文件上传**

 **  
**

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

