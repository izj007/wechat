#  「更新」Visual-Studio-BOF-template

[ 黑客在思考 ](javascript:void\(0\);)

**黑客在思考** ![]()

微信号 hackthink

功能介绍 Red Team / Offensive Security

____

___发表于_

收录于合集

     BOF这个东西无论是使用效果还是目前的生态，都非常的好，唯一不方便的就是开发起来不得劲，主要有编写、调试这两个方面，其实主要就是编写起来费事，需要确保用到的Win32 API都定义好，并且十分冗长。

    所以为了写起来方便点，之前参考 

https://github.com/securifybv/Visual-Studio-BOF-template

这个项目以及把TrustedSec 定义的很多常用Win32API声明

https://github.com/trustedsec/CS-Situational-Awareness-
BOF/blob/master/src/common/bofdefs.h

把他俩结合起来发布了一个Visual Studio的BOF模板：

https://github.com/evilashz/Visual-Studio-BOF-template/tree/oldversion

但确实还是需要定义很多冗长的声明。

### DFR

    前段时间看到CS官方这篇文章：https://www.cobaltstrike.com/blog/simplifying-bof-development，解决了这问题，今天找时间将里面提到的解决方法再次结合到Visual-Studio-BOF-template里面。（没有Kit的大JB小子，呜呜呜）

他们用预处理器宏定义来解决了冗长的声明

    
    
    #define DFR_LOCAL(module, function) \  
    DECLSPEC_IMPORT decltype(function) module##$##function; \  
    decltype(module##$##function) *##function = module##$##function; 解释一下  
    

  1. `DECLSPEC_IMPORT decltype(function) module##$##function;`
    * `DECLSPEC_IMPORT` 是一个常见的宏，用于声明一个函数或变量是在另一个模块中定义的，这通常用于 Windows 的 DLL 导入。在这里，它声明了一个名为 `module$function` 的外部函数，其类型为 `function` 的类型（用 `decltype` 获取）。
  2. `decltype(module##$##function) *function = module##$##function;`
    * 这行代码首先定义了一个指针变量 `*function`，其类型为 `module$function` 的类型。然后将这个指针变量初始化为 `module$function` 的地址。

    大体来说现在可以比较方便的使用c++来编写BOF、并且可以调试、Beacon API我只实现了一个BeaconPrintf，剩下的可以自己到beacon.cpp写一下实现，另外调试时候的参数打包提取我没有实现。

    使用的时候只需要导入这个模板（导入方式看之前的文章：[使用 Visual Studio 开发 CS 的 BOF](http://mp.weixin.qq.com/s?__biz=MzI5NzU0MTc5Mg==&mid=2247484673&idx=1&sn=d1628ed1f77c638ba414185bfeabf8ee&chksm=ecb2cccedbc545d89bd098632be7fb23c273e585a4f2167089b39d483c42deac13b6078ba3b5&scene=21#wechat_redirect)），将需要的API这样进行导入即可。

![]()

调试的时候这么切换即可：

![]()

  

去这找改完的模板：

https://github.com/evilashz/Visual-Studio-BOF-template/tree/main

### 彩蛋

BOF + VEH = ？

![]()

### 最后

星球3日后根据多方位因素价格上调10元，到￥229，望周知。

![]()

  

  

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

