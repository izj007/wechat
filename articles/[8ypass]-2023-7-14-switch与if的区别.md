#  switch与if的区别

原创 8ypass  [ 8ypass ](javascript:void\(0\);)

**8ypass** ![]()

微信号 b8ypass

功能介绍 广东人的安全圈

____

___发表于_

收录于合集

**记录一下笔记,从汇编理解switch和if的区别**

 **首先看下switch在case分支少的情况下与if汇编的区别** **  
**

  

![](https://gitee.com/fuli009/images/raw/master/public/20230714175152.png)

 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20230714175154.png)

可以看到在switch分支少的情况下，if和switch区别不大，都是通过cmp比较后然,通过比对jcc结果进行跳转。在分支少的情况下 两者效率差不多。

  

 **case数量多的情况下**

![](https://gitee.com/fuli009/images/raw/master/public/20230714175155.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230714175156.png)

可以发现明显变化了。

下面分析下  

  *   *   *   *   *   *   *   *   *   * 

    
    
    0040BA52 0F BE 45 FC          movsx       eax,byte ptr [ebp-4] //将局部变量(ebp-4)放入寄存器eax中0040BA56 89 45 B8             mov         dword ptr [ebp-48h],eax //将eax放入内存中(ebp-48h)0040BA59 8B 4D B8             mov         ecx,dword ptr [ebp-48h] //再拷贝一份到ecx0040BA5C 83 E9 41             sub         ecx,41h //当前的值-41h 既0x44-0x41h =30040BA5F 89 4D B8             mov         dword ptr [ebp-48h],ecx //将运算后的值放到ebp-48h 用于后续比较0040BA62 83 7D B8 05          cmp         dword ptr [ebp-48h],5 //这里为什么比较5呢 0040BA66 77 55                ja          $L624+0Fh (0040babd) //跳转default0040BA68 8B 55 B8             mov         edx,dword ptr [ebp-48h]0040BA6B FF 24 95 68 BB 40 00 jmp         dword ptr [edx*4+40BB68h]  
    

  

switch汇编在比较前做了一些工作,将当前用于比较的值放入eax ecx寄存器中，并对该值-41h
//3。这里为啥要-41h(41h是A),运算后的结果与5进行比较。(这里5+41h其实就是F,对应代码中最后一个case分支),接着ja根据寄存器的值判断是否需要跳转到0040babd地址(default分支),不需要的话重新将值放入寄存器
并根据switch提前分配的一张表运算后得到地址进行跳转。  

也就是这条指令  

![](https://gitee.com/fuli009/images/raw/master/public/20230714175157.png)

根据地址查看表  

![](https://gitee.com/fuli009/images/raw/master/public/20230714175158.png)

表会将switch中每一个case中代码的首地址存储到内存中,且按照case常量从小到大的顺序排列好,一个地址占用四子节  

![](https://gitee.com/fuli009/images/raw/master/public/20230714175159.png)

接着通过 edx值*4+表地址得到要jmp的地址,完成后续运算。  

  

从这里可以看出switch在case分支多的情况下与if的汇编不同，效率也比汇编高。

首先会判断case的值与最小值和最大值进行比较判断,判断是否在区间内,如果不在走default,如果在的话在根据偏移量+switch为我们生成好的表得到jmp需要跳转的地址。

  

这里可以看出switch在case分支多的情况下效率要比ifelse高。

  

  

 **  
**

 **  
**

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

