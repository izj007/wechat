#  Dismember-内存扫描

AYAT0m  [ 合一安全 ](javascript:void\(0\);)

**合一安全** ![]()

微信号 AYAT0m

功能介绍 合一安全专注渗透测试，渗透工具、网络安全技术等资源分享。

____

___发表于_

收录于合集

#利器 60 个

#网络安全 47 个

工具介绍

  
![](https://gitee.com/fuli009/images/raw/master/public/20230105215552.png)

  

Dismember是一款针对Linux内存安全的测试与扫描工具，该工具本质上是一个基于命令行的工具，专为Linux操作系统而设计，可以帮助广大研究人员扫描Linux系统上的所有进程，并尝试从中搜索常见的敏感信息或自定义的正则表达式匹配项。

该工具基于Go语言开发，目前仍在积极开发阶段，之后可能会升级为一个完整的渗透测试工具。

工具命令

  
![](https://gitee.com/fuli009/images/raw/master/public/20230105215552.png)

  

命令| 描述  
---|---  
grep| 根据给定字符串或正则表达式搜索进程内存数据  
scan| 根据预定义的敏感数据模式搜索进程内存数据  
files| 显示进程可访问的文件列表  
find| 搜索一个给定进程名称的PID，如果匹配到了多个进程，则返回第一个匹配项  
info| 显示目标进程的相关信息  
kernel| 显示关于内核的信息  
kill  
| 使用SIGKILL终止一个或多个进程的执行  
list  
| 枚举目标系统中所有的进程  
resume| 使用SIGCONT恢复挂起的进程  
suspend| 使用SIGSTOP挂起一个进程  
tree| 显示一个进程和所有子进程的进程树  
  
  

工具使用

  
![](https://gitee.com/fuli009/images/raw/master/public/20230105215552.png)

  

通过PID搜索进程中的某个模式匹配

    
    
    #搜索进程8012（PID）中的内存信息：  
      
    dismember grep -p 8012 'passwd.*'

通过进程名称搜索进程中的某个模式匹配

    
    
    #搜索进程“redis”的内存相关信息：  
      
    dismember grep -n redis 'key.*'

搜索所有进程中的某个模式匹配  

    
    
    #搜索所有进程中的GitHub API令牌：  
      
    dismember grep 'gh[pousr]_[0-9a-zA-Z]{36}'

搜索所有进程中的内存敏感信息  

    
    
    #搜索所有可访问进程内存中的常见敏感信息：  
      
    dismember scan

工具演示

  
![](https://gitee.com/fuli009/images/raw/master/public/20230105215552.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230105215611.png)

 _ **项目地址:**_  

    
    
    https://github.com/liamg/dismember

或微信公众号回复" ** _DIS1128_** "下载

* * *

  
  
![](https://gitee.com/fuli009/images/raw/master/public/20230105215612.png)精彩推荐  
  
  
[Arsenal快速部署Bug
Bounty](http://mp.weixin.qq.com/s?__biz=Mzg2Mzc0ODA0NQ==&mid=2247487091&idx=1&sn=da3dc1bda277449431aff0c79220c464&chksm=ce72a4d1f9052dc7bce6eb8301f443437ce174aa0cd041e01c070b90170485cb64ac2a895f64&scene=21#wechat_redirect)  
[windows日志分析](http://mp.weixin.qq.com/s?__biz=Mzg2Mzc0ODA0NQ==&mid=2247487010&idx=1&sn=a9f14f5c1e6833d2b7906151a2184a40&chksm=ce72a480f9052d9606e66ab2ef78f726740840c52f1a1602e643398b1a22bca24b960ad21883&scene=21#wechat_redirect)  
[微信小程序抓包测试](http://mp.weixin.qq.com/s?__biz=Mzg2Mzc0ODA0NQ==&mid=2247486818&idx=1&sn=84c28d06e566dce8fe9f53ba4d8ba172&chksm=ce72a7c0f9052ed6c866e877b30ff2b9fff2edebb2693e8f105404a01b145b6216897ae4c9a7&scene=21#wechat_redirect)  

  

 **免责声明  **  

合一安全提供的资源仅供学习，利⽤本公众号合一安全所提供的信息⽽造成的任何直接或者间接的后果及损失，均由使⽤者本⼈负责，公众号合一安全及作者不为此承担任何责任，一旦造成后果请⾃⾏承担责任！合一安全部分内容及图片源自网络转载，版权归作者及授权人所有，若您发现有侵害您的权利，请联系我们进行删除处理。谢谢
!

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

