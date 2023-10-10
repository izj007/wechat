#  【严重漏洞】JWT token可允许恶意操作，包括远程代码执行（RCE）

moon  [ 皓月当空w ](javascript:void\(0\);)

**皓月当空w** ![]()

微信号 hanaffectionl

功能介绍 最快的威胁情报，最全的漏洞评估，总有你想知道的

____

___发表于_

收录于合集 #漏洞早知道 92个

  

![]()

皓月当空，明镜高悬

![]()  

  

  

文末有图片更好保存~

  

![]()漏洞早知道  

 **漏洞名称** ：JWT token可允许恶意操作，包括远程代码执行（RCE）

 **漏洞出现时间** ：2023年10月6日

 **影响等级** ：严重

 **影响版本** ：

\- < 5.2.2  

**漏洞说明：**  

    影响用户可以对Manager和API访问的身份验证中使用的JWT令牌（JSON Web令牌）进行逆向工程，伪造有效的NeuVector令牌以在NeuVector中执行恶意活动。这可能导致RCE。补丁升级至NeuVector[5.2.2版](https://open-docs.neuvector.com/releasenotes/5x)或更高版本和最新Helm图表（2.6.3+）。+在5.2.2中，JWT签名证书由控制器自动创建，有效期为90天，并自动轮换。+使用Helm-based部署/升级到5.2.2为Manager、REST API、ahd注册表适配器生成唯一证书。

 **修复方式：**

  * 升级置最新版本

  * https://github.com/neuvector/neuvector/security/advisories/GHSA-622h-h2p8-743x  

 **相关链接：  
**

  * https://github.com/neuvector/neuvector/security/advisories/GHSA-622h-h2p8-743x

  * https://open-docs.neuvector.com/releasenotes/5x  

 ** **最快的威胁情报，最全的漏洞评估****  

 **![]()**

  

  

Tips  

![]()  
  
  
  
  
  
  
  
  
  
  
  

JSON Web Token (JWT) 是一个开放标准 (RFC 7519)，它定义了一种紧凑（compact）且自包含（self-
contained）的方式，用于在各方之间以JSON 对象方式安全地传输信息。  

  

图片版本更好保存哦~

![]()

  

  

目前正在筹备漏洞情报中心，有什么建议和想法可以公众号留言，建议一旦采取，将获得内测名额

目前就这个鬼样子，ui我实在是无能为力，有热心小伙愿意设计ui也可以找我  

![]()

![]()

![]()

  

  

近期文章：

[【1day细节】某OA pweb 接口sql注入  
](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484881&idx=1&sn=4e657c8fcc089c836184cf58c51d2b72&chksm=cf6f7b4df818f25be7848c5b789fb6d17032d4ab6d68b415d5395e2aeb1d394c3d6f9fa03be1&scene=21#wechat_redirect)

[【1day细节】某OA detail
sql注入](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484909&idx=1&sn=131ff4ccaed9a2ebe142801dc8321c17&chksm=cf6f7b71f818f267945cd2c0ba418a7985ea344c359e06c8595843a0d173726ad9b0cbab239f&scene=21#wechat_redirect)  
[](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484909&idx=1&sn=131ff4ccaed9a2ebe142801dc8321c17&chksm=cf6f7b71f818f267945cd2c0ba418a7985ea344c359e06c8595843a0d173726ad9b0cbab239f&scene=21#wechat_redirect)

[【8-29 网安面试题】今天的题目很简单，我猜你可能不会  
](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484843&idx=1&sn=772b303b7bfebde91f5178944bbcd375&chksm=cf6f7b37f818f22196b97caa7967c4f5d3f097608df9207c8860b0fc300e22378d7a211403d8&scene=21#wechat_redirect)

[【8-27 网安面试题】这些基础题你还记得吗  
](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484823&idx=1&sn=33b81fbc4fb970c623e77d7a5ef46525&chksm=cf6f7b0bf818f21d3450c49112319228ee7fb2557f4914af6808ec6fc7fa3f9c5bd38780228c&scene=21#wechat_redirect)

[](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484823&idx=1&sn=33b81fbc4fb970c623e77d7a5ef46525&chksm=cf6f7b0bf818f21d3450c49112319228ee7fb2557f4914af6808ec6fc7fa3f9c5bd38780228c&scene=21#wechat_redirect)[【严重漏洞】
Apache FreeRDP
出现多个cve漏洞](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484859&idx=1&sn=eb3e9f6d87304e78741397da0c29936b&chksm=cf6f7b27f818f2313bb5832d63d54d85dc146bf3ebcc06278a2c8dd59605e7b1d442b0a3fa25&scene=21#wechat_redirect)  

[【高危漏洞】【未修复】EduSoho企培开源版存在未授权访问漏洞  
](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484863&idx=1&sn=fd7d16aef7c10a2699fb491b0ea02bbe&chksm=cf6f7b23f818f235e455e36c1ff8cf56d2dd964ea7f37a11ce67e7c63d2cb705f212ad02278e&scene=21#wechat_redirect)

[【高危漏洞】达梦企业管理器（DEM）存在未授权访问漏洞](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484888&idx=1&sn=9e4d6603a8c3dc5d63532fa5571f1ccf&chksm=cf6f7b44f818f25269d6a7ebfa54a19b4f84d83e1c19fd110710a353d53bc0a67065685847fa&scene=21#wechat_redirect)  

[【高危漏洞】达梦大数据分析平台存在未授权访问漏洞  
](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484863&idx=2&sn=f6032b4c7a109a838b9392ef2950abce&chksm=cf6f7b23f818f23587ab5d2095882f8ff6a1bb0e205a67c49016dfe860ed4abe67b47e5467ac&scene=21#wechat_redirect)

[【高危漏洞】北京启明星辰信息安全技术有限公司天镜Web应用检测系统存在弱口令漏洞](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484888&idx=2&sn=7fa6fe96332176f5c1feb21c9064a9e5&chksm=cf6f7b44f818f2523bf768e1a85a1edbbc6e5db8747a7c9982bf89e62e9915309b6e92fe3834&scene=21#wechat_redirect)  

[【严重漏洞】CVE-2023-34968
Samba信息泄露漏洞](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484853&idx=1&sn=e1e42f0123d773143f6eb8b56669ec70&chksm=cf6f7b29f818f23fe32ad614b7c99821d28a1f20e3b8ca2fc4543b754e6c1de14e878862b4ff&scene=21#wechat_redirect)

[【高危漏洞】启明星辰信息安全技术有限公司4A统一安全管控平台存在命令执行漏洞](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484849&idx=1&sn=30d011f2463c4912c42491289a590062&chksm=cf6f7b2df818f23b8376cbb2f6e2da22b223ea184c98106ddbbae747b44a0d2299a3c3202d27&scene=21#wechat_redirect)

[【高危漏洞】CVE-2023-40195-Apache Airflow
存在反序列化漏洞](http://mp.weixin.qq.com/s?__biz=Mzg4MDg5NzAxMQ==&mid=2247484843&idx=2&sn=b6bcc3857a123bc3a0f79238617ada50&chksm=cf6f7b37f818f2212881d7cbb330d01874949be7a51e6433fbc79450d7ec968bc3eaaca37b51&scene=21#wechat_redirect)

  

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

