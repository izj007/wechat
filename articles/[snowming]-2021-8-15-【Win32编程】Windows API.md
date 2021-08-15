## 什么是 API？


所谓`API`就是`Application Programming Interface`的缩写，其实就是操作系统留给应用程序的一个调用接口，应用程序通过调用操作系统的 API 而使操作系统去执行应用程序的命令。

在 Windows 中，系统 API 是以函数调用的方式提供的，API 函数都是包含在系统 DLL 中的导出函数，DLL 文件不能直接执行，它们通常由 exe 在执行时装入，其内含有一些资源以及可执行代码等，系统 DLL 中就含有了 API 函数的执行代码。

直接调用 API 编程又称之为 SDK 编程。SDK 即 software develop kit（软件开发工具包）的缩写，它包含了进行 Windows 软件开发的文档和 API 函数的输入库、头文件。早期 SDK 是一个单独发放的包，现在一些开发环境已经包含了它。


API 和 SDK 是开发 Windows 应用程序所必需的东西，所以其他编程框架和类库都是建立在它们之上的。像 MFC 类库就是对 API 的封装，所以 SDK 编程相对比较灵活。SDK 还有一个优点就是其编写的程序体积会比用类库的小得多。

##调用 DLL 中的 API 函数

那具体怎么调用 DLL 中的 API 函数呢？

在SDK中API函数的申明是包含在头文件和`导入库`中的，所以在程序中我们必须要有API函数的头文件（`.h`）和其导入库(`.LIB`)，可以分别用`#include`和`#pragma comment`语句加载到程序中。

具体步骤如下:

 1. 包含要调用函数的头文件，比如在文件开头使用`#include<Windows.h>`，这里的`Windows.h`就是我们要包含的头文件。
 2. 连接到指定的导入库文件(`.LIB`)。一般情况下 IDE 默认已经连接了常用的导入库文件，所以这一步不用我们完成。在没有默认连接的情况下，我们可以在文件开头使用`#pragma comment(lib, "ws2_32")`，其中`ws2_32`就是导入库的名称。
 3. 调用要使用的 API 函数。


##一个弹出对话框的小程序

```
#include <iostream>
#include <Windows.h>
#include<tchar.h>


int main()
{
    //调用 MessageBox 这个 API 函数
    //使用_T函数解决"const char *" 类型的实参与 "LPCWSTR" 类型的形参不兼容的问题
    MessageBox(NULL,_T("hello snowming"),_T("Test"),0);
    return 0;
}
```



![title](https://leanote.com/api/file/getImage?fileId=5e9d1683ab6441104c031de5)



- 这里的 MessageBox 是 API 函数，它用于显示一个指定风格的对话框，其申明包含在 Windows.h 中。


##一个创建文件的程序

```
#include <iostream>
#include <Windows.h>
#include <string>
#include <tchar.h>


int main()
{
    TCHAR Path[255];   //文件路径
    TCHAR FileName[255];
    char Data[512] = "-----------by snowming-----------";
    for (int i = 0; i != 10; i++)
    {
        //得到Windows目录
        GetWindowsDirectory(Path, sizeof(Path));
        //用i的值加.txt来给文件命名
        wsprintf(FileName, _T("\\&d.txt"), i);
        //给Path赋以完整路径
        lstrcat(Path, FileName);
        HANDLE hFile;
        //创建文件
        hFile = CreateFile(Path, GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
        if (hFile == INVALID_HANDLE_VALUE)
        {
            continue;
        }
        DWORD dwWrite;
        //把Data中的数据写入文件
        WriteFile(hFile, &Data, strlen(Data), &dwWrite, NULL);
        //关闭文件句柄
        CloseHandle(hFile);
        memset(Path, 0x00, 255);
        memset(FileName, 0x00, 255);
    }
    return 0;
}
```
