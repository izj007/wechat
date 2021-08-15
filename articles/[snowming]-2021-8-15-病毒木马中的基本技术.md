大多数的病毒木马在成功植入用户计算机之后，在执行核心恶意代码之前，会先进行一些初始化操作。如：

- 运行单一实例
- DLL 延迟加载
- 资源释放

本文讨论「运行单一实例」这一操作。


**<u>作用</u>**


如果病毒木马被多次重复运行，系统中就会存在多份病毒木马的进程，这会增加暴露的风险。欲解决此问题，就要确保系统上只运行一个病毒木马的进程实例。


确保仅运行一个进程实例的实现方法有很多。如：

- 扫描进程列表
- 枚举程序窗口
- 共享全局变量

这里尝试一种使用广泛且简单的方法，即通过「创建系统命名互斥对象」的方式来实现。

**<u>函数</u>**

`CreateMutex` 函数

![title](https://leanote.com/api/file/getImage?fileId=5e9e7500ab6441549600e8cc)


**<u>程序实现</u>**

- VS 2019
- 控制台应用（`C++`）

```
#include <Windows.h>
#include <iostream>
#include <tchar.h>
BOOL IsAlreadyRun()
{
	HANDLE hMutex = NULL;
	hMutex = ::CreateMutex(NULL,FALSE,_T("TEST"));
	if (hMutex)
	{
		if(ERROR_ALREADY_EXISTS == ::GetLastError())
		{
			return TRUE;
		}
		return FALSE;
	}

}
int main() {
	if (IsAlreadyRun() == FALSE)
		std::cout << "Not Already Run!" << std::endl;
	else
		std::cout << "Already Run!!!" << std::endl;
	//“请按任意键继续”而不是直接退出
	system("pause");
	return 0;
}
```



![title](https://leanote.com/api/file/getImage?fileId=5e9da043ab6441104c0523f5)


**<u>程序分析</u>**


![title](https://leanote.com/api/file/getImage?fileId=5e9e7575ab6441549600e9b3)

![title](https://leanote.com/api/file/getImage?fileId=5e9e7588ab6441549600e9d9)







