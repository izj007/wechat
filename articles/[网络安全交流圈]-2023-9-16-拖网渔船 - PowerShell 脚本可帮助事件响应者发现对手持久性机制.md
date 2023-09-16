#  拖网渔船 - PowerShell 脚本可帮助事件响应者发现对手持久性机制

网络安全交流圈  [ 网络安全交流圈 ](javascript:void\(0\);)

**网络安全交流圈** ![]()

微信号 gh_6d11e0d3a78e

功能介绍 审计，渗透，二进制，kali，分享圈子。

____

___发表于_

收录于合集

![]()  

这是什么？拖网渔网是一个 PowerShell 脚本，旨在帮助事件响应者发现  Windows 主机上的潜在入侵指标
，主要关注持久性机制，包括计划任务、服务、注册表修改、启动项、二进制修改等。目前，拖网渔船可以检测MITRE和原子红队特别指出的大多数持久性技术，并定期添加更多检测。

## 主要特点

  * 扫描 Windows 操作系统以查找各种持久性技术（如下所示）

  * 使用 MITRE 技术和调查快速启动元数据的 CSV 输出

  * 分析和修复指导文档 （ https://github.com/joeavanzato/Trawler/wiki/Analysis-and-Remediation-Guidance ）

  * 每个检测的动态风险分配

  * 跨 Windows 10/Server 2012|2016|2019|2022 的常见 Windows 配置的内置允许列表

  * 从“黄金”企业映像中捕获持久性元数据，以便在运行时用作动态允许列表

  * 通过驱动器重新定位分析装载的磁盘映像

  

检查项：

  * 计划任务

  * 使用者

  * 服务项目

  * 运行进程

  * 网络连接

  * WMI事件使用者（命令行/脚本）

  * 启动项目发现

  * BITS作业发现

  * Windows辅助功能修改

  * PowerShell配置文件存在

  * 来自受信任位置的Office加载项

  * SilentProcessExit监控

  * Winlogon Helper DLL劫持

  * 映像文件执行选项劫持

  * RDP跟踪

  * 远程会话的UAC设置

  * 打印监视器DLL

  * LSA安全和身份验证包劫持

  * 时间提供程序DLL

  * 打印处理器DLL

  * 靴子/登录活动设置

  * 用户初始化登录脚本劫持

  * ScreenSaver可执行文件劫持

  * Netsh DLL

  * AppCert DLL

  * AppInit DLL

  * 应用匀场

  * COM对象劫持

  * LSA通知劫持

  * 'Office test'用法

  * Office GlobalDotName用法

  * 终端服务DLL劫持

  * 自动拨号DLL劫持

  * 命令AutoRun处理器滥用

  * Outlook OTM劫持

  * 信任提供程序劫持

  * LNK目标扫描（可疑术语、多个扩展名、多个EXE）

  * 'Phantom' Windows DLL名称加载到正在运行的进程（例如未签名的WptsExtensions.dll）

  * 扫描关键操作系统目录中的未签名EXE/DLL

  * 无报价服务路径劫持

  * PATH二进制劫持

  * 常见文件关联劫持和可疑关键字

  * 可疑证件追捕

  * GPO脚本发现/扫描

  * NLP开发平台DLL覆盖

  * AeDebug/.NET/Script/Process/WER调试替换

  * 资源管理器“加载”

  * Windows终端启动OnUserLogin劫持

  * 应用程序路径不匹配

  * 服务DLL/ImagePath不匹配

  * GPO扩展DLL

  * 潜在的COM劫持

  * 非标准LSA扩展

  * DNSServerLevelPluginDll存在

  * Explorer\MyComputer实用程序劫持

  * 终端服务初始程序检查

  * RDP启动程序

  * Microsoft遥测命令

  * 非标准AMSI提供商

  * Internet设置LUI错误DLL

  * PeerDist\扩展DLL

  * ErrorHandler.CMD检查

  * 内置诊断DLL

  * MiniDump辅助DLL

  * KnownManagedDebugger DLL

  * WOW64兼容层DLL

  * EventViewer MSC劫持

  * 卸载字符串扫描

  * 策略管理器DLL

  * SEMgr钱包DLL

  * WER运行时异常处理程序

  * HTML帮助（.CHM）

  * 远程访问工具对象（文件、目录、注册表项）

  * ContextMenuHandler DLL检查

  * Office AI.exe Presence

  * Notepad++插件

  * MSDTC注册表劫持

  * 讲述人DLL劫持（MSTTSLocEnUS.DLL）

  * 可疑文件位置检查

  

![]()  

![]()

![]()

 **下载链接：https://github.com/joeavanzato/Trawler  
**

 **欢迎添加微信进行业务咨询：  
**

 **承接以下业务：  
**

 **![]()**

![]()

  

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

