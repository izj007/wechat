#  发现新恶意团伙"紫狐"！针对finalshell的供应链事件

原创 刨洞安全团队 [ 刨洞安全团队 ](javascript:void\(0\);)

**刨洞安全团队** ![]()

微信号 gh_55f3b3854b4b

功能介绍
刨洞群官方公众号，一群热爱网络安全的人，关注渗透测试、红队攻防、代码审计、内网渗透、WAF绕过、云原生安全等领域技术研究，分享网安学习心得，永远充满激情。

____

___发表于_

收录于合集

> FinalShell - 一个免费且好用的ssh工具

最近刚重装完系统，新系统需要安装一些环境，在网上找 `finalshell` 的时候，发现搜出来两个 `finalshell` 官网：

![]()

第二个官网的域名是：`www.finalshell.org`

这个域名感觉更真实一点，点进去之后长这样：

![]()

下载发现安装包 `Defender` 会报毒

![]()

到这里已经发现事情的严重性了，众所周知，`finalshell` 拥有大量的用户。而此网站在搜索引擎里排名第二，很多像我这样的小白用户还是很容易上当下载的

先把这个域名丢到微步看一下，发现注册时间为 `2023-07-30` 日，是刚刚注册几个月的新域名

![]()

注册信息都有设置隐私保护，看不到  

![]()

数字证书这里我们可以看到也是 `2023-09-27` 刚申请的证书，并且还是 `Let's Encrypt` 的免费证书

![]()

再看正确官网：`hostbuf.com`

是白名单域名，注册时间在 `2016` 年，且 `whois` 等信息都是公开的

![]()

![]()

我们将下载回来的安装包样本上传到微步沙箱看看  

![]()

  

参考云沙箱进行分析

从静态分析模块中可以看出，样本的是 `WinRAR` 打包的

![]()

从行为检测模块中可以看出，样本在运行后会 `%temp%` 目录中释放两个文件，分别是：`install.exe` 和 `install_8.exe`

![]()  

同时还有疑似载荷下载行为

![]()

在执行流程中，可追溯到这个IP：`107.148.48.35`

这个 `IP` 来自于 `install_8.exe` ，此时再结合沙箱释放文本中的检出，大概可以实锤这个 `install_8.exe` 是木马了

![]()

![]()

总结一下上面的信息，样本采用 `"Self-extracting (SFX) archives"` 方式，将 `install.exe` 和
`install_8.exe` 打包，而其中的 `install_8.exe` 是一个马，解压后并会自动运行连接到 `107.148.48.35`
下载其他文件

  

人工分析

首先使用 `7z` 工具验证一下安装包情况，可发现挂马的 `FinalShell` 是将马和正常的 `FinalShell` 打包一起，利用 `SFX`
的特性安装到系统中

![]()

![]()

 **第一阶段分析：**

将 `install_8.exe` 拖到 `ida` 打开，提示以下信息 (再次实锤为木马)

![]()

样本首先会根据 `C://ProgramData//Micros//xiaomum.lnk` 是否存在来判断机器是否被感染过

如果没有过，则在 `http[:]//107.148.48.35:35/3.txt` 下载 `shellcode`，并采用文件映射的方式，放在内存中开始运行

![]()

![]()

 **第二阶段：**

将 `shellcode` 转换为 `exe`，投入到云沙箱中可以看到，该 `exe` 中存在多个 `URL`

![]()

可见有 `6个` 功能组件，就留给其他人分析，大概有释放 `DLL` 的模块、持久化模块还有利用漏洞驱动强制 `KILL` 进程的模块

    
    
    http[:]//107.148.48.35:35/1.txt  
    http[:]//107.148.48.35:35/2.txt  
    http[:]//107.148.48.35:35/3.txt  
    http[:]//107.148.48.35:35/4.txt  
    http[:]//107.148.48.35:35/5.txt  
    http[:]//107.148.48.35:35/6.txt

  

组织关联

目前，微步团队已判定为 `"紫狐"` 组织

![]()

此域名情报已更新为 `"恶意软件"`

![]()

某些 `CSDN` 的博文还会提到去此恶意网址下载 `finalshell`，具有极强的诱导性

![]()

大家下载请认准官方网站为：

https://www.hostbuf.com/

  

  * 

    
    
    感谢：@烟花易冷丶 @隐

  

![]()

关注公众号后台回复 `0001` 领取域渗透思维导图，`0002` 领取VMware 17永久激活码，`0003` 获取SGK地址，`0004`
获取在线ChatGPT地址，`0005` 获取 Windows10渗透集成环境，`0006` 获取 CobaltStrike 4.9.1破解版

  

  

