#  注入任意代码到运行中的 Electron 应用

原创 0xcc [ 非尝咸鱼贩 ](javascript:void\(0\);)

**非尝咸鱼贩** ![]()

微信号 awkwardfish1

功能介绍 临渊羡鱼，不如在家咸鱼

____

___发表于_

收录于合集

_第一次发布手滑，发现马赛克没打干净，只能重发了_

Electron 这玩意儿是开发起来爽，用起来想吐。不知不觉电脑上装了一个又一个臃肿不堪的 Chromium
副本，不知给全球变暖贡献了多少碳排放量。怪不得这几年天气越来越变态。  

新的 “NT 架构” QQ 也加入了 Electron 大军。

记得几年前还能见到 QQProtect 的驱动。最近主力机不用 Windows，不清楚了。换 Electron
确实可以一套代码跨平台，但这样一来别说进程保护，这破框架还自带了运行时代码注入的接口。

大概是 PC 端逐渐式微，现在没什么人做什么尾巴、盗号之类的事情，所以没必要再折腾保护了？

本来想在 Windows 下面演示。我的 Windows 开了 OneDrive 同步，直接触发了一个 bug。

![]()

还是继续 Mac。

Electron 即使是打包到生产环境，仍然带了调试器后门。

带上参数 \--remote-debugging-port，然后用 Chrome 的 chrome://inspect 页面，或者直接用 websocket
和调试协议通信，就能在 electron 上下文执行任意 js。以前还出过利用 DNS rebinding 实现完全远程代码执行的例子。

开发者可以在业务初始化的代码里直接检测这个 flag，拒绝执行。同时也不能处理 app 进程已经在运行的情况。  

不过 Node.js 很贴心地给了另一个动态启用调试的方式，命令行发送 SIGUSR1 信号：

  * 

    
    
    kill -SIGUSR1 [pid]

  

Windows 没有对应的机制，需要在 nodejs 里用

  * 

    
    
    process._debugProcess(pid)

  

源码在这里，可以看到 Windows 下还是用的 CreateRemoteThread。

https://github.com/nodejs/node/blob/9dd574c9e232/src/node_process_methods.cc#L348

应该不会被终端安全软件放过吧……  

开启了调试之后就可以在 Chrome 的里看到远程调试目标了：

![]()

![]()

然后就可以整活了  

  *   * 

    
    
    require('electron').webContents.getAllWebContents()  .forEach(c => c.loadURL('javascript:alert(location)'))

![]()

想注入二进制模块？写一个 dll，`process.dlopen` 一下。

 **这是框架的特性，不是安全边界。** 毕竟都能运行任意本地代码了，能干的事情太多了。但如果你很介意进程被人乱插代码，可能在用 Electron
之前要好好考虑一番。

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

