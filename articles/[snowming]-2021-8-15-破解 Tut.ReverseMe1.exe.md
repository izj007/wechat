# 0x01 运行

![title](https://leanote.com/api/file/getImage?fileId=5efc2146ab64411c2800070c)


点击确定之后：

![title](https://leanote.com/api/file/getImage?fileId=5efc2207ab64411c28000715)


![title](https://leanote.com/api/file/getImage?fileId=5efc2213ab64411e2600063e)


![title](https://leanote.com/api/file/getImage?fileId=5efc221fab64411e2600063f)


根据提示，破解的目标为：

- 删掉所有的废话（去除 Nag Screen）
- 找到正确的注册码

# 0x02 去除消息框


x32dbg 打开程序。这个是一个 VB 代码文件，从其依赖的 MSVBVM50.dll 就可以看出。或者从地址 0040116D 处的指令 call ThunRTMain() 也可以看出这是 Visual Basic 代码写的程序。


![title](https://leanote.com/api/file/getImage?fileId=5efc2263ab64411e26000645)



找到了 EP，打一个断点。从地址 0040116D 处的指令 F8 步过（不能 F7 步进，不然就进入了 ThunRTMain() 函数内部，分析的不是程序代码，而是 VB 引擎代码了）：



![title](https://leanote.com/api/file/getImage?fileId=5efc2281ab64411e26000647)


指令 00402C17 和指令 00402C18 处，是典型的栈帧结构，函数的开头。
指令 00402C85 和指令 00402CBE 处，是把 MessageBox 的两个参数以逆序方式压入栈。


![title](https://leanote.com/api/file/getImage?fileId=5efc22a9ab64411e26000649)


地址 00402CFE 处，调用了 rtcMsgBox() 函数，这是 VB 的消息框函数。在此处下断点。F9 运行程序，此处调用的是这个刚打开程序时候的 Nag Screen：


![title](https://leanote.com/api/file/getImage?fileId=5efc22bfab64411e2600064c)


点击确定回到主页面，然后点击 `Nag?` 按钮。


![title](https://leanote.com/api/file/getImage?fileId=5efc22efab64411e2600064f)


EIP 跳到了 ThunRTMain() 函数的头部，栈帧结构的地方。

![title](https://leanote.com/api/file/getImage?fileId=5efc2314ab64411e26000652)


继续运行，发现同样是跳到地址 00402CFE 处的指令，调用 rtcMsgBox 函数：

![title](https://leanote.com/api/file/getImage?fileId=5efc2333ab64411c28000723)


查看程序所有调用的所有 API 目录。

![title](https://leanote.com/api/file/getImage?fileId=5efc234cab64411c28000727)


有4处代码调用了 rtcMsgBox() 函数，给它们全都打上断点：


![title](https://leanote.com/api/file/getImage?fileId=5efc2376ab64411c2800072a)

运行发现，弹出 Nag Screen 的多处代码具有相同的运行代码，也就是地址 00402CFE 处的 call rtcMsgBox。所有只需要对这一处打补丁即可。


# 0x03 打补丁：去除消息框


需要修改地址 00402CFE 处的 call 命令：


```
00402CFE | E8 1DE4FFFF    | call <JMP.&rtcMsgBox>     |
```

上面的 CALL 指令，大小为5字节（E8 1D E4 FF FF）。

修改为如下代码：

![title](https://leanote.com/api/file/getImage?fileId=5efc23e6ab64411e26000655)

![title](https://leanote.com/api/file/getImage?fileId=5efc23f4ab64411c2800072f)


其中：add 指令大小为3字节（83 C4 DE），剩余2字节用 NOP 指令填充（以保证代码不会乱）。

**为什么是 add 14 呢？这主要是为了平衡栈。**
rtcMsgBox() 函数有五个参数：

![title](https://leanote.com/api/file/getImage?fileId=5efc2424ab64411e2600065a)
>注：Win32 API MessageBoxW() 也是四个参数加一个返回值，所以也是5个参数。

每个参数占4字节：
![title](https://leanote.com/api/file/getImage?fileId=5efc2466ab64411c28000734)

4*5=20，10进制的20就是16进制的14。
往栈底方向加 14，就可以平衡栈帧。

再者从 00402CFE 地址处继续 F8 步过执行，发现在地址 74194174 处有一个 ret 14 语句。这说明函数约定调用为 stdcall。这里的 ret 14 也是为了平衡栈。这也验证了我们计算的 14 是正确的。
![title](https://leanote.com/api/file/getImage?fileId=5efc2477ab64411c28000736)


总之现在我们把地址 00402CFE 处的 call 指令改为了 add 指令和两个 nop 指令：

![title](https://leanote.com/api/file/getImage?fileId=5efc249fab64411c2800073a)

打完这五个补丁之后的程序，运行却发生问题。
![title](https://leanote.com/api/file/getImage?fileId=5efc24bfab64411e26000661)


原因在于没有正确处理 rtcMsgBox() 函数的返回值（EAX 寄存器）。如图，在 00402CFE 地址处调用 rtcMsgBox() 函数后，00402D0C 处将返回值 EAX 存储到特定变量 [EBP-9C]。
![title](https://leanote.com/api/file/getImage?fileId=5efc24dcab64411c2800073e)
此处消息框的返回值应该是0（表示“确定”按钮）。若存储的为0之外的值，则表示程序终止。
![title](https://leanote.com/api/file/getImage?fileId=5efc2500ab64411c28000740)


沿着这个思路如果我们把 call 命令改为：
````
ADD ESP,14
MOV EAX,0
```
但是这样8个字节，超出了 call 命令的大小，会侵占到后面的代码。所以此路不通，得换一种思路。

在 00402CFE call rtcMsgBox() 指令往上翻，找到了地址 00402C17 处函数开始的栈帧：

![title](https://leanote.com/api/file/getImage?fileId=5efc2523ab64411e26000665)

地址 00402CFE 处的 call rtcMsgBox() 指令也是属于此函数内部的代码。所以如果上层函数无法调用或直接返回，那么最终将不会调用 rtcMsgBox 函数。想到将 00402C17 处的指令修改为 RETN XX，达到栈的平衡即可。

那么如何确定调整栈的这个参数大小呢？
要根据传递给函数的参数大小。


![title](https://leanote.com/api/file/getImage?fileId=5efc2547ab64411c28000743)


确认 402C17 函数的起始代码存储在栈中的返回地址为 740DE5A9。

进入返回地址 740DE5A9，此代码区域是 MSVBVM50.dll 模块区域。执行 740DE5A7 处的 CALL EAX 指令后即返回 740DE5A9 地址处。


![title](https://leanote.com/api/file/getImage?fileId=5efc257bab64411e26000668)


设置断点后再次运行调试器，可以得知 EAX 的值为 00402656。
![title](https://leanote.com/api/file/getImage?fileId=5efc25a0ab64411c28000747)

转到 00402656 处，最终跳转到 00402C17 处：

![title](https://leanote.com/api/file/getImage?fileId=5efc25b8ab64411c28000749)
![title](https://leanote.com/api/file/getImage?fileId=5efc25c5ab64411c2800074c)


综上可以看出， 740DE5A7 处的 CALL EAX 指令最终调用的是  00402C17 处的函数。我们从 00402C17 处一路 F8 步过执行，
![title](https://leanote.com/api/file/getImage?fileId=5efc25ddab64411e2600066e)


最终在地址 00402DB2 处看到了 ret 4，由于使用的是 stdcall 调用方式，所以栈由调用者负责清理。所以知道了参数个数为1（大小为4）。
![title](https://leanote.com/api/file/getImage?fileId=5efc25f6ab64411c2800074e)


所以回到地址 00402C17 处：
![title](https://leanote.com/api/file/getImage?fileId=5efc260fab64411c28000751)

![title](https://leanote.com/api/file/getImage?fileId=5efc261cab64411c28000752)

应用了这三个补丁之后，再次执行程序，果然就没有 Nag Screen 了。点击 `Nag?` 按钮也没有响应事件。

![title](https://leanote.com/api/file/getImage?fileId=5efc2636ab64411c28000753)

# 0x03 打补丁：查找注册码

查找正确的注册码的话，已知输入了错误的注册码会弹出错误提示框。搜索 `Sorry! Wrong registration code` 这一字符串。

![title](https://leanote.com/api/file/getImage?fileId=5efc2663ab64411c28000756)

找到了其位于地址 00402A69 处。

![title](https://leanote.com/api/file/getImage?fileId=5efc2728ab64411e2600067e)

根据经验，上面一定有一个条件跳转语句，还有一个比较字符串的 VB 函数。于是往上翻：
![title](https://leanote.com/api/file/getImage?fileId=5efc278bab64411c28000767)
就在不远的 00402A2A 处有一个字符串参数 `I'mlena151`，然后在地址 00402A2F 处有 __vbaStrCmp() 函数，__vbaStrCmp()  API 是 VB 中比较字符串的函数。到此几乎可以确定正确的注册码就是  `I'mlena151`。


![title](https://leanote.com/api/file/getImage?fileId=5efc27c1ab64411c28000768)


![title](https://leanote.com/api/file/getImage?fileId=5efc27d1ab64411c28000769)


**注册码验证成功！至此此程序破解完成。**