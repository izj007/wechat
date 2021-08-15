# 0x01 基础知识


- HKCU = HKEY_CURRENT_USER
- HKLM = HKEY_LOCAL_MACHINE
- HKCR = HKEY_CLASSES_ROOT

![title](https://leanote.com/api/file/getImage?fileId=5e0f066eab64416fe100282b)

上图是 Windows 10 企业版的 `regedit` 的截图。`HKCU` 和 `HKLM` 这2个注册表驱动器有什么特殊之处呢？

在 Powershell 中，仅定义了此2个注册表驱动器。其他的一些如 HKCR 没有被预定义。当然可以自己手动定义。

> 参考：[What, no HKCR in PowerShell?](https://blogs.msdn.microsoft.com/lior/2009/06/18/what-no-hkcr-in-powershell/)

# 0x02 原理分析

**前提：**一些高权限的程序会调用 `HKCR:\` 下的键值。

**思路：**

1. 通过修改 `HKCU:\` 下面的键值同步修改 `HKCR:\` 下的键值。
2. 把原本的键值改为 `cmd.exe` 等 shell 程序。
3. 如果高权限的程序在运行过程中调用此处被修改过的键值，就会以高权限启动我们设定的程序。
4. 如此便实现了 Bypass UAC。

**问题：**

难点在于如何找这种可利用程序：

1. 是高权限程序
2. 需要调用 `HKCR:\` 下的键值。

# 0x03 工具准备

- sigcheck.exe
- Process Monitor

**sigcheck.exe**

[sigcheck.exe](https://docs.microsoft.com/en-us/sysinternals/downloads/sigcheck) 工具可以查看 exe 的 manifest，在 manifest 中可以看到程序的权限。

下图中我查看了 `certutil.exe` 的权限，是普通权限（`asInvoker` 跟随调用者）。

![title](https://leanote.com/api/file/getImage?fileId=5e0f0e4aab64416fe10029a0)

![title](https://leanote.com/api/file/getImage?fileId=5e0f0f71ab64416fe10029d3)

![title](https://leanote.com/api/file/getImage?fileId=5e0f0fdeab64416fe10029ea)

对于 XML 文件中引用的 UAC 执行权限级别，分别代表下列含义：

![title](https://leanote.com/api/file/getImage?fileId=5e0f1d9bab64416fe1002c5f)

`highestAvailable` 和 `requireAdministrator` 的区别是:前者是以当前用户可以获得的最高权限运行，后者是仅以系统管理员权限运行。

我们找的话要找 `highestAvailable` 的，因为如果我们都能以管理员权限运行了，肯定就不需要 bypass UAC 了。

另外有更加直观的判断方法是：

查看文件图标，如果带有 UAC 标志，那么一定是高权限的程序，如图：

![title](https://leanote.com/api/file/getImage?fileId=5e0f1ef0ab64416fe1002ca1)

但是就我们 Bypass UAC 的出发点，我们还得判断其 UAC 执行级别是 `highestAvailable` 还是 `requireAdministrator`。比如我找到了 `wusa.exe` 这个程序也是 `highestAvailable`。

**Process Monitor**

借助 [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)，可以查看程序运行过程中的*注册表*、文件、网络、进程间的调用关系。

# 0x04 实验

实验环境：Windows 7 SP1 x64

启动 Process Monitor，运行 eventvwr.exe。Process Monitor 选择 Tools-Process Tree，找到 eventvwr.exe。

![title](https://leanote.com/api/file/getImage?fileId=5e12a08bab644139ad0050f0)

右键- Go To Event，如图：

![title](https://leanote.com/api/file/getImage?fileId=5e12a1d9ab644137b200524b)

然后过滤出 `eventvwr.exe` 进程的活动：



![title](https://leanote.com/api/file/getImage?fileId=5e12a7a8ab644137b2005370)

仔细查看进程调用关系，如图：


![title](https://leanote.com/api/file/getImage?fileId=5e12ab1aab644139ad00530f)

![title](https://leanote.com/api/file/getImage?fileId=5e12a9ceab644139ad0052ba)



![title](https://leanote.com/api/file/getImage?fileId=5e12aab2ab644139ad0052fa)

获取了以下一些信息：

- `eventvwr.exe` 的权限为 `high`；
- `eventvwr.exe` 首先查询键值 `HKCU\Software\Classes\mscfile\shell\open\command`，查询结果为 `NAME NOT FOUND`；
- `eventvwr.exe` 接着查询键值 `HKCR\mscfile\shell\open\command`，结果为 `SUCCESS`。

## 修改测试
如果修改键值 `HKCU\Software\Classes\mscfile\shell\open\command`，使其查询结果为 `SUCCESS`，会如何呢？

首先需要修改键值 `HKCU\Software\Classes\mscfile\shell\open\command`，为测试可以把值改为 `calc.exe`。

因为我这里的 `regedit` 中的键值只到 `HKCU\Software\Classes\` 这层目录，所以我新增了后面的表项，并把值设为 `C:\Windows\System32\calc.exe`。

![title](https://leanote.com/api/file/getImage?fileId=5e12af76ab644139ad0053e5)


然后再次运行 `eventvwr.exe`，就发现启动了 `calc.exe`。

使用 Process Monitor 查看进程调用关系，如图：

![title](https://leanote.com/api/file/getImage?fileId=5e12d7b1ab644139ad005bbb)

此时对键值 `HKCU\Software\Classes\mscfile\shell\open\command` 的查询结果为 `SUCCESS`。

![title](https://leanote.com/api/file/getImage?fileId=5e12d7f1ab644137b2005cf0)

计算器权限为 high，成功绕过 UAC。


至此，成功通过修改 `HKCU\Software\Classes\mscfile\shell\open\command`，实现 BypassUAC，获得了高权限。

# 0x05 总结

其实此方法 2016 年就提出来了，据说 Win10 系统已对该处做了修复，但是本新手只是学习一下思路。

在进程 `eventvwr.exe` 启动的时候，首先查找注册表位置 `HKCU\Software\Classes\mscfile\shell\open\command`。如果该处为空，接着查找注册表位置 `HKCR\mscfile\shell\open\command`(此处默认值为 `%SystemRoot%\system32\mmc.exe "%1" %*`)，以高权限启动 `mmc.exe`，最后打开 `eventvwr.msc`。

![title](https://leanote.com/api/file/getImage?fileId=5e12e67aab644139ad005ea1)


修改注册表 `HKCU\Software\Classes\mscfile\shell\open\command` 的键值只需要普通用户权限而已，但是通过在注册表 `HKCU\Software\Classes\mscfile\shell\open\command` 中添加 payload，就可以在启动 `mmc.exe` 之前执行预设的 payload。



修改 `HKCU\Software\Classes\mscfile\shell\open\command`后，会劫持所有 `.msc` 文件的运行，如 `gpedit.msc`：


![title](https://leanote.com/api/file/getImage?fileId=5e12e2d5ab644137b2005f07)

该方法 BypassUAC 的优点为：

- 无文件
- 不需要进程注入
- 不需要复制特权文件


按照此方法，可以继续对 system32 下的 `highestAvailable` 权限的 exe 进行测试，如 `wusa.exe`，寻找可以实现 UACBypass 的键值。这种 hook 的思路，可以继续学习。




# 0x06 参考资料:

[1] https://enigma0x3.net/2016/08/15/fileless-uac-bypass-using-eventvwr-exe-and-registry-hijacking/
[2] https://enigma0x3.net/2016/05/25/userland-persistence-with-scheduled-tasks-and-com-handler-hijacking/
[3] https://blog.gdatasoftware.com/2014/10/23941-com-object-hijacking-the-discreet-way-of-persistence
[4] https://enigma0x3.net/2016/08/15/fileless-uac-bypass-using-eventvwr-exe-and-registry-hijacking/
[5] https://3gstudent.github.io/3gstudent.github.io/Userland-registry-hijacking/
[6] https://www.cnblogs.com/elisha-blogs/p/msc.html
