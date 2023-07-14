#  免杀工具|Golang版SigThief

[ Zer0ne安全研究 ](javascript:void\(0\);)

**Zer0ne安全研究** ![]()

微信号 JC_SecNotes

功能介绍 信安新手的成长手札

____

___发表于_

收录于合集

#golang 3 个

#项目 5 个

JSigThief

    
    
    项目地址：https://github.com/chroblert/JSigThief  
    

## 介绍

原版是python语言编写的，现在用golang写一版SigThief,  
本工具属于JBypass免杀系列的其中一个工具。

本工具实现了python版sigthief中的大部分功能，目前只支持对exe程序进行签名提权和添加…

## 使用

![](https://gitee.com/fuli009/images/raw/master/public/20230714174841.png)

### 检查某程序是否有签名

`jsigthief.exe check -i <具体文件路径>`  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174842.png)

### 导出已签名程序的数字签名

`jsigthief.exe export -i <已签名文件路径> -o <导出路径>`  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174843.png)

### 向目标文件添加导出的数字签名

`jsigthief.exe add -s <导出的.sig文件> -t <待添加签名的程序> -o <签名后输出路径>`  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174844.png)

### 直接偷取某程序的签名并添加到待签名的程序中

`jsigthief.exe -i <带有数字签名的PE文件> -t <待签名的PE文件> -o <签名后的输出路径>`  

![](https://gitee.com/fuli009/images/raw/master/public/20230714174845.png)

  

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

