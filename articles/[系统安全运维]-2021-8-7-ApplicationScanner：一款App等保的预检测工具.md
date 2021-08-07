##  ApplicationScanner：一款App等保的预检测工具

[ 系统安全运维 ](javascript:void\(0\);)

**系统安全运维** ![]()

微信号 Taurus-1314147

功能介绍 每日分享，绝不偷懒🙈🙈

____

__

收录于话题

关于ApplicationScanner

ApplicationScanner是一个快速稳定的App代码扫描工具，该工具基于Python3.7实现其主要功能，apk检测部分需要JDK
11的支持，因此具备较好的跨平台特性，目前支持在Linux和Mac系统上使用，暂不支持Windows。

## 功能介绍

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    _____    /\               / ____|   /  \   _ __  _ __| (___   ___ __ _ _ __  _ __   ___ _ __  / /\ \ | '_ \| '_ \___ \ / __/ _` | '_ \| '_ \ / _ \ '__| / ____ \| |_) | |_) |___) | (_| (_| | | | | | | |  __/ |/_/    \_\ .__/| .__/_____/ \___\__,_|_| |_|_| |_|\___|_|         | |   | |         |_|   |_|  
                                 By ParadiseDuo  [Version: 2.0]  
    Usage:    python3 AppScanner.py -i *.apk/*.ipa  
        -h help    -i <inputPath>    -s save cache (Default clear cache)

使用ApplicationScanner可以对ipa和apk文件进行扫描，快速发现存在风险的代码，检测项目与等保的检测项目进行了对齐，换句话说，如果ApplicationScanner没有扫到的问题，等保扫描时大概率也检测不到。

针对apk文件，会检测以下风险项目

  * 应用数据任意备份风险检测

  * Broadcast Receiver动态注册检测

  * 剪切板敏感信息泄露检测

  * 数据库文件任意读写检测

  * SDCARD加载dex检测

  * AES/DES弱加密风险检测

  * FFMPEG任意文件读取检测

  * Fragment注入攻击检测

  * Intent组件隐式调用风险检测

  * IP泄露检测

  * JS资源文件泄露检测

  * 日志泄漏风险检测

  * PendingIntent错误使用Intent风险检测

  * 网络端口开放威胁检测

  * 全局可读写风险检测

  * Java反射检测

  * 截屏攻击风险检测

  * So文件破解风险检测

  * SDCARD加载so检测

  * SQL注入检测

  * URL泄露检测

  * WEB STORAGE数据泄露检测

  * WebView安全检测

    * WebView绕过证书校验漏洞

    * WebView远程代码执行检测

    * WebView远程调试检测

    * WebView明文存储密码检测

    * WebView未移除有风险的系统隐藏接口漏洞

  * InnerHTML的XSS漏洞检测

  * Zip文件解压目录遍历检测

针对ipa文件，会检测以下风险项目:

  * 不安全的API函数引用风险检测

  * 未使用自动管理内存技术风险检测

  * 地址空间随机化技术检测

  * 编译器堆栈保护技术检测

  * 证书类型检测

  * iBackDoor控制漏洞检测

  * IP泄露检测

  * 内存分配函数不安全风险检测

  * 创建可执行权限内存风险检测

  * 调试日志函数调用风险检测

  * 代码未混淆风险检测

  * 注入攻击风险检测

  * 可执行文件被篡改风险检测

  * SQLite内存破坏漏洞检测

  * HTTP传输数据风险检测

  * AES/DES加密算法不安全使用检测

  * 弱哈希算法检测

  * 随机数不安全使用检测

  * Webview组件跨域访问风险检测

  * XcodeGhost感染检测

  * ZipperDown解压漏洞检测

## 工具安装

  * 

    
    
    > npm -g install js-beautify> git clone https://github.com/paradiseduo/ApplicationScanner.git> cd ApplicationScanner> pip install -r requirements.txt

如果是Mac，需要安装binutils：

  * 

    
    
    > npm -g install js-beautify> git clone https://github.com/paradiseduo/ApplicationScanner.git> cd ApplicationScanner> pip install -r requirements.txt

## 工具使用

  * 

    
    
    > python3 AppScanner.py -i test.apk

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    _____    /\               / ____|   /  \   _ __  _ __| (___   ___ __ _ _ __  _ __   ___ _ __  / /\ \ | '_ \| '_ \___ \ / __/ _` | '_ \| '_ \ / _ \ '__| / ____ \| |_) | |_) |___) | (_| (_| | | | | | | |  __/ |/_/    \_\ .__/| .__/_____/ \___\__,_|_| |_|_| |_|\___|_|         | |   | |         |_|   |_|  
                                 By ParadiseDuo  [Version: 2.0]I: Using Apktool 2.5.0 on test.apkI: Loading resource table...I: Decoding AndroidManifest.xml with resources...I: Loading resource table from file: /Users/xmly/Library/apktool/framework/1.apkI: Regular manifest package...I: Decoding file-resources...I: Decoding values */* XMLs...I: Baksmaling classes.dex...I: Copying assets and libs...I: Copying unknown files...I: Copying original files...  
    检测项目: 签名信息项目描述: 签名验证详细信息危险等级: 信息扫描结果:VerifiesVerified using v1 scheme (JAR signing): trueVerified using v2 scheme (APK Signature Scheme v2): trueVerified using v3 scheme (APK Signature Scheme v3): falseVerified using v4 scheme (APK Signature Scheme v4): falseVerified for SourceStamp: falseNumber of signers: 1Signer #1 certificate DN: C=US, O=Android, CN=Android DebugSigner #1 certificate SHA-256 digest: 11fd518047589c9bfcbbbb45711917d77ee92f214cae3139a746d1049f635190Signer #1 certificate SHA-1 digest: a579de8a6dbd5edb575823c5b86ace003df6dc40Signer #1 certificate MD5 digest: 93a85244b2463b52f682de6972fc331bSigner #1 key algorithm: RSASigner #1 key size (bits): 2048Signer #1 public key SHA-256 digest: 9b1f3a1ac030576fb25b45d3f4a55025044c7de6bd2b1ebaa3ac89968ab06d2dSigner #1 public key SHA-1 digest: aa41abb46b7a14d386a13953c8a587e538f97096Signer #1 public key MD5 digest: b2cdc1649e14715779251f33030887cb  
    检测项目: 证书指纹项目描述: 证书指纹信息危险等级: 信息扫描结果:所有者: C=US, O=Android, CN=Android Debug发布者: C=US, O=Android, CN=Android Debug序列号: 1生效时间: Wed Jul 01 18:00:50 CST 2020, 失效时间: Fri Jun 24 18:00:50 CST 2050证书指纹:     SHA1: A5:79:DE:8A:6D:BD:5E:DB:57:58:23:C5:B8:6A:CE:00:3D:F6:DC:40     SHA256: 11:FD:51:80:47:58:9C:9B:FC:BB:BB:45:71:19:17:D7:7E:E9:2F:21:4C:AE:31:39:A7:46:D1:04:9F:63:51:90签名算法名称: SHA1withRSA主体公共密钥算法: 2048 位 RSA 密钥版本: 1  
    检测项目: 权限信息项目描述: 应用使用权限信息危险等级: 信息扫描结果:  包名: com.hijack.demo.hijack  使用权限列表      android.permission.ACCESS_COARSE_LOCATION      ...      com.heytap.mcs.permission.RECIEVE_MCS_MESSAGE  
    检测项目: Zip文件解压目录遍历检测项目描述: 检测Apk中是否存在Zip文件解压目录遍历漏洞危险等级: 高危扫描结果:com.hijack.demo.hijack.TestZip.smali : 74  
    检测项目: 截屏攻击风险检测项目描述: 检测App是否存在截屏攻击风险检测危险等级: 低危扫描结果:com.hijack.demo.hijack.QQActivity.smali  
    检测项目: 网络端口开放威胁检测项目描述: 检测App中是否存在网络端口开放风险危险等级: 低危扫描结果:com.hijack.demo.hijack.UdpClient$1.smali UDP : 76com.hijack.demo.hijack.SocketServer.smali TCP : 31  
    检测项目: WebView远程代码执行检测项目描述: 检测App应用的Webview组件中是否存在远程代码执行漏洞危险等级: 高危扫描结果:com.hijack.demo.hijack.MyWebView.smali : 129

  

 **欢迎关注 系统安全运维  **

 **觉得不错点个 **“赞”** 、“在看”哦**
**![](https://gitee.com/fuli009/images/raw/master/public/20210807085533.png)**

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

ApplicationScanner：一款App等保的预检测工具

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

