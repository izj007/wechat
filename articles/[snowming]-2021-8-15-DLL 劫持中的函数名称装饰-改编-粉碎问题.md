>因为蚂蚁笔记官方图片服务器极其不稳定，如果本文出现图片加载不出的问题，请转到此链接查看：https://note.roger101.com/blog/post/snowming/08900703978d


起因是朋友给了我一个白 EXE，让我做个黑 DLL，交流交流手法。

经我测试，我要劫持的是 Common.dll 这个导入表里面的 DLL（隐式加载，位于 .rdata 节区）：

![title](https://leanote.com/api/file/getImage?fileId=5f588129ab64414ac20007ea)

此 DLL 有两个导出函数，因为我没有对应的 Common.dll，所以当我写黑 DLL 的时候，不能简单地做函数转发。虽然我已经找到原 Common.dll 了，但是感觉这样就太作弊了，失去了交流的意义，于是我要自己实现 Common.dll这个黑 DLL。

这个 DLL 特殊的地方主要在于两个导出函数都是被名称改编了的。这样的话如果我用 dllexport 方法导出，里面的 `@`、`?` 等特殊符号无法通过函数名的语法规则。

# 0x01 第一次尝试：.def 文件

```
LIBRARY
EXPORT
?InitBugReport@XXBugReport@@YAXPB_W000GGKHHKKP6GHPAUtagBugReportInfo@1@PBD200PAPAXPAKPAX@Z@Z=Func1
?ValidateBugReport@XXBugReport@@YAXXZ=Func2
```

然后在原 DLL 工程里面实现了 Func1 和 Func2。也就是我为源码中的 Func1 和 Func2 指定了导出的名字，分别为那一大串。

编译通过了，看起来没问题。结果我发现，生成的 DLL 的导出函数名，会被 @ 符号截断，那么实际生成的两个导出函数就是：

```
?InitBugReport
?ValidateBugReport
```

如何避免这个`@`截断呢？我在网上搜了半天，发现一个老外提过和我一样的问题：https://stackoverflow.com/questions/7413242/visual-c-exporting-decorated-function-name-in-a-def-file

回答就一个，推荐他用 __declspec(dllexport)。可是在我的场景下根本用不了！为什么？

因为如果用 dllexport 去导出函数的话，`?`、`@`等特殊符号根本无法通过函数名的语法！！！这也是我最开始使用 `.def` 文件的原因。

多种尝试无法解决 `@` 符号在 .def 生成的导出函数中被截断的问题。 

# 0x02 第二次尝试：回归名称改编


想通过 `.def` 文件走捷径失败之后，思考了很久，我决定回归名称改编问题本身。

也就是我直接写 C++，在里面实现函数名，不指定 extern c，让编译器和 linker 来帮我改编名称！

https://docs.microsoft.com/en-us/cpp/build/reference/decorated-names?view=vs-2019


查阅 MSDN 的文档，找到了 `undname` 这个工具，这个工具用于从名称改编中还原出函数签名：




![title](https://leanote.com/api/file/getImage?fileId=5f58741aab64414ac2000733)



然后开始写 DLL：


```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"
#define EXTERNC extern "C"
#define NAKED __declspec(naked)
#define ALCDECL EXTERNC NAKED void __cdecl
#define EXP __declspec(dllexport)


class XXBugReport
{
public:
    struct tagBugReportInfo
    {
        int i;
    };
    struct tagBugReportInfo* tagBugReportInfo;



    EXP void __cdecl InitBugReport(wchar_t const*, wchar_t const*, wchar_t const*, wchar_t const*, unsigned short, unsigned short, unsigned long, int, int, unsigned long, unsigned long, int(__stdcall*)(struct XXBugReport::tagBugReportInfo*, char const*, char const*, wchar_t const*, wchar_t const*, void**, unsigned long*, void*))
    {
        MessageBoxW(NULL, L"HELLO", TEXT(__FUNCTION__), MB_OK);
    }


    EXP void __cdecl ValidateBugReport(void)
    {
        MessageBoxW(NULL, L"HELLO", TEXT(__FUNCTION__), MB_OK);
    }


};



BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

生成的 DLL 却存在一个问题：


![title](https://leanote.com/api/file/getImage?fileId=5f587489ab644148c6000701)

本来应该是 Y 的地方都是 QA。我以为是调用约定的问题，尝试把`__cdecl`替换为 `__stdcall`、`__fastcall`、`__vectorcall`都不行。



# 0x03 解决问题


查阅了一个外国教授的文档，他对 MSVC 的 C++ 名称改编有所研究。文档中提到：

![title](https://leanote.com/api/file/getImage?fileId=5f58809fab64414ac20007e0)
http://www.kegel.com/mangle.html


在我写的 DLL 中，的确是用类来限定函数名的，为了满足 `::` 的限定符。那么只要不用类，就会生成 `Y` 类型，于是我换成 namespace 来限定函数和 struct：


```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"
#define EXTERNC extern "C"
#define NAKED __declspec(naked)
#define ALCDECL EXTERNC NAKED void __cdecl
#define EXP __declspec(dllexport)


namespace XXBugReport
{

    struct tagBugReportInfo
    {
        int i;
    };
    struct tagBugReportInfo* tagBugReportInfo;



    EXP void __cdecl InitBugReport(wchar_t const*, wchar_t const*, wchar_t const*, wchar_t const*, unsigned short, unsigned short, unsigned long, int, int, unsigned long, unsigned long, int(__stdcall*)(struct XXBugReport::tagBugReportInfo*, char const*, char const*, wchar_t const*, wchar_t const*, void**, unsigned long*, void*))
    {
        MessageBoxW(NULL, L"HELLO", TEXT(__FUNCTION__), MB_OK);
    }


    EXP void __cdecl ValidateBugReport(void)
    {
        MessageBoxW(NULL, L"HELLO", TEXT(__FUNCTION__), MB_OK);
    }
};



BOOL APIENTRY DllMain(HMODULE hModule,
    DWORD  ul_reason_for_call,
    LPVOID lpReserved
)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```


查看生成 DLL 的导出函数：


![title](https://leanote.com/api/file/getImage?fileId=5f5874bdab644148c6000705)


完全一致。

尝试用白 EXE 调用此 DLL：

![title](https://leanote.com/api/file/getImage?fileId=5f5874d6ab644148c6000708)

毫无问题，至此劫持成功！



----------


参考文档：


- https://www.agner.org/optimize/calling_conventions.pdf
- http://www.kegel.com/mangle.html
- https://docs.microsoft.com/en-us/cpp/build/reference/decorated-names?view=vs-2019
- http://aturing.umcs.maine.edu/~meadow/courses/cos335/Asm06-Linkage.pdf
- http://cad_contest.ee.ncu.edu.tw/CAD-contest-at-ICCAD2014/problem_b/contest_file_formats.pdf
- https://blog.csdn.net/Liuchuang_MFC/article/details/49560793
- https://docs.microsoft.com/en-us/cpp/build/exporting-from-a-dll-using-def-files?view=vs-2019


