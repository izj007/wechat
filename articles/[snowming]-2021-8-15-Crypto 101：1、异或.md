# 0x01 逻辑异或/单位异或

异或（XOR）本身是一个 `Boolean binary operator`。

- 接收两个参数
- 输入和输出都只能是 true 或 false
- 结果为 1（true） 或者 0（false）

异或运算到底做了什么？

`programmable inverter`：一个输入 bit 决定是否去倒置另一个输入 bit，或者仅仅是不加改变的保留。


倒置/翻转规则：

- 0 XOR 0 = 0
- 1 XOR 1 = 0
- 1 XOR 0 = 1
- 0 XOR 1 = 1

也就是当参数中的一个为真（但不是 both）时输出为真。


# 0x02 按位异或

上面的异或是操作的单位  bit 或布尔值。当我们处理包含多位（bit）的值时，大多数编程语言提供了对整数“按位异或”（bitwise XOR）的方法。

1. 将两个整数表示为二进制形式
2. 对参数对应位进行异或

如：


![title](https://leanote.com/api/file/getImage?fileId=5f69f1f7ab64415d46000b8e)

# 0x03 编程语言中的按位异或

**C/C++：**


C或C ++中的^（按位XOR）将两个数字用作操作数，并对两个数字的每一位进行XOR。如果两个位不同，则XOR的结果为1。

```
// C Program to demonstrate use of bitwise operators 
#include <stdio.h> 
int main() 
{ 
    // a = 5(00000101), b = 9(00001001) 
    unsigned char a = 5, b = 9; 
  
    // The result is 00001100 
    printf("a^b = %d\n", a ^ b); 
  
    return 0; 
} 

//输出：
a ^ b = 12
```


**c#：**


```
逻辑异或运算符^
该^操作符计算它的操作数的逻辑异或，也称为逻辑XOR。对于bool操作数，^运算符计算的结果与不等式运算符（!=）相同。

Console.WriteLine(true ^ true);    // output: False
Console.WriteLine(true ^ false);   // output: True
Console.WriteLine(false ^ true);   // output: True
Console.WriteLine(false ^ false);  // output: False
```


```
对于整数数值类型的^操作数，运算符计算其操作数的按位逻辑异或。
该^运算符计算按位逻辑异或，也被称为逐位逻辑XOR其整体操作数，：

uint a = 0b_1111_1000;
uint b = 0b_0001_1100;
uint c = a ^ b;
Console.WriteLine(Convert.ToString(c, toBase: 2));
// Output:
// 11100100
```

**Python：**
```
# print bitwise XOR operation   
print("a ^ b =", a ^ b)  
```


**VBA：**

逻辑异或：

![title](https://leanote.com/api/file/getImage?fileId=5f69fe6cab64415d46000bc4)

按位异或：

![title](https://leanote.com/api/file/getImage?fileId=5f69febeab64415d46000bc6)


**总结看出在编程语言中，逻辑异或和按位异或都是同一个符号。**