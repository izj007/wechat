## DLL 劫持

DLL 劫持原理就不再老生常谈，参考最权威的微软官方文档：

[Dynamic-Link Library Search Order](https://docs.microsoft.com/zh-cn/windows/win32/dlls/dynamic-link-library-search-order?redirectedfrom=MSDN)

或者此篇：

[浅谈DLL劫持](https://www.cnblogs.com/bmjoker/p/11031238.html)


## 适用范围

本文中所介绍的劫持安全狗更新服务来维持权限的方法，网站安全狗、服务器安全狗通杀，因为不管是网站安全狗还是服务器安全狗都会有下图所示的更新服务：

![title](https://leanote.com/api/file/getImage?fileId=5e8ebf5aab64412fae001fa4)


## 利用思路

安全狗有个 System 权限启的自启动的更新服务：

![title](https://leanote.com/api/file/getImage?fileId=5e8f49efab64412fae003a4a)

监控其 `Update.exe` 调用失败的记录，选择 `VERISON.dll` 作为调用顺序劫持的目标：

![title](https://leanote.com/api/file/getImage?fileId=5e8f48c7ab64412fae003a19)

通过工具查看【使用到的】导入 DLL 的导出函数：

![title](https://leanote.com/api/file/getImage?fileId=5e8f48d9ab644131a3003aaf)

决定自己编译一个 Version.dll 实现此几个函数，进行调用顺序劫持。

## 操作方法

`Visual Studio` → `新建项目`→`新建动态链接库DLL(C++)`：

>注：测试过通过 external "C" 从 DLL 中导出函数会报错（因为重载了 Windows API）。
![title](https://leanote.com/api/file/getImage?fileId=5e8f44e6ab644131a30039fd)
![title](https://leanote.com/api/file/getImage?fileId=5e8f44ffab64412fae00396d)
![title](https://leanote.com/api/file/getImage?fileId=5e8f4529ab64412fae003977)
然后也尝试了先把三个函数名先改为长度一致的其他名字，然后在 010editor 中修改 DLL 的函数名，但是镜像损坏，也无法被调用。
经过各种测试，通过 `.def` 文件导出函数的方法成功了。


![title](https://leanote.com/api/file/getImage?fileId=5e8f4602ab644131a3003a36)



通过添加现有项的方式导入 test.def：

![title](https://leanote.com/api/file/getImage?fileId=5e8f464aab64412fae0039a6)


![title](https://leanote.com/api/file/getImage?fileId=5e8f4681ab64412fae0039ac)



![title](https://leanote.com/api/file/getImage?fileId=5e8f47cbab644131a3003a7f)

把编译生成的 DLL 文件改名为 `VERSION.dll`，丢到 SafeDogUpdateCenter 文件夹下：


![title](https://leanote.com/api/file/getImage?fileId=5e8f46d4ab644131a3003a5a)


测试：点击运行 `Update.exe` 即可弹出计算器：


![title](https://leanote.com/api/file/getImage?fileId=5e8f4741ab64412fae0039d1)


环境问题遇到了一个报错，但是不影响 calc.exe 的执行。再者，在这里因为我只是为了方便调试，所以双击用户权限运行的。劫持服务继承 SYSTEM 权限，和用户不在一个桌面，所以用户也看不到什么报错弹窗。


那么，要进行权限维持，就把 calc.exe 改为马就行了（这个思路主要是想结合 Cobalt Strike 的 artifact）。因为更新服务本身是个自启动服务，所以我们也不用再加启动项。更新服务每次开机启动，然后调用 Update.exe，exe 调用劫持 dll，马就被执行上线了，隐蔽性较高。

>或者把马写进 DLL 中，因为 DLL 调用 beacon.exe 隐蔽性不够强，得劫持 DLL 还得上传 beacon。所以还需要对 beacon.exe 做额外的免杀处理，因为 beacon.exe 需要落地目标主机上，还需要一直驻留。



## 效果分析


![title](https://leanote.com/api/file/getImage?fileId=5e8f4bccab644131a3003b3a)

被劫持的 `Update.exe` 是有证书的，白 exe + 黑 dll 标准白加黑启动项，而且此更新服务还是白服务启动，还带驱动保护。所以做权限维持相对比较稳。



-------------------


参考文档：

1. [深入分析 DLL 调用过程实现“自适应” DLL 劫持](https://www.4hou.com/posts/wRPR)，嘶吼，丝绸之路，2020/04/05