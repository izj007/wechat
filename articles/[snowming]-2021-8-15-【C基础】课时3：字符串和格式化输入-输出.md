# 第4章：字符串和格式化输入/输出

**<u>本章要点</u>**

- 函数：strlen()
- 关键字：const
- 字符串
- 如何创建、存储字符串
- 如何使用strlen()函数获取字符串的长度
- 用C预处理器指令#define和ANSIC的const修饰符创建符号常量

## 示例程序

![title](https://leanote.com/api/file/getImage?fileId=5e493fc7ab6441522b007370)

**<u>1、scanf_s()的参数个数问题</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e49415fab6441522b0073b8)


如果是平时这样只给 scan_f()函数输入一个存值实参，实际运行的话，运行到这步用户输入就不会再执行了，VS也会警告：
>缺少“scanf_s”的整型参数(对应于转换说明符“2”)。	

查阅资料得知：

>  在使用VS时，应编译器要求需使用更加安全的scanf_s代替scanf，然而scanf_s函数在接收字符数组时有坑。
原因却是scanf_s在使用数组指针作为写入变量地址参数时，还需要加一个buffer长度参数，更安全（C程序员随时都要考虑内存溢出问题）。
<br>
![title](https://leanote.com/api/file/getImage?fileId=5e494346ab6441542b00742f)
<br>
参考资料：[VS2017使用scanf_s函数报错: (ucrtbased.dll)写入位置 0x00F6B000 时发生访问冲突。](https://blog.csdn.net/Learner027/article/details/88643958)

**也就是说，scanf_s()函数在传入字符或字符数组时候，需要传入2个参数，后一个参数是数组大小。**

占位符理论上使用`%s`或`%c`都可以。因为`%s`代表整数，字符实质上是整数（转为ASCII码或其他码）；`%c`就是很直接的代表char了。

但是最好不要使用`%c`，因为会乱码：

![title](https://leanote.com/api/file/getImage?fileId=5e49446dab6441542b007479)

原因大概是因为：本机的编译器不一定使用了ASCII编码，在作为字符转整数时候，会乱码。

还需要注意一点的是，如代码中的注释所写：因为是数组，所以传入`name`变量的时候，`name`前面可以省略掉`&`符号。

也就是下面两种写法都可以：

```
char name[40]; 

scanf_s("%s", &name,40); //写法1
scanf_s("%s", name,40);  //写法2
```

**<u>2、sizeof大小问题</u>**

```
char name[40]; //name是一个可容纳40个字符的数组

size = sizeof name; //输出在内存中所占字节数
letters = strlen(name); //输出字符串长度
```
sizeof计算name变量在内存中占的bytes数，要注意因为之前的声明，所以内存中会预留40字节的缓存区给name变量，所以所占一定是40字节。而跟实际的字符串长度(`strlen()`)无关。


![title](https://leanote.com/api/file/getImage?fileId=5e494010ab6441542b007372)

![title](https://leanote.com/api/file/getImage?fileId=5e494030ab6441522b007385)

如果输入值上溢也不会影响sizeof，只会影响strlen以及后面的代码部分：

![title](https://leanote.com/api/file/getImage?fileId=5e4947b0ab6441542b007509)

**<u>3、新特性</u>**

- 用C预处理器把字符常量DENSITY定义为62.4（头文件为string.h）。

---------------------------

**<u>char类型数组和null字符</u>**

C语言没有专门用于存储字符串的变量类型，字符串都被储存在char类型的数组中。数组由连续的存储单元组成，字符串中的字符被储存在相邻的存储单元中，每个单元储存一个字符。

![title](https://leanote.com/api/file/getImage?fileId=5e49f9acab644117cc001483)



如图，如果输入40个字符，会报错：

![title](https://leanote.com/api/file/getImage?fileId=5e49fa28ab644115d200146c)

> 一般而言，C把函数库中相关的函数归为一类，并为每类函数提供一个头文件。例如，`printf()`和`scanf()`都隶属标准输入和输出函数，使用`stdio.h`头文件。`string.h`头文件中包含了`strlen()`函数和其他一些与字符串相关的函数(如拷贝字符串的函数和字符串查找函数)。

**<u>字符和字符串</u>**

字符串常量`"x"`和字符常量`'x'`不同。区别在于：

1.  `'x'`是基本类型(char)，而`"x"`是派生类型(char数组)；
2. `"x"`实际上由两个字符组成：`'x'`和空字符`\0`。

**<u>strlen()函数</u>**

**<u>常量和C预处理器</u>**

常量主要是为了弥补以下缺陷：

- 声明变量可以提供一个符号名，但是毕竟是变量，程序可能会无意间改变它的值。
- 所以C语言提供了一个更好的方案——C预处理器。

C预处理器的功能：

 1.  使用`#include`包含其他文件的信息。
 2. 定义常量。(编译时替换、明示常量)

格式：

```
#define NAME value  //没有等号
```
  
预处理器的作用：

1. 预处理源文件：
    - 替换常量。
    - 加载头文件及其中函数
2. 处理好之后再交给编译器


**<u>%1.2f</u>**

%m.nf

m是整个数据的宽度，n是小数点的位数。

`1.2f`是非法的，小数点前的数字必须大于小数点后的数字。小数点前的数值规定了打印的数字的总宽度。


`1.2f`指总位数1位，保留小数点后2位数字，浮点数形式输出。由于小数点后就有两位数，总位数1位这个约束条件实际上是无效的。

`%3.1f`的意思是输出float型数据，保留1位小数，并尽量使整个输出至少占用3个字符的位置（其中小数点也算1个位置）。

至于`1.2f`，除了保留2位小数以外，小数点前面的1在输出中不会起什么作用的，因为输出的数怎么也不会比1个字符少。


**<u>const关键字</u>**

- 限定一个变量为只读。
- 声明为：
```
const int MONTHS = 12;
```

- 注意，在C语言中，用const类型限定符声明的是变量，不是常量。

**<u>printf()</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e4a17bdab644117cc001a12)



**<u>从scanf()角度看输入</u>**


![title](https://leanote.com/api/file/getImage?fileId=5e4cdd5cab6441621b00200e)

