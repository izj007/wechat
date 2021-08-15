C的项目是这样组织的：

1、usehotel.c 

> - 主流程控制文件，里面包含main()函数
- 需要通过 #include包含头文件 hotel.h
- 使用了hotel.c里面的一些函数，主要放函数调用

2、 hotel.c

> - 是一些自定义功能函数的合集
- 主要是放具体函数定义的。


3、hotel.h

> - 头文件，主要放define的符号常量和hotel.c中所有的函数原型。

函数的三部曲：原型、定义、调用就这样被组织起来了。

使用的时候：

- 把 usehotel.c和hotel.c放在一个项目下编译（IDE 前提下），这样就无需跟python一样import函数，即可相互调用。
> 可以感觉到，使用vs studio 这种IDE时候，每次运行是运行了项目下所有的文件，都没有错误时候才可运行。

- usehotel.c和hotel.c都#include "hotel.h"。

这样就把三个文件紧密的结合起来了。


**<u>usehotel.c：</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e0333ab6441609b0021b0)

**<u>hotel.c：</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e035cab6441609b0021b6)

**<u>hotel.h：</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e0402ab644162980021e9)

