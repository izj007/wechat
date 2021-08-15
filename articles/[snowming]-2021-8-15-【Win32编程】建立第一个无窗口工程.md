![title](https://leanote.com/api/file/getImage?fileId=5e9d0954ab64410e3d02f165)


```
// 无窗口工程.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
#include <Windows.h>
#pragma comment(linker, "/subsystem:\"Windows\" /entry:\"mainCRTStartup\"")

int main()
{
    std::cout << "Hello World!\n";
}
```


主要是通过：
```
#include <Windows.h>
#pragma comment(linker, "/subsystem:\"Windows\" /entry:\"mainCRTStartup\"")
```

在头文件中加入如下代码，就不会有控制台窗口了，不会像应用程序那样跳出一个很明显的窗口。


