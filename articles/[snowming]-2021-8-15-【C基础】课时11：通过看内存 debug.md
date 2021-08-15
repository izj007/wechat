# 0x01 问号

事情的起因是我在写一个程序的时候，遇到了一件怪事：


![title](https://leanote.com/api/file/getImage?fileId=5e6754b4ab6441339b006956)

就是，我打印 page 数组里的元素，除了第一个字符是正常输出，其他的都是`?`。

但是我这个 page 数组是通过 `get_strings()`这个函数来赋值的，我在 `get_strings()` 这个函数里面打印了这个 `page` 数组，完全正常。这说明赋值成功：

![title](https://leanote.com/api/file/getImage?fileId=5e6756bdab644135960068ca)


那为什么我在 main 函数里面打出来就是 `?`。

这让我想起了一句歌词：
>小问号，你是否有很多的朋友。为什么、别人在那看漫画，我却要学画画，以后长大了别人都抄我写的歌。

# 0x02 解决

经过咨询大佬，我很快得到了问题的原因：

在 main 函数中，声明 page 数组的时候，我一不小心把数组元素类型指定为了 `int`，而 page 数组作为用户输入的字符串其实是 `char` 类型数组。

>大佬说：看下内存对应的具体数据是啥？猜测是不可显示字符。<br/>
我说：应该不是不可显示字符。因为通过 `get_strings` 方法可以打印出来剩余字符，就是普通的字母而已。<br/>
大佬继续进行了解释：`%c` 输出问号，就表示是不可在屏幕上显示的字符。`main` 函数中的 page[] 数组是 int 数组，`get_strings()` 中的是 char 数组。1 int 占 4 bytes，1 char 占 1 byte，位置不一样，导致在对应的 int 中的 page[1] 其实不是你输入的数据。


总之，就是 page 数组的类型不一致导致的这个 bug，当我把 main 函数中的 page 数组的类型改为 char，问题就圆满解决了：


![title](https://leanote.com/api/file/getImage?fileId=5e675c0dab6441339b006ab9)

忽略这生涩稚嫩的代码......

# 0x03 看内存

找大佬请教了如何通过看内存 debug，大概明白了？记录一下：

先下两个断点：一个在 `get_strings()` 方法处，一个在 for 循环前。

![title](https://leanote.com/api/file/getImage?fileId=5e676f8eab6441339b006ea0)




然后点击`本地 Windows 调试器`：

![title](https://leanote.com/api/file/getImage?fileId=5e677095ab64413596006dc7)


打开 `调试` → `窗口` → `内存` → `内存1`；
打开 `调试` → `窗口` → `监视` → `监视1`；

现在 VS 出现了两个新窗口，一个内存窗口，一个监视窗口：

![title](https://leanote.com/api/file/getImage?fileId=5e677224ab64413596006e11)

在监视窗口添加项 `page`，按下回车，就出现了 page 数组的地址。

在内存窗口就此地址进行搜索：

![title](https://leanote.com/api/file/getImage?fileId=5e6772d2ab64413596006e39)

可以看到，已经为数组分配了内存地址，但是现在内存里面的数据还都是 `cc`，Debug 模式下默认初始变量就是 `cc`。

然后继续走，点`继续`，输入字符串 `abcde`：

![title](https://leanote.com/api/file/getImage?fileId=5e677499ab64413596006e96)

按下回车之后，可以看到 page 数组在内存中对应的 5 byte 有了值：

![title](https://leanote.com/api/file/getImage?fileId=5e67751dab64413596006ea9)



![title](https://leanote.com/api/file/getImage?fileId=5e67757cab64413596006ebd)

然后查看此时的每个元素的值：

<u>page[0]</u>

通过给监视窗口添加项 `page[0]`，可以获取 page[0] 元素的起始地址为 A0，又因为其类型为 1 指向int的指针，所以此元素占 4 bytes。那么它的值就是 61626364，但是输出的时候又通过 printf 函数的 %c 进行截断，所以是 61 就对应了 a。


![title](https://leanote.com/api/file/getImage?fileId=5e678b61ab6441339b0073f2)


<u>page[1]</u>

同理，page[1]的起始地址为第5个byte，就是65cccccc输出应该为 e：

![title](https://leanote.com/api/file/getImage?fileId=5e678b3eab6441339b0073ef)

<u>page[2]</u>

page[2] 都是 cc，所以输出 `?` 表示字符不可显示：


![title](https://leanote.com/api/file/getImage?fileId=5e678b8aab6441339b0073f6)

<u>page[3]</u>

page[3]同page[2]：

![title](https://leanote.com/api/file/getImage?fileId=5e67791dab64413596006f6a)

<u>page[4]</u>

page[4]同page[2]：

![title](https://leanote.com/api/file/getImage?fileId=5e677950ab64413596006f71)

然后继续走，for 循环的显示结果和分析一致：

![title](https://leanote.com/api/file/getImage?fileId=5e677a7aab6441339b0070c6)

所以其实就明白了为什么是这样的输出了，原因如下：

![title](https://leanote.com/api/file/getImage?fileId=5e677c37ab6441339b00711b)


# 0x04 验证

把 page 数组的类型改为 char。

![title](https://leanote.com/api/file/getImage?fileId=5e677d14ab64413596007025)

如上图，可以看到结果，从起始地址 F9A8 开始，每个元素占 1 byte：

- `&page[0]` == 0x00000082300FF5A8，page[0]的值为61，即 `a`；
- `&page[1]` == 0x00000082300FF5A9，page[0]的值为62，即 `b`；
- `&page[2]` == 0x00000082300FF5AA，page[0]的值为63，即 `c`；
- `&page[3]` == 0x00000082300FF5AB，page[0]的值为64，即 `d`；
- `&page[4]` == 0x00000082300FF5AC，page[0]的值为65，即 `e`。

# 0x05 总结

其实通过查看内存 debug 也没有那么神秘。

内存只是一块数据，是不会告诉我们 page[1] 是什么的，只会提供一个起始地址。还是需要我们自己先辨别数据类型、结合数据类型来解释它。

而解释是一件非常主观的事情，自己想怎么解释就怎么解释。

总之这是我第一次搞这个，手还很生，今后继续练习。

