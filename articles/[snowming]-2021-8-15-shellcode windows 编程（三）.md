## ① 项目代码

根据 https://www.freebuf.com/articles/web/231663.html 一文中展示的 shellcode 生成框架、实现了基于 Win32 API 的远程文件下载功能。

项目代码位于：
https://github.com/Snowming04/shellcode_generate-test_framework

因为此文中原理论述足够清楚，故本文中不再谈论。


## ② 编译选项

本次工程创建是win32空项目，使用的是VS2019编译relase/x86，编译之前进行如下设置：

1. 创建工程时关闭SDL检查
2. 属性->C/C++->代码生成->运行库->多线程 (/MT)如果是debug则设置成MTD
3. 属性->常规->平台工具集->设置为Visual Studio 2017- Windows XP (v141_xp)，如果没有则可以去安装上对应的兼容xp的组件
4. 属性->C/C++->代码生成->禁用安全检查GS
5. 关闭生成清单 属性->链接器->清单文件->生成清单 选择否
6. C/C++ -> 语言 -> 禁用语言扩展 选择否
7. C/C++ -> 语言 -> 符合模式 选择否

## ③ 项目结构
头文件：

- api.h：在此 头文件中定义实现 shellcode 功能用到的函数对应的签名，以及结构体
- header.h：shellcode 生成框架中的函数原型

源代码文件：

- 0.entry.cpp
- a.start.cpp
- b.work.cpp：shellcode 功能代码
- z.end.cpp

其他文件：

- SCer.exe：快速测试 shellcode 功能的工具



## ④ 效果展示


![title](https://leanote.com/api/file/getImage?fileId=606c40ceab644179a6000552)

![title](https://leanote.com/api/file/getImage?fileId=606c4106ab644179a6000554)

![title](https://leanote.com/api/file/getImage?fileId=606c4178ab644177a10004d8)


## ⑤ 参考文档


- [Shellcode编程——编写自己想要功能的Shellcode](https://www.freebuf.com/articles/web/231663.html)