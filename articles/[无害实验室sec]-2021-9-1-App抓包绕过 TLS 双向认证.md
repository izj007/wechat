#  App抓包绕过 TLS 双向认证

blog.yii2.cc  [ 无害实验室sec ](javascript:void\(0\);)

**无害实验室sec** ![]()

微信号 WUHAISEC

功能介绍 非营利性组织，不接广告

____

__

收录于话题

## 敬告

本文所阐述内容仅供技术探讨及研究，所有环境均为实验环境，不存在任何违反破坏计算机行为。文中所有技术与工具仅供研究学习使用，产生一切后果由使用者自负

 **原创作者：hldh214**

 **原文链接：https://blog.yii2.cc/**

 **  
**

# 开始之前

在解释一种技术之前我们往往要先清楚的了解这种技术有什么用, 解决了什么问题  
追根溯源的讲, 移动互联网的经久不衰, 或多或少是因为移动端相对 web 端更封闭  
封闭则意味着知识产权能得到更好的保护, 同时也让安全防护(这里特指中间人攻击)变的更容易实现  
毫不夸张的说, 在 WebAssembly 还没普及之前, web 项目对于中间人攻击是一点办法都没有的  
而对于移动端来说, app 在一定程度上保证了通信安全, 不被窃听  
谈到窃听, 这里有两种解决方案, 一种是基于用户名/密码的认证

 如下图

![](https://gitee.com/fuli009/images/raw/master/public/20210901083222.png)

  

而另一种就是今天的主角: TLS 双向认证,

如下图  

![](https://gitee.com/fuli009/images/raw/master/public/20210901083224.png)

基于用户名/密码的认证可以说是最易于理解的方案, 在很多 app 里面也得到了应用  
而相对 TLS 双向认证而言, 前者对代码侵入更多, 耦合性更高, 难于维护  
更关键的是, 认证逻辑没有统一的规范, 如果设计不当, 反而会搬石砸脚  
而后者有统一的规范(TLS), 可以直接在负载均衡上面配置(比如 nginx), 不涉及代码层, 易于维护

# 踩点

本来打算用 fiddler, 无论我怎么配置, `ClientCertificate.cer` 这个功能都不起效(也可能是我不会用)  
所以临时弄了个学习版的 charles 凑合, 毕竟是商业软件, 用的比 fiddler 顺手多了 :P  
另外准备了 dex2jar 和 Java Decompiler 用来做逆向, 网上教程一搜一大把  
想都不要想下载最新版 app, 直接解压缩 apk 文件, 发现有好几个 `.dex` 文件(multidex), 如下图

![](https://gitee.com/fuli009/images/raw/master/public/20210901083225.png)

另外在 `assets` 目录下发现了我们需要的客户端证书 `client.p12`, 如下图

![](https://gitee.com/fuli009/images/raw/master/public/20210901083226.png)

我们尝试导入这个证书, 发现是需要密码的(这不废话吗xD), 故准备反编译 dex 了  
祭出神器 dex2jar, 一顿操作猛如虎, 然而…

![](https://gitee.com/fuli009/images/raw/master/public/20210901083227.png)

大胆猜测是加了壳或者用了什么混淆, 上网查阅发现 dex2jar 已经很久没更新了, 陷入僵局  

# 降维打击

直接逆向这条思路是肯定不行了, 得想想别的办法, 脑子里面突然冒出四个字: 历史版本  
于是乎, 下载了一堆历史版本, 逐个尝试后终于发现在某个版本是没有给 dex 加壳的  
立马脱掉裤子开干, 用 jd 打开 jar 文件, 全局搜索一下, 此时屏幕上已经出现了一点精斑

![](https://gitee.com/fuli009/images/raw/master/public/20210901083228.png)

拿到密码后就简单了, 启动 charles, 导入 p12 格式证书(Client Certificates), 设置 SSL 白名单(SSL
Proxying)  
Proxy -> SSL Proxying Settings -> SSL Proxying && Client Certificates  

![](https://gitee.com/fuli009/images/raw/master/public/20210901083230.png)

![]()

在 app 里发一次请求后 charles 提示输入证书的密码, 一顿操作后顺利完成任务, 打完收工

![](https://gitee.com/fuli009/images/raw/master/public/20210901083232.png)

# refs

https://www.secpulse.com/archives/54027.html  
https://stackoverflow.com/questions/48959777/how-to-use-applications-client-
certificate-with-charles

## 最后  

 **由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

  

 **无害实验室sec
拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**

  

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

App抓包绕过 TLS 双向认证

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

