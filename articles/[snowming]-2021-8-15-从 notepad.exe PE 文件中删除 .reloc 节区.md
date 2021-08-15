`.reloc` 是 PE 文件的重定位节区，对 EXE 形式的 PE 文件来说，「基址重定位表」项对运行没什么影响，将其删除后程序仍然正常运行。


# 0X01 删除 .reloc 节区头


用 CFF Explorer 打开 C:\Windows\System32\notepad.exe：


![title](https://leanote.com/api/file/getImage?fileId=5f056729ab64415f2a0013e6)



可以看到 `.reloc` 节区头从文件偏移 000002B0 开始，大小为 28（截止到 000002D4+4 BYTE）。


使用 Hex Editor 打开该区域（2B0~2D7）：

![title](https://leanote.com/api/file/getImage?fileId=5f056c0eab6441611f0013ca)


全部用 0 覆盖：

![title](https://leanote.com/api/file/getImage?fileId=5f056d2eab64415f2a001423)


# 0x02 删除 .reloc 节区


从 CFF Explorer 中可以看到，文件中 .reloc 节区的起始偏移为 0023C00。

![title](https://leanote.com/api/file/getImage?fileId=5f056d95ab64415f2a001431)


由此开始到文件末尾为 .reloc 节区。


>注：因为 .reloc 为 PE 头中的最后一个节区头，对应的节区也在文件最末尾的地方。


从 0023C00 偏移开始一直使用 Hex Editor 删除到文件末端所有数据：

![title](https://leanote.com/api/file/getImage?fileId=5f0571ceab6441611f001405)


至此，`.reloc` 节区已被物理删除。但是由于尚未修改其他 PE 头信息，文件仍无法正常运行。


# 0x03 修改 IMAGE_FILE_HEADER

在 CFF Explorer 中查看 `IMAGE_FILE_HEADER` - `Number of Sections` 项：

![title](https://leanote.com/api/file/getImage?fileId=5f0572feab6441611f001413)

本来是 6 个节区，但是因为我们已经删掉了 reloc 节区，所以要把此值改为 5。

修改 000000F6 处的值修改为 05：

![title](https://leanote.com/api/file/getImage?fileId=5f057405ab64415f2a00147b)


# 0x04 修改 IMAGE_OPTIONAL_HEADER

删除 `.reloc` 节区后，（进程虚拟内存中）整个映像就随之减少相应大小。映像大小值存储在 `IMAGE_OPTIONAL_HEADER` - `size of Image` 中，需要对其修改。


![title](https://leanote.com/api/file/getImage?fileId=5f057534ab6441611f001431)


从 PE 头中看出，当前 SizeofImage 的值为 2B000。问题在于，要减去多少才能让程序正常运行。



![title](https://leanote.com/api/file/getImage?fileId=5f05762cab64415f2a001498)

之前已经得知 `.reloc` 节区的 VirtualSize 值为 21AB，将其根据 Section Alignment 扩展后变为 3000。所以应该从 Size of Image 减去 3000 才正确。

>注：
如果要在 PE32 里面删掉某个节区，那么内存中也应该删掉对应的 VirtualSize。如果此节区对应的 VirtualSize 不对齐 Section Alignment，那么应该扩展为 Section Alignment 的整数倍。这个原因是因为节区按「页」分主要是为了满足节区的 RWX 属性的一致性。
所以节区要按 Section Alignment 的整数倍分，其余填充 NULL，主要是满足设置在节区 `IMAGE_SECTION_HEADER` 结构体数组的 characteristics 成员中设置的属性。
<br/>
但是如果 PointerToRawData 没有跟 FileAlignment 对齐（PointerToRawData 本应该是 FileAlignment 的整数倍），那么在计算 RAW 时候，PointerToRawData 应该向小取整数倍。因为这样才能包含节区起始偏移。


2B000 - 3000 = 28000

修改 00000140 处的值：

原本是 0002B000：

![title](https://leanote.com/api/file/getImage?fileId=5f057722ab64415f2a0014aa)

修改为 00028000：

![title](https://leanote.com/api/file/getImage?fileId=5f05776cab6441611f00144f)


修改后的 notepad.exe 应该能够正常运行了。


notepad 依赖 mui 资源，这样执行：

```
md zh-cn && cd zh-cn && copy C:\Windows\System32\zh-CN\notepad.exe.mui
```

>堆栈：
![title](https://leanote.com/api/file/getImage?fileId=5f059271ab6441611f001595)

# 0x05 总结：


若想要准确删除位于文件末尾的 `.reloc` 节区，需要按照以下4个步骤操作：

1. 整理 `.reloc` 节区头
2. 删除 `.reloc` 节区头
3. 修改 IMAGE_FILE_HEADER
4. 修改 IMAGE_OPTIONAL_HEADER

