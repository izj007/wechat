CS （测试版本 4.0）生成的默认宏样本，如下定义此结构体：

```
Private Type PROCESS_INFORMATION
    hProcess As Long
    hThread As Long
    dwProcessId As Long
    dwThreadId As Long
End Type
```

但其实：

```
hProcess As Long
hThread As Long
```

在64位机器环境下均定义错误。

因为 64 位环境下， VBA 中 Long 仅为4字节，如果按照这个宏样本去运行，取第三个参数的值，就会发现并不是正确的 PID：

```
Debug.Print pInfo.dwProcessId
```

但 CS 的默认宏样本却能正确运行，这是因为 hProcess 不过是一个进程句柄，是可以被4字节容纳下的。但是错误的内存大小定义会影响到占位和第三个参数的值，所以应该根据系统选择性的定义：


```
#If VBA7 And Win64 Then
Private Type PROCESS_INFORMATION
    hProcess As LongPtr
    hThread As LongPtr
    dwProcessId As Long
    dwThreadId As Long
End Type

#Else
Private Type PROCESS_INFORMATION
    hProcess As Long
    hThread As Long
    dwProcessId As Long
    dwThreadId As Long
End Type

#End If
```

注： `LongPtr` 在64位系统中为8字节。