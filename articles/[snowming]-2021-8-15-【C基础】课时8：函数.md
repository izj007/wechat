**<u>本章要点</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dbc61ab6441609b0013fd)

**<u>strlen()函数</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dbc96ab644162980013ff)

**<u>不是基本数据类型的数组</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dc128ab6441609b0014fc)

list不是基本数据类型，这种数组在下一章会讲。

**<u>ANSI C风格的函数原型</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dc22fab6441609b00152e)

**<u>函数签名</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dc273ab6441629800151a)


**<u>编译器的工作</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dc346ab64416298001556)


**<u>定义带形式参数的函数</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dc5b4ab6441609b0015ef)

**<u>驱动程序</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dc8c8ab6441609b00168c)

**<u>返回值可以是表达式的值</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dca0eab6441609b0016d9)

**<u>当返回值的类型与函数声明的类型不匹配</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dcb09ab6441609b00170d)


**<u>在函数中使用多个return语句</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dcc6cab6441609b00175b)

**<u>为什么需要函数（类型）声明</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dcec1ab6441609b0017cc)


**<u>函数原型的位置</u>**

- 可以放在主调函数内，也可以放在主调函数外
- 主要要保证在函数调用之前声明

![title](https://leanote.com/api/file/getImage?fileId=5e5dcf18ab644162980017a9)

**<u>头文件中包含函数声明，但没有具体定义</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dcfc0ab6441609b0017f2)

**<u>ANSI C函数原型</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dd4f8ab644162980018db)


**<u>函数原型和函数定义的形参名称可以不一致</u>**


![title](https://leanote.com/api/file/getImage?fileId=5e5dd5a5ab6441609b00190c)

**<u>无参数和未指定参数</u>**


![title](https://leanote.com/api/file/getImage?fileId=5e5ddacbab64416298001a15)

- 无参数需要用 void 关键字说明，而不是直接给形参那个括号留空。

**<u>函数原型的意义</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5ddb52ab64416298001a34)


## 递归


**<u>递归的定义</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5ddc91ab6441609b001a82)

**<u>递归函数停止语句</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5df56dab64416298001f2f)


**<u>尾递归的定义</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5df5ffab6441609b001f0c)


**<u>一个尾递归函数</u>**


![title](https://leanote.com/api/file/getImage?fileId=5e5df696ab64416298001f6c)

![title](https://leanote.com/api/file/getImage?fileId=5e5df6c6ab6441609b001f35)


**<u>循环和递归的选择问题</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5df71bab6441609b001f45)


**<u>编译多源代码文件的程序</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dfbb0ab6441609b002033)


**<u>头文件的作用</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5dfeb8ab644162980020f2)



------------------------



**<u>查找地址：&运算符</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e0e0aab6441609b0023ce)


![title](https://leanote.com/api/file/getImage?fileId=5e5e0f81ab64416298002422)

- 所谓的主调函数就是 `main()` 函数。

--------------------


## 指针

**<u>指针的定义</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e127dab644162980024b2)



**<u>指针的使用</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e13b3ab6441609b0024e3)

- `变量`指向`指针`。

**<u>间接运算符：*</u>**


![title](https://leanote.com/api/file/getImage?fileId=5e5e14f1ab6441629800252f)


**<u>与指针相关的运算符</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e1560ab6441609b002532)

**<u>声明指针</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e2090ab6441629800277c)

![title](https://leanote.com/api/file/getImage?fileId=5e5e210aab6441609b00274f)

**<u>指针是一种单独的类型，不是整数类型</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e2150ab644162980027a2)


**<u>变量：名称、地址和值</u>**

变量三属性：

- 名称
- 地址
- 值

指针：

- 地址
- 值

变量：

- 名称
- 值

![title](https://leanote.com/api/file/getImage?fileId=5e5e2348ab6441609b0027be)


**<u>使用指针在函数间通信</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e2529ab6441609b00281d)

![title](https://leanote.com/api/file/getImage?fileId=5e5e2576ab6441609b00282b)

![title](https://leanote.com/api/file/getImage?fileId=5e5e25ffab6441629800287b)


**<u>scanf()函数</u>**

![title](https://leanote.com/api/file/getImage?fileId=5e5e2664ab6441609b002859)


**<u>传递值</u>**


![title](https://leanote.com/api/file/getImage?fileId=5e5e23eeab6441629800281c)


## 指针的作用

![title](https://leanote.com/api/file/getImage?fileId=5e5e2476ab64416298002836)

**函数签名**


![title](https://leanote.com/api/file/getImage?fileId=5e5e242cab6441629800282e)