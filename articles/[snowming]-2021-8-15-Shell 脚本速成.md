# 0x00 前言

这段时间快速把 Micropoor 的内网课程看完了一遍，里面出现了很多 Shell 脚本。

Shell 脚本有什么好处？

1. 无需安装其他软件
2. 适合任务自动化，擅长系统管理任务

通过 Shell 编程，大大提高渗透效率。


# 0x01 第一个 shell 脚本

功能：启动 msfconsole

``` shell
vim start_msf.sh
chmod +x start_msf.sh
# 运行
./start_msf.sh
```
start_msf.sh 的具体内容：
``` shell
#!/bin/sh
msfconsole
```

# 0x02 引入变量

功能：输出一个变量名

``` shell
touch test.sh
chmod +x test.sh
# 运行
./test.sh
```

test.sh 的具体内容：
``` shell
#!/bin/sh
name='变量名'
echo $name
```
注意：`=`前后不能有空格，否则就会出现 `./test.sh: 2: name: not found`这个错误，也就是说变量定义会失败。



有时候变量名可能会和其它文字混淆，如下代码：
``` shell
#!/bin/sh
num=2
echo "this is the $numnd"  
```

上述脚本并不会输出 `this is the 2nd`，只会打印 `this is the`；这是由于 shell 会去搜索变量 numnd 的值，而实际上这个变量此时并没有值。


修改方法：用花括号圈定变量名：
```shell
#!/bin/sh
# 这是一个注释
num=2
echo "this is the ${num}nd"
```

注意 shell 脚本的注释是 `#`。


# 0x03 for 循环

for var in ....; do .... done   
``` shell
#!/bin/sh
for var in A B C; do 
    echo "var is $var" 
done 
```
![title](https://leanote.com/api/file/getImage?fileId=5ddfbd24ab64411a00006d71)

注：sh 不支持 C 语言风格的 for 循环写法，所以下面的脚本一定要把 shell 指定为 `bash`。参考：[shell脚本：Syntax error: Bad for loop variable错误解决方法](https://blog.csdn.net/liuqinglong_along/article/details/52191382)

``` shell
#!/bin/bash
for ((var=0;var<=3;var++)); do 
    echo "var is $var" 
done 
```

![title](https://leanote.com/api/file/getImage?fileId=5ddfc268ab64411808006ea8)


上面的脚本更 `shellish` 的写法是：


``` shell
#!/bin/bash
for var in `seq 3`; do 
    echo "var is $var" 
done 
```

![title](https://leanote.com/api/file/getImage?fileId=5ddfc3d9ab64411808006ef9)

注意通过上面通过两个「`」引入了命令，直接使用命令执行的结果。


## 0x04 while 循环

语法：

``` shell
while condition
do
	command
done
```

**测试命令**：

可以使用**测试命令**来对条件进行测试。比如可以比较字符串、判断文件是否存在及是否可读等等...

通常用`[]`来表示条件测试，注意这里的空格很重要，要确保方括号前后的空格。



- `[ -f "somefile" ]`：判断是否是一个文件
- `[ -x "/bin/ls" ]`：判断/bin/ls是否存在并有可执行权限
- `[ -n "$var" ]`：判断`$var`变量是否有值
- `[ "$a" = "$b" ]`：判断`$a`和`$b`是否相等  



示例代码：

``` shell
#!/bin/bash
COUNTER=0
while [ $COUNTER -lt 5 ]
do
    COUNTER=$((COUNTER + 1))
    echo $COUNTER
done
```

![title](https://leanote.com/api/file/getImage?fileId=5ddfcd12ab644118080070a4)

注意：

1. `[` 后和 `]` 前要空格。参考：[“[0: command not found” in Bash [duplicate]](https://stackoverflow.com/questions/42558479/0-command-not-found-in-bash)
2. `lt` 即为 `less than`，小于。



# 0x05 if 语句

语法：

``` shell
if ....; then
    ....
elif ....; then
    ....
else
    ....
fi
```

$SHELL 变量：

![title](https://leanote.com/api/file/getImage?fileId=5ddfd1aeab64411a00007123)

注意：上面的 SHELL 必须大写。变量 $SHELL 包含了登录 shell 的名称。


``` shell
#!/bin/sh
if [ "$SHELL" = "/bin/bash" ]; then
    echo "bash"
else
    echo "your login shell is $SHELL"
fi
```

![title](https://leanote.com/api/file/getImage?fileId=5ddfd53cab64411808007232)


注：再次注意 `[]` 前后的空格问题。不然结果可能出错。

# 0x06 函数

函数的主要使用场景是代码复用。函数定义部分应该写在一个 Shell 脚本的开头。


``` shell
# 定义
functionName() 
{
body
}
# 调用
functionName
```


## 无返回值函数：

``` shell
#!/bin/bash

firstFunction(){
    echo "1 try!"
}
firstFunction
```

![title](https://leanote.com/api/file/getImage?fileId=5ddfd931ab64411a00007288)


## 有返回值函数：

注：`read var` 命令：提示用户输入，并将输入内容赋值给变量 var
``` shell
#!/bin/bash

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```

![title](https://leanote.com/api/file/getImage?fileId=5ddfdbd1ab64411808007372)

函数返回值在调用该函数后通过 `$?` 来获得。注意：`$?` 仅对其上一条指令负责，一旦函数返回后其返回值没有立即保存入参数，那么其返回值将不再能通过 `$?` 获得。

注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至 shell 解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。



## 函数传参

在 Shell 脚本中，调用函数时可以向其传递参数。在函数体内部，通过 `$n` 的形式来获取参数的值，例如，`$1`表示第一个参数，`$2`表示第二个参数......当 n >=10 时，需要使用 `${n}` 来获取参数。

带参数的函数示例：

```
#!/bin/bash
# author:Snowming

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

![title](https://leanote.com/api/file/getImage?fileId=5ddfde2eab64411a00007379)


注意，`$10` 不能获取第十个参数，获取第十个参数需要 `${10}`。当 n >=10 时，需要使用 `${n}` 来获取参数。

另外，还有几个特殊字符用来处理参数：

![title](https://leanote.com/api/file/getImage?fileId=5ddfdeb2ab644118080073f9)



# 0x07 后记
 
先学这么多，就基本具备了写简单的 Shell 脚本的能力以及读懂别人的 shell 脚本的能力。
 

 
其实 Shell 脚本并不难，无非是多条 Linux 命令合到一起，加了一些控制语句、条件控制、变量等。
 
Shell 脚本语法坑多，以后想必会遇到不少问题。剩下的语法，也在实践中慢慢补充。

------------------

参考链接：

1. [Shell 函数](https://www.runoob.com/linux/linux-shell-func.html)，菜鸟教程
2. [shell 基础，提取码 gexj](https://pan.baidu.com/s/18rWmztroEQriWAaV-gQrDA)
