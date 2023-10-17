#  reconFTW！一款自动化侦察工具

six2dez  [ Hack分享吧 ](javascript:void\(0\);)

**Hack分享吧** ![]()

微信号 HackShareB

功能介绍 专注分享国内外优秀安全工具和开源项目

____

___发表于_

收录于合集

#工具 298 个

#信息搜集 67 个

**声明：** 该公众号分享的安全工具和项目均来源于网络，仅供安全研究与学习之用，如用于其他用途，由使用者承担全部法律及连带责任，与工具作者和本公众号无关。  
  
---  
  
  

现在只对常读和星标的公众号才展示大图推送，建议大家把Hack分享吧“设为星标”，否则可能看不到了！  

  

 **工具介绍**

reconFTW为您自动执行整个侦察过程，它优于子域枚举以及各种漏洞检查和获取有关目标的最大信息的工作。  
reconFTW 使用大量技术（被动、暴力、排列、证书透明度、源代码抓取、分析、DNS
记录...）进行子域名枚举，帮助您获得最多和最有趣的子域名，以便您领先于其他子域名。竞赛。  
它还执行各种漏洞检查，如 XSS、开放重定向、SSRF、CRLF、LFI、SQLi、SSL 测试、SSTI、DNS 区域传输等等。除此之外，它还对目标执行
OSINT 技术、目录模糊测试、dorking、端口扫描、屏幕截图、nuclei扫描。  
![]()  

 **工作流程**

![]()

  

 **工具使用**

 **注意：** 当您在主机（例如 VM/VPS/云）上而不是在 Docker 容器中安装 reconFTW 时，这适用。  

  

### 显示帮助部分

  * 

    
    
    ./reconftw.sh -h

对单个目标执行全面侦察  

  * 

    
    
    ./reconftw.sh -d target.com -r

###

### 对目标列表执行全面侦察

  * 

    
    
    ./reconftw.sh -l sites.txt -r -o /output/directory/

###

### 对时间更密集的任务执行全面侦察（仅适用于 VPS）

  * 

    
    
    ./reconftw.sh -d target.com -r --deep -o /output/directory/

###

### 在多域目标中执行侦察

  * 

    
    
    ./reconftw.sh -m company -l domains_list.txt -r

###

### 通过 axiom 集成执行侦察

  * 

    
    
    ./reconftw.sh -d target.com -r -v

###

### 执行所有步骤（整个侦察+所有攻击），又名 YOLO 模式

  * 

    
    
    ./reconftw.sh -d target.com -a

  

 **下载地址**

 **点击下方名片进入公众号**

 **回复关键字【** **231017** **】获取** ** **下载链接****

  

* * *

 **信  安 考 证**

  
  
需要考以下各类安全证书的可以联系我，价格优惠、组团更便宜，还送【潇湘信安】知识星球 **1** 年！

CISP、PTE、PTS、DSG、IRE、IRS、NISP、PMP、CCSK、CISSP、ISO27001...  
  
---  
  
![]()

往期推荐工具

[渗透测试常用的数据库综合利用工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247484579&idx=1&sn=2f2054508220223100066edf3064b37b&chksm=9036e21ba7416b0d7afecc97d3338d954023a51a5622939782273f264207363e4ea8ba9d18de&scene=21#wechat_redirect)

[RequestTemplate红队内网渗透工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247484512&idx=1&sn=90d24f2727ba99c870d7112c406bf208&chksm=9036e2d8a7416bce7e190896de3c7cbd428d385d5563717bce46b0c7f51f33ea18d3e0f1af1e&scene=21#wechat_redirect)

[Railgun -
一款GUI界面的渗透工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247484499&idx=1&sn=edeae57e8306db1bfdcb72e0b0b655d2&chksm=9036e2eba7416bfdbe6a495b60a89d1a677fc745aa512036072224fc1537d8931276e55f7c90&scene=21#wechat_redirect)

[GoBypass -
Golang免杀生成工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247484019&idx=1&sn=05133dd5cb31474078eb4ab019f164f3&chksm=9036e4cba7416ddd92c8f9147bb2d283ddedcdfb4d46ed5987d192863008be9bc655025e0697&scene=21#wechat_redirect)

[ByPassBehinder冰蝎WebShell免杀工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247484096&idx=1&sn=487a1f9daf0090071913aeceeb266d53&chksm=9036e478a7416d6ebced132fe4a03bb1d2584d7fb3eb433612151648743322ff900ae849d572&scene=21#wechat_redirect)

[SweetBabyScan内网资产探测漏扫工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247484070&idx=1&sn=eb4cd4f6aac6e0a0ef26a75e4bc2bc7a&chksm=9036e41ea7416d088f70e8593099f750c81b079de1b4b14219276b352a2e83a3eb1d5829c94b&scene=21#wechat_redirect)

[掩日 -
适用于红队的综合免杀工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247483997&idx=1&sn=1a87fc6bfec2a48dd3dfa5bc2852e8e8&chksm=9036e4e5a7416df3ff7f5b4cf55b4c8c57cde271128de0b9fcdf381a19afb207223deb88fb61&scene=21#wechat_redirect)

[AuxTools浮鱼渗透辅助工具箱V1](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247483907&idx=1&sn=4e7cf5de0472f6685edb64a598e6aff8&chksm=9036e4bba7416dad5aa253462a84d0c47fdb4e89914c70bd5a3fbbd8b4ef05572a4e7e702faa&scene=21#wechat_redirect).0

  

[POCbomber红队快速打点漏洞检测工具](http://mp.weixin.qq.com/s?__biz=MzA4NzU1Mjk4Mw==&mid=2247483790&idx=1&sn=35e3a0fa64cff8d72313fa41ddcbfc3c&chksm=9036e736a7416e20dcd23cfede5c887bfecf5e489234eeb67040bcbbc6326cb42373d226a186&scene=21#wechat_redirect)

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

