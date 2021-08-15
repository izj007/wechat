# 0x01 「Cobalt Strike 中的桌面控制功能」概述

在 Cobalt Strike 中，在获取目标机器的 Beacon shell 的前提下，要与目标主机上的桌面交互，通过 `[beacon]` → `Explore` → `Desktop(VNC)`。这会将一个 VNC 服务器转入当前进程的内存中并通过 Beacon 对连接建立隧道。

当 VNC 服务器准备就绪时，Cobalt Strike 会打开一个标签为 `Desktop HOST@PID` 的标签页。

也可以使用 Beacon 的 `desktop` 命令来将一个 VNC 服务器注入一个特定的进程。使用 `desktop pid 架构 low|high` 命令。最后一个参数用于指定 VNC 会话的画质。

# 0x02 问题描述

环境：

- 目标系统为 Windows 10，上面有 360(`ZhuDongFangYu.exe`) 杀软。
- Cobalt Strike 3.14 非试用版

通过选项卡 `Desktop (VNC)` 选项无法打开 Desktop 标签页：

![title](https://leanote.com/api/file/getImage?fileId=5e21890dab64415c0100299d)

![title](https://leanote.com/api/file/getImage?fileId=5e218848ab64415e0e002893)

![title](https://leanote.com/api/file/getImage?fileId=5e21881fab64415c01002971)

![title](https://leanote.com/api/file/getImage?fileId=5e2188b8ab64415c0100298c)

可以看到在团队服务器的 7609 端口派生了一个 VNC 服务器，但是与 VNC 服务器的连接没有响应。

尝试的一些思路是：

1、查看 VNC 的 DLL 是不是存在：

在 Cobalt Strike 团队服务器上确认存在：

![title](https://leanote.com/api/file/getImage?fileId=5e218a76ab64415e0e0028f0)

2、查看团队服务器的 7609 端口是否开放：

![title](https://leanote.com/api/file/getImage?fileId=5e218aeaab64415c010029f9)

看上去就是此服务，那也不是端口的问题。

# 0x03 问题解决

解决方案就是：

一种思路是：把 Desktop(VNC) 工具注入到 `explorer.exe` 进程中，这样回来一个会话，即用此会话去开 VNC Desktop。

但是用 explorer.exe 的话，可能对方会明显感觉卡顿。最好是切到一个在线的用户权限上，像截屏、键盘记录等这些后渗透功能，一般都要到对应的用户空间下操作。

**具体命令：**

在 Beacon 控制台中，
```
desktop [explorer pid] x86|x64 low|high
```
注：

- `low|high` 控制截屏画质。


**操作实例：**

![title](https://leanote.com/api/file/getImage?fileId=5e216d67ab64415e0e0023cb)

![title](https://leanote.com/api/file/getImage?fileId=5e216d2eab64415c0100245c)

![title](https://leanote.com/api/file/getImage?fileId=5e216d90ab64415c01002471)

**注意：**


有时候也会有这种情况，就是用户存在，没登录进去，一个标志就是没有 `winlogon.exe`：

![title](https://leanote.com/api/file/getImage?fileId=5e218cf3ab64415c01002a55)

这种情况下，键盘记录、截屏、VNC 桌面等这些后渗透工具都用不了。

不过我这里肯定不是这种情况，因为我这里 `explorer.exe`、`winlogon.exe` 这两个进程都存在。

只有用户登陆了，才会有 `explorer.exe`、`winlogon.exe` 这两个进程。

一个题外话：

如果进入桌面标签页无法键入，检查桌面底部按钮的状态，也要确保 `View only` 没有被按下。默认情况下，为了阻止操作者意外的移动鼠标，`View only` 被默认按下了。

另外，实际对此功能的测试中，延迟卡顿较为严重，据说 Cobalt Strike 的此 Desktop VNC 功能不是太好用，特别是在目标系统为 Windows 10 时。那么这种情况下一般就是转发3389，直接 mstsc。





----------------

## 参考文档：

1. [Cobalt Strike mannual 4.0](https://www.cobaltstrike.com/downloads/csmanual40.pdf)，Cobalt Strike 官网
2. 感谢我的同事 `@L.N.` 的指点，感谢 `@Beli1v1` 参与讨论

