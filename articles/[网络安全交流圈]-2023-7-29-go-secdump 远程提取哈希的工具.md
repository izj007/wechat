#  go-secdump 远程提取哈希的工具

时光与她  [ 网络安全交流圈 ](javascript:void\(0\);)

**网络安全交流圈** ![]()

微信号 gh_6d11e0d3a78e

功能介绍 审计，渗透，二进制，kali，分享圈子。

____

___发表于_

收录于合集

![]()

##  

##  

## 描述：

Package go-secdump 是一种用于从 SAM 远程提取哈希的工具 注册表配置单元以及来自安全配置单元的 LSA 机密和缓存哈希
无需任何远程代理，也无需接触磁盘。

该工具构建在库  https://github.com/jfjallid/go-smb  之上 并使用它来与Windows远程注册表通信以检索注册表
直接从内存中获取密钥。

它是作为一种学习体验和概念证明而构建的，它应该 可以从 SAM 配置单元和 LSA 远程检索 NT 哈希 机密以及域缓存的凭据，而无需先保存
注册表配置单元到磁盘，然后在本地分析它们。

要克服的主要问题是SAM和安全配置单元仅是 可由 NT 权限\系统读取。但是，我注意到本地组 管理员对注册表配置单元具有 WriteDACL 权限，并且可以
因此用于临时授予对自身的读取访问权限以检索 机密，然后还原原始权限。  

## 用法：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    Usage of ./go-secdump:  -d string      domain  -debug      enable debugging  -hash string      hex encoded NT Hash for user  -host string      host  -noenc      disable smb encryption  -pass string      password  -port int      SMB Port (default 445)  -smb2      Force smb 2.1  -user string      username

## 例子：

转储注册表哈希

  * 

    
    
    ./go-secdump --host DESKTOP-AIG0C1D2 -user Administrator -pass adminPass123

下载地址：https://github.com/n00py/go-secdump  

 **承接以下业务：**

![]()  

 **欢迎添加微信业务咨询：**

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

