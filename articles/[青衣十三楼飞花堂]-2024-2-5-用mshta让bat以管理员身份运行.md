#  用mshta让bat以管理员身份运行

原创 土鸡炖蘑菇  [ 青衣十三楼飞花堂 ](javascript:void\(0\);)

**青衣十三楼飞花堂** ![]()

微信号 scz------

功能介绍 C/ASM程序员之闲言碎语

____

___发表于_

    
    
    作者: 土鸡炖蘑菇@52pojie  
    创建: 2024-01-30 17:14  
    更新: 2024-02-05 11:39  
    https://scz.617.cn/windows/202401301714.txt  
    

有些bat可能要求以管理员身份运行，一般都是资源管理器中右键选「以管理员身份运行」，弹出UAC框，继续；或在管理员级cmd、explorer中启动bat。为在Win10中获取管理员级explorer，不大容易，参看

《Win10开管理员级资源管理器》

    
    
    https://scz.617.cn/windows/202401111737.txt  
    

有些bat分发给小白，可能都说不清楚啥叫「以管理员身份运行」，但又要求如此，此时若bat自动弹UAC框申请提权，是个小白友好的主意。52pojie网友「土鸡炖蘑菇」分享了一个奇技淫巧，我第一次学到。

在some.bat最前面加两行:

    
    
    %1 mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c %~s0 ::","","runas",1)(window.close)&&exit /b 0  
    cd /d "%~dp0"  
    

双击some.bat后，自动弹UAC框，无需右键选择"以管理员身份运行"。勿删::，否则some.bat进入无限循环，只能长按Ctrl-C退出。

顺便说一下「如何在cmd中判断当前cmd是否提权过」，不是在Process
Explorer、任务管理器或其他什么工具中判断某个cmd是否提权过，而是自己判断自己是否提权过。

    
    
    chcp 437  
    whoami /groups | findstr /c:"S-1-5-32-544" | findstr /c:"Enabled group"  
    whoami /priv | findstr SeDebugPrivilege  
    

无输出，表示普通cmd；有输出，表示cmd已提权。对比下列两组输出:

    
    
    $ whoami /groups | findstr /c:"S-1-5-32-544"  
    BUILTIN\Administrators  Alias   S-1-5-32-544    Group used for deny only  
      
    $ whoami /groups | findstr /c:"S-1-5-32-544"  
    BUILTIN\Administrators  Alias   S-1-5-32-544    Mandatory group, Enabled by default, Enabled group, Group owner  
    

推荐"whoami /priv"，不受cmd当前代码页影响。除了whoami，可能有人用过at、net，我不推荐。比如"net
session"要求Server服务启动中，我的Server服务常年禁用状态。

由是，写一个测试用的some.bat

    
    
    @echo off  
      
    %1 mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c %~s0 ::","","runas",1)(window.close)&&exit /b 0  
    cd /d "%~dp0"  
      
    whoami /priv | findstr SeDebugPrivilege > nul  
    if %errorlevel% equ 0 (  
        echo You are Administrator  
    ) else (  
        echo You are not Administrator, exiting...  
        ping -n 4 127.0.0.1 > nul 2>&1  
        exit /b 1  
    )  
      
    pause  
    

可在explorer中双击，也可在cmd中执行。可用rem注释掉mshta那行，形成对比。这不构成安全漏洞，UAC框还是会弹的，只是说some.bat会自己申请提权。

这种技巧我这辈子都用不上，是不是在一些不太合法的需求中用得着啊，总觉得正经人用不上。再就是runas，因为没这需求，未测试过，不知满足原始需求否，有试成功的给个细节反馈呗，要求Win10/Win11可用。

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 用mshta让bat以管理员身份运行

原创 土鸡炖蘑菇  [ 青衣十三楼飞花堂 ](javascript:void\(0\);)

轻触阅读原文

![]()

青衣十三楼飞花堂

赞 分享 在看 写留言

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

