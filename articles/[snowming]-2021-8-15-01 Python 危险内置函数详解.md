# 0x01 前言
时间的车轮滚滚而过，我终于学到代码审计了。


------------------------

# 0x02 Python 危险内置函数

在 **Python3** 里面，主要有：

 - `eval()`
 - `exec()`
 - `compile()`

![title](https://leanote.com/api/file/getImage?fileId=5db676ecab644148470005f7)


> 注：
python3 删去了 `execfile()` 函数，代替方法如下：
``` shell
with open('test1.py','r') as f:
    exec(f.read())
```

在 **Python2** 里面，主要有：

 - `eval()`
 - `execfile()`
 - `compile()`


 ![title](https://leanote.com/api/file/getImage?fileId=5db676c9ab64414a420005ad)

> 注：
需要说明的是在 Python2 中 `exec` 不是函数，而是一个内置语句(statement)，但是 Python 2 中有一个 `execfile()` 函数。可以理解为 Python 3 把 `exec` 这个 statement 和 `execfile()` 函数的功能够整合到一个新的 `exec()` 函数中去了。

------------------------

# 0x03 eval() 函数（执行单个表达式）

功能： 将字符串当成有效的表达式来求值并返回计算结果。

语法：
``` shell
eval(expression, globals= None, locals= None)
```

> 官方文档中的解释：
将字符串 str 当成有效的表达式来求值并返回计算结果。
globals（全局）和 locals（局部）参数是可选的，如果提供了globals参数，那么它必须是 dictionary 类型；
如果提供了 locals 参数，那么它可以是任意的 map 对象。

eval() 函数三个参数的用法可以参考此两篇博客，写的很清楚了：

- [Python-eval()函数](https://www.cnblogs.com/Xuuuuuu/p/10127029.html)
- [十五、python沉淀之路--eval()的用法](https://www.cnblogs.com/jianguo221/p/8975899.html)


总结来说，eval() 函数的第二、三个参数，也就是 `globals` 和 `locals` 是可以省略的，如果传入了，它们的作用是定义作用域的。`globals` 代表作用域为全局、`locals` 代表作用域为局部。如有冲突，以 `locals` 的作用域为准 。

> **第二三个参数分别指定能够在 eval 中使用的函数等，如果不指定，默认为 globals() 和 locals() 函数中包含的模块和函数。 -- [python 为什么说eval要慎用](https://www.jb51.net/article/158470.htm)**

## 危险用法 -- 命令执行

**注意：**

eval只能执行 Python 的表达式类型的代码，不能直接用它进行 import 操作，但 exec 可以。如果非要使用 eval 进行 import，则使用 `__import__` 来动态加载 import 模块：

**执行命令：**

![title](https://leanote.com/api/file/getImage?fileId=5db696b1ab644148470006d5)

> 注：
使用 os.system("cmd")，其执行过程中会输出显示 cmd 命令执行的信息。
例如：print os.system("mkdir test") >>>输出：0
可以看到结果打印出0，表示命令执行成功；否则表示失败（再次执行该命令，输出：子目录或文件 test 已经存在。1）。

**获取 shell：**
``` shell
eval(__import__('os').system('sh'))
```
![title](https://leanote.com/api/file/getImage?fileId=5db69522ab644148470006ba)

![title](https://leanote.com/api/file/getImage?fileId=5db693d1ab644148470006af)
图片来源：[Python2 内置函数](https://docs.python.org/2.7/library/functions.html)

更多关于 eval() 函数的危险性可以参考：[Python 中 eval 的强大与危害](https://blog.csdn.net/liuchunming033/article/details/87643041)

## eval() 限制

`eval`只接受一个**单个表达式**，不接受一个包含Python语句的代码块：loops，try：except：，class 和函数/方法`def`定义等等。

python 中的表达式就是变量赋值中的值：
a_variable = (任何你可以放在这些括号内的是一个表达式)

![title](https://leanote.com/api/file/getImage?fileId=5db6a5f4ab64414a42000779)

------------------------

# 0x04 exec() 函数（执行多行代码）

**定义:**

exec() 函数用于执行指定的 Python 代码。对象必须是字符串或代码对象。

该 exec() 函数接受较大的代码块，与 eval() 仅接受单个表达式的函数不同。

**注意 1：**

要注意内存中是否已经加载了要使用的函数，查看方法为：

`print(dir())`

只有加载了的函数才可以使用。


![title](https://leanote.com/api/file/getImage?fileId=5db6b3f3ab64414a42000916)
在上图中，`from math import *`之后，内存加载了`math`模块中的所有方法。

![title](https://leanote.com/api/file/getImage?fileId=5db6b625ab644148470009de)

**注意 2：**

如果有多个要执行的命令。使用 `\n` 连接：

![title](https://leanote.com/api/file/getImage?fileId=5db81c7cab644113360004b7)

## 危险用法 -- 命令执行

![title](https://leanote.com/api/file/getImage?fileId=5db82003ab644113360004ca)


------------------------

# 0x05 execfile() 函数（执行文件中的Python代码）

**语法：**

``` shell
execfile(filename[, globals[, locals]])
```

**参数：**

- `filename` -- 文件名，必传参数。
- `globals` -- 可选参数。全局变量--这里指绝对路径。如果被提供，则必须是一个字典对象。
- `locals` -- 可选参数。本地变量--这里指相对路径。如果被提供，可以是任何映射对象。

参考：[python之函数用法execfile()](https://www.cnblogs.com/dengyg200891/p/4945723.html)


**返回值：**

返回表达式执行结果。

**实例：**

- 执行单行表达式：

![title](https://leanote.com/api/file/getImage?fileId=5db8f5c3ab6441112e000794)

- 执行多行 Python 代码：

![title](https://leanote.com/api/file/getImage?fileId=5db8f74aab6441133600076d)

## 危险用法 -- 命令执行

![title](https://leanote.com/api/file/getImage?fileId=5db900e1ab644113360007cb)


## 在 Python3 中使用 execfile()

Python3 删去了 execfile()，代替方法如下：

``` shell
with open('test1.py','r') as f:
  exec(f.read())
```

![title](https://leanote.com/api/file/getImage?fileId=5db925d5ab6441112e0008c7)


-----------------------------


# 0x06 compile() 函数（必须和执行函数搭配使用）

[Python compile函数有什么用？](https://segmentfault.com/q/1010000017171114)


功能：

`compile()` 函数将一个字符串编译为字节代码。

语法：

``` shell
compile(source, filename, mode, flags=0, dont_inherit=False, optimize=-1)
```

参数：

| 参数      |    说明  | 
| :--:      |  :--    |
| source    | 字符串或者AST（Abstract Syntax Trees）对象 | 
| filename    |  代码文件名称，如果不是从文件读取代码则传递一些可辨认的值 |  
| mode	|   指定编译代码的种类。可以指定为 exec, eval, single|
|flags|变量作用域，局部命名空间，如果被提供，可以是任何映射对象 |
|dont_inherit|是用来控制编译源码时的标志|

返回值：

返回表达式执行结果。



**官方文档 [Python2 内置函数](https://docs.python.org/2.7/library/functions.html) 中对 compile() 函数的说明：**

- **source 参数**
`compile()` 函数会将 `source` 编译成代码或 `AST` （抽象语法树）对象。代码对象可以被 `exec()` 或 `eval()` 执行。`source` 可以是常规的字符串、**字节字符串**，或者 `AST` 对象。

- **filename 参数**
`filename` 实参需要是代码读取的文件名；如果代码不需要从文件中读取，可以传入一些可辨识的值（经常会使用 `'<string>'`）。

- **mode 参数**
`mode` 实参指定了编译代码必须用的模式。如果 `source` 是语句序列，可以是 `'exec'`；如果是单一表达式，可以是 `'eval'`；如果是**单个交互式语句**，可以是 `'single'`。（在最后一种情况下，如果表达式执行结果不是 `None` 将会被打印出来。）

- flags 参数和 dont_inherit 参数
可选参数 `flags` 和 `dont_inherit` 控制在编译 `source` 时要用到哪个 `future` 语句（future 语句 是一种针对编译器的指令，指明某个特定模块应当使用在特定的未来某个 Python 发行版中成为标准特性的语法或语义）。
在我们的目的中可以暂时忽略这两个参数。

- optimize 参数
optimize 实参指定编译器的优化级别；默认值 -1 选择与解释器的 -O 选项相同的优化级别。显式级别为 0 （没有优化；__debug__ 为真）、1 （断言被删除， __debug__ 为假）或 2 （文档字符串也被删除）。

- **触发异常**
如果编译的源码不合法，此函数会触发 `SyntaxError` 异常；如果源码包含 null 字节，则会触发 `ValueError` 异常。

- 注解：在 'single' 或 'eval' 模式编译多行代码字符串时，输入必须以至少一个换行符结尾。这使 code 模块更容易检测语句的完整性。

- 在 3.2 版更改: 允许使用 Windows 和 Mac 的换行符。在 'exec' 模式不再需要以换行符结尾。增加了 optimize 形参。

- 在 3.5 版更改: 之前 source 中包含 null 字节的话会触发 TypeError 异常。

## 实例

![title](https://leanote.com/api/file/getImage?fileId=5db91589ab6441133600083c)


## 危险用法 -- 命令执行


![title](https://leanote.com/api/file/getImage?fileId=5db91ac8ab6441133600084e)







参考链接：

[1] [Python2 内置函数](https://docs.python.org/2.7/library/functions.html)
[2] [深入理解 Python 中的 builtin 和 builtins](https://blog.51cto.com/xpleaf/1764849)
[3] [`__import__`详解](https://www.jianshu.com/p/e7ee9b2c83b9)
[4] [Python` __import__`() 函数](https://www.runoob.com/python/python-func-__import__.html)
[5] [python执行系统命令的方法：os.system(), os.popen(), subprocess.Popen()](https://blog.csdn.net/taohuaxinmu123/article/details/48828255)
[6] [Python：exec() 函数](https://www.w3resource.com/python/built-in-function/exec.php)
[7] [Python中的exec()](https://www.geeksforgeeks.org/exec-in-python/)
[8] [Python内置函数(12)——compile](https://www.jianshu.com/p/c09df3f53441)
[9] [Python compile() 函数](https://www.runoob.com/python/python-func-compile.html)
[10] [Python compile函数有什么用？](https://segmentfault.com/q/1010000017171114)
[11] [Python execfile() 函数的用法及实例](https://www.imaizx.com/726.html)
[12] [python之函数用法execfile()](https://www.cnblogs.com/dengyg200891/p/4945723.html)