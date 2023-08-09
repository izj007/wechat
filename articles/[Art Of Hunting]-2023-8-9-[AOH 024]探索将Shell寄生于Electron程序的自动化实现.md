#  [AOH 024]探索将Shell寄生于Electron程序的自动化实现

原创 Djerryz  [ Art Of Hunting ](javascript:void\(0\);)

**Art Of Hunting** ![]()

微信号 gh_d3ebfd9e0148

功能介绍 未知的漏洞存在于我们的想象中，直到它被安全的艺术所塑造

____

___发表于_

收录于合集

#electron 1 个

#rat 1 个

#aoh 1 个

#漏洞挖掘 11 个

#硬核 15 个

## 前言

越来越多的桌面应用程序选择Electron框架。

Electron提供了一种可以调试的方法，通常是利用Chrome的inspect功能或通过Node.js调用inspect指令等方式进行。

通过阅读inspect的源码，大致理解了其功能实现，并基于此开发了一种自动寄生常见Electron程序的工具。

结合与（C2）服务器建立连接，进而实现简单的远程控制。

由于被寄生(注入)的程序大部分是著名的，拥有庞大的终端用户数量，带有被信任的数字签名，因此它们的行为被用户和其他软件所广泛信任，在这些程序上下文中执行恶意命令提供了极好的隐蔽性和稳定性。

当然，对于这些被注入的程序，有必要仔细考虑此问题带来的潜在法律风险，即用户分析程序行为时，他们可能会惊讶地发现执行恶意行为的父进程，是来自他们所信任的应用程序范围。

  

## Electron_Shell项目

已开源：https://github.com/djerryz/electron_shell

特性：

  * 支持几乎全部的操作系统

    * mac

    * linux

    * windows

  * 支持几乎全部electron开发的桌面程序   (部分厂商认定该问题属于本地命令执行的范围，因此给予了赏金)

    * Microsoft Team

    * Discord

    * GitHubDesktop

    * QQ

    * 淘宝直播

    * vscode

    * 等等

  * 所有恶意操作都由被注入的程序执行,  可用于绕过终端管控的ACL策略

  * 部分免杀

    * 静态免杀  

    * 启发式不免杀-原因是本项目的shell在操作时实际还是调用cmd，太暴力

    * anti-AV: 可以将完整的shell功能用nodejs实现，并修改相应注入代码，原生实现文件、网络等一系列敏感操作，当然这需要基于项目做二次开发

  

## 演示

从部署C2，到生成植入器，到寄生于常见的桌面程序并执行cmd命令:

  

  

  

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

