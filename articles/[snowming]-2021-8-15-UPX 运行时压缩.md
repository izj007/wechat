

![title](https://leanote.com/api/file/getImage?fileId=5f02dc82ab64411f370014d5)


复制 `C:\Windows\SysWOW64\notepad.exe` PE 文件至工作文件夹下，使用命令参数对 notepad.exe 进行运行时压缩。



![title](https://leanote.com/api/file/getImage?fileId=5f02e3a6ab64411f3700152b)

虽然会立刻被杀：

![title](https://leanote.com/api/file/getImage?fileId=5f02e3c3ab64411f3700152d)


可以看出压缩后文件尺寸明显变小：

![title](https://leanote.com/api/file/getImage?fileId=5f02e439ab64411d380014e9)

使用 CFF Explorer 查看 UPX 压缩后的 PE 文件：

![title](https://leanote.com/api/file/getImage?fileId=5f02e601ab64411d380014fd)

|Member|Value|
|:--:|:--:|
|Virtual Size|1c000|
|RVA|1000|
|Size of Raw Data|0|
|Pointer to Raq Data|400|

可以看到第一个节区 UPX0 的 RawDataSize 为0，即第一个节区在磁盘文件中是不存在的。那么 UPX 为何要创建这个空的节区呢？


可以看到 VirtualSize 的值为 1c000，位于第一个节区。也就是说，经过 UPX 压缩后的 PE 文件在运行瞬间将文件中的压缩的代码解压到内存中的第一个节区。

而解压缩代码与压缩的源代码都在第二个节区。文件运行时首先执行解压缩代码，把处于压缩状态的源代码解压到第一个节区。解压过程结束后即运行源文件的 EP 代码。