加我微信好友，邀请你进交流群

  

![]()

  

  

  

往期推荐

  
  
[

对某金融App的加解密hook+rpc+绕过SSLPinning抓包

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247484615&idx=1&sn=b0cfad610f3e27476b5ea35862060b61&chksm=c35843e4f42fcaf295a2846511ee06c6e2a89a30e1924763264947906e6959319392399f1f8d&scene=21#wechat_redirect)[

疑似境外黑客组织对CSDN、吾爱破解、bilibili等网站发起DDoS攻击

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247484581&idx=1&sn=ebf96305b45f385ac2d4652b3274ed03&chksm=c3584386f42fca906f91c79479e67f98e0c28a7b65e9cef5338f7b4dd045d9393a52fc01586c&scene=21#wechat_redirect)[

Fofa新产品 - 绕CDN溯源真实IP！

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247484562&idx=1&sn=81069adf9a0658d02187bd32165d5953&chksm=c35843b1f42fcaa7379c3aa56ad08a6f5af97fc597310daa896a104ee6e32fe7354779c604ad&scene=21#wechat_redirect)[

Cobalt Strike 4.8 正式发布上线！

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247484549&idx=1&sn=c740626cd569a96ccc8ca15db33e459f&chksm=c35843a6f42fcab0420c4b0deaa5d99827e18604d8cbafeb1f457a6d984cc64f9185345f0316&scene=21#wechat_redirect)[

团队在线Windows进程识别正式内测

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247484462&idx=1&sn=ab9b33db5d5bf47acd2a8c7b7e6ed443&chksm=c358430df42fca1bd43b2bf25540c96d417ce31dbbaf0d1a99d4333249e8003c29c79c509299&scene=21#wechat_redirect)[

突发！微信疑似存在RCE

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247484314&idx=1&sn=7c171d3b48384d338149cdd3c41f1aa3&chksm=c35844b9f42fcdafcfc02adadaa8409c3627b79fa88760e8af93f0012ff5d5ece24723990ce4&scene=21#wechat_redirect)[

APK逆向分析入门-以某斗地主APP为例

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247493978&idx=1&sn=97fa37f3f363aa102dff6369fbfdf9fd&chksm=c35bae79f42c276f4b8e8ccd708f5295af37fc9cb768bb0731cdf65f0ec118304259aae14e52&scene=21#wechat_redirect)[

APK逆向分析入门-以某车载音乐APP为例

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247493953&idx=1&sn=7035f780f0ab396a5fbcf2f05f5bea96&chksm=c35bae62f42c27749bd7c08a660580fe78060a0e54def6000772f2e315a8d3dbd8e2ad030525&scene=21#wechat_redirect)[

记一次渗透中因为JS拿下整个云

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247493923&idx=1&sn=52b9b730cab4bedf8fa2bd5894dead54&chksm=c35bae00f42c2716345644ec9f8c3c385638966e77285d2deabd71c0f79e03635f30ba8affc9&scene=21#wechat_redirect)[

Juniper 新洞 CVE-2023-36845 浅析

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247493901&idx=1&sn=47d75f457035c00c586285c00e0bc9b5&chksm=c35bae2ef42c2738fa82e10796491e9f1165f4c3ce49a35597b8f421ebab94ca3c8f3530950d&scene=21#wechat_redirect)[

记一次红队经历

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247493839&idx=1&sn=94a2f6aa4d493366835d4b2b69897c96&chksm=c35bafecf42c26fa7c1ab2c8afc06d8da8c60f629bdfe2b32c682165b9c63d48354f3c23d159&scene=21#wechat_redirect)[

Apache ActiveMQ RCE 分析

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247493806&idx=1&sn=8ec529e85faddcb65f5b5daf065f733d&chksm=c35baf8df42c269b8f569e3e235c572b8e9c9a954a31fd8abbb5b8827753851a5573f7c345db&scene=21#wechat_redirect)[

WordPress Core RCE Gadget 分析

](https://mp.weixin.qq.com/s?__biz=Mzk0OTM5MTk0OA==&mid=2247493821&idx=1&sn=bf2748ca29c1244376cb7caf7828a8b1&chksm=c35baf9ef42c2688ece46bced335616250306a9a9d1678bd43ae96ce1aa57be08b9d2ed9d641&scene=21#wechat_redirect)

备用号，欢迎关注

  

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

