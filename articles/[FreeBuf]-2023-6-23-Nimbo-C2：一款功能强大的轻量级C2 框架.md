#  Nimbo-C2：一款功能强大的轻量级C2 框架

原创 Alpha_h4ck  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 中国网络安全行业门户

____

___发表于_

收录于合集 #工具 76个

![](https://gitee.com/fuli009/images/raw/master/public/20230623164745.png)![](https://gitee.com/fuli009/images/raw/master/public/20230623164746.png)

##  

## **  关于Nimbo-C2 **

  
Nimbo-C2是一款功能强大的轻量级C2 框架，Nimbo-C2代理支持x64
Windows&Linux操作系统。该工具基于Nim和Python开发，其WIndows端使用了.NET组件。Nim的功能非常强大，但在跟Windows系统交互时使用PowerShell可能会更加简单，因此该工具的部分功能是基于PowerShell实现的。Nimbo-C2的Linux代理更加的精简，只能执行基本命令，其中包括ELF加载（通过memfd技术实现）等。  
工具所有的服务器端组件都基于Python开发，并具备下列功能：  

> 1、HTTP监听器负责管理代理；2、构建器负责生成代理Payload；3、Nimbo-C2是一个交互式C2组件，负责管理所有组件；

##  

##  **  功能介绍 **

  

> 1、构建EXE、DLL、ELF
> Payload；2、使用NimProtect加密植入物配置和字符串；3、使用UPX封装Payload，并对PE代码进行混淆处理以增加检测和解包的难度；4、HTTP通信加密；5、C2命令行终端支持命令自动补全；6、在内存中执行PowerShell命令；7、提供文件上传和下载命令；8、内置扫描发现命令；9、支持屏幕截图、剪贴板数据窃取和音频记录；10、LSASS和SAM
> Hive转储；11、Shellcode注入；12、内联.NET程序集执行；13、具备持久化感染能力；14、支持UAC绕过；15、其他...

##  

##  **  工具安装 **

  

首先，我们需要使用下列命令将该项目源码克隆至本地，并切换至项目目录中：  

    
          *   *   * 
    
    
    
    git clone https://github.com/itaymigdal/Nimbo-C2  
    cd Nimbo-C2

  
接下来，构建Docker镜像：  

    
          * 
    
    
    
    docker build -t nimbo-dependencies .

  
切换到源文件目录中，并运行Docker镜像，暴露的端口为80端口，并会将Nimbo-C2目录加载进容器中（如果是Linux，则需要将下列命令中的${pwd}替换为$(pwd)）：  

    
    
      
    

  *   *   * 

    
    
    cd Nimbo-C2  
    docker run -it --rm -p 80:80 -v ${pwd}:/Nimbo-C2 -w /Nimbo-C2 nimbo-dependencies
    
    
    （向右滑动，查看更多）

##  

##  **  工具使用 **

  
首先，我们需要根据自己的需求修改config.jsonc文件。  
然后运行下列命令启动Nimbo-C2：  

    
          * 
    
    
    
    python3 Nimbo-C2.py

  
使用help命令可以查看该工具的帮助信息。

##  

##  **  工具主窗口 **

    
    
      
      
    

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    Nimbo-C2 > help  
      
      
        --== Agent ==--  
        agent list                    ->  查看活动代理  
        agent interact <agent-id>     ->   与代理交互  
        agent remove <agent-id>       ->  移除代理数据  
      
      
        --== Builder ==--  
        build exe                     ->  构建exe代理(-h查看帮助信息)  
        build dll                     ->  构建dll代理 (-h查看帮助信息)  
        build elf                     ->  构建elf代理 (-h查看帮助信息)  
      
      
        --== Listener ==--  
        listener start                ->   开启监听器  
        listener stop                 ->  终止监听器  
        listener status               ->   打印监听器状态  
      
      
        --== General ==--  
        cls                           ->  清屏  
        help                          ->  打印工具帮助信息  
        exit                          ->  退出

##

    
    
    （向右滑动，查看更多）

##  

##  **  代理窗口 **

###  

###  **Windows代理**

    
    
      
      
    

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     Nimbo-2 [d337c406] > help  
      
      
        --== Send Commands ==--  
        cmd <shell-command>                    ->  执行一个Shell命令  
        iex <powershell-scriptblock>           ->  在内存中执行PowerShell命令  
      
      
        --== File Stuff ==--  
        download <remote-file>                 ->  从代理下载一个文件  
        upload <loal-file> <remote-path>       ->  向代理上传一个文件  
      
      
        --== Discovery Stuff ==--  
        pstree                                 ->  显示进程树  
        checksec                               ->  检查安全产品  
        software                               ->  检查已安装的软件  
      
      
        --== Collection Stuff ==--  
        clipboard                              ->  检索剪贴板数据  
        screenshot                             ->  检索屏幕截图  
        audio <record-time>                    ->  记录音频  
      
      
        --== Post Exploitation Stuff ==--  
        lsass <method>                         ->  转储lsass.exe [方法:  direct,comsvcs] (需要提权)  
    sam                                    ->  使用reg.exe转储sam,、security  
    system hive (需要提权)  
        shellc <raw-shellcode-file> <pid>      ->  向远程进程注入Shellcode  
        assembly <local-assembly> <args>       ->  执行.net程序集、  
      
      
        --== Persistence Stuff ==--  
        persist run <command> <key-name>       ->  设置运行密钥  
        persist spe <command> <process-name>   ->  使用静默进程退出技术实现持久化(需要提权)  
      
      
        --== Privesc Stuff ==--  
        uac fodhelper <command> <keep/die>     ->  使用fodhelper UAC绕过技术实现会话提权  
        uac sdclt <command> <keep/die>         ->  使用sdclt UAC绕过技术实现会话提权  
      
      
        --== Interaction stuff ==--  
        msgbox <title> <text>                  ->  弹窗消息  
        speak <text>                           ->  基于sapi.spvoice实现语音交互  
      
      
        --== Communication Stuff ==--  
        sleep <sleep-time> <jitter-%>          ->  修改休眠时间间隔  
        clear                                  ->  清理挂起命令  
        collect                                ->  重新收集代理数据  
        kill                                   ->  终止代理运行  
      
      
        --== General ==--  
        show                                   ->  显示代理详情  
        back                                   ->  返回主窗口  
        cls                                    ->  清屏  
        help                                   ->  打印工具帮助信息  
        exit                                   ->  退出
    
    
    （向右滑动，查看更多）

###  

###  **Linux代理**

    
    
      
      
    

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     Nimbo-2 [51a33cb9] > help  
      
      
        --== Send Commands ==--  
        cmd <shell-command>                    ->  执行一个终端命令  
      
      
        --== File Stuff ==--  
        download <remote-file>                 ->  从代理下载一个文件  
        upload <local-file> <remote-path>      ->  向代理上传一个文件  
      
      
        --== Post Exploitation Stuff ==--  
        memfd <mode> <elf-file> <commandline>  ->  使用memfd_create系统调用在内存中加载ELF  
                                                   implant模式: 以子进程形式加载ELF并返回  
                                                   task模式: 以子进程形式加载ELF，并等待执行完成后的输出结果  
      
      
      
      
        --== Communication Stuff ==--  
        sleep <sleep-time> <jitter-%>          ->  修改休眠时间间隔  
        clear                                  ->  清理挂起命令  
        collect                                ->  重新收集代理数据  
        kill                                   ->  终止代理运行  
      
      
        --== General ==--  
        show                                   ->  显示代理详情  
        back                                   ->  回到主窗口  
        cls                                    ->  清屏  
        help                                   ->  打印工具帮助信息  
        exit                                   ->  退出
    
    
    （向右滑动，查看更多）

##  

##  **  工具运行截图 **

  
工具主界面：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623164747.png)  
代理下载文件：  
![]()  
注入Shellcode：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623164748.png)  
记录麦克风数据（录音）：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623164749.png)  
发送命令：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623164750.png)  
获取剪切板数据和屏幕截图：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623164751.png)  
UAC绕过：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623164752.png)  
服务器端主界面：  
![](https://gitee.com/fuli009/images/raw/master/public/20230623164753.png)

##

##  

##  **  许可证协议 **

  
本项目的开发与发布遵循MIT开源许可证协议。

##  

##  **  项目地址 **

  
 **Nimbo-C2** ：https://github.com/itaymigdal/Nimbo-C2

##  

![]()

  

![](https://gitee.com/fuli009/images/raw/master/public/20230623164754.png)https://github.com/itaymigdal/NimProtecthttps://github.com/upx/upx  
  

![](https://gitee.com/fuli009/images/raw/master/public/20230623164755.png)  

[![]()](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247490948&idx=1&sn=7562d1d417c7a70b2b74086a1fc9e949&chksm=ce1ce71bf96b6e0d764474a5d1d79f764feaceb056e0cf9969fa49a5495383bc2f3e404daef4&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20230623164756.png)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247490922&idx=1&sn=bb73f64157cb710773a0523f7abd7f17&chksm=ce1ce7f5f96b6ee30127150dde8cd90350e497b365e1c591e8cd91056dbf99245ac0018333ef&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20230623164757.png)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247490903&idx=1&sn=861d15fcbe0026fb18aedb41d7f08e9b&chksm=ce1ce7c8f96b6edecdd5e5f16bf4a4fd5a00fb705f70d2f5a485ae14d40b420b08785a28d82a&scene=21#wechat_redirect)

![](https://gitee.com/fuli009/images/raw/master/public/20230623164758.png)

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

