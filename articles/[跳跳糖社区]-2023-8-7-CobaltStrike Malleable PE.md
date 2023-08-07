#  CobaltStrike Malleable PE

原创 HaCky  [ 跳跳糖社区 ](javascript:void\(0\);)

**跳跳糖社区** ![]()

微信号 tttangsec

功能介绍
跳跳糖是一个安全社区，旨在为安全人员提供一个能让思维跳跃起来的交流平台。希望你在这里能够找到灵感，找到志同道合的人。https://tttang.com/

____

___发表于_

收录于合集

#二进制安全 31 个

#木马与病毒 4 个

******点击** **蓝** **字  / 关注我们****** ** ** ********

## 0x00 Malleable PE

Malleable PE
直译就是可拓展PE，通常来说，很多同学在做免杀的时候，会针对Loader进行免杀，并不会考虑针对beacon进行免杀，这就导致了很多杀软/EDR的内存防护能针对默认设置的beacon进行查杀。C2Profile提供了很好地操作beacon的方法，C2Profile不仅仅可以自定义beacon的通信属性(例如uri，header等等)，还可以对beacon进行操作，从而实现免杀的目的。

## 0x01 stage指标

C2Profile文件的stage块控制beacon的相关属性。如下图是官方文档提供的默认stage块。这个默认配置中，大概配置了一下几个属性，`userwx`,`complie_time`,`image_size_x86`,`image_size_x64`,`obfuscate`等等。这几个属性可以决定beacon部分PE属性和部分操作。

![]()mark

`transform-x86`或者`transform-x64`可以修改反射dll的数据。这两个块支持3个命令，`prepend`,`append`,`strrep`。`prepend`是在beacon之前添加字节数据，这样可以规避那些单纯检测内存开头部分数据的EDR。`append`是在beacon数据之后添加额外的数据，而`strrep`则是替换字符串操作，其实很多厂商会将beacon的导出函数`ReflectiveLoader`作为检测的一个点，具体可以看flu0rite的记一次cs
bypass卡巴斯基内存查杀，这篇文章详细介绍了卡巴针对beacon的内存查杀的特征，其中就有`ReflectiveLoader`。

![]()mark

除此以外呢，stage还可以将字符串添加到beacon的.rdata段。

![]()mark

  

  

下面是针对stage块的一些配置属性。

  * • checksum：Beacon的PE的检验和

  * • cleanup：释放反射dll的内存

  * • compile_time：编译时间

  * • entry_point：入口点

  * • image_size_x86(image_size_x64)：Beacon的PE的SizeOfImage(映像大小)

  * • module_x86(module_x64)：默认情况下，beacon是通过VirtualAlloc开辟内存的，如果设置module_x86属性，就可以Load module_x86指定的模块，然后将beacon复写到该模块的内存区域。

  * • name：决定beacon dll导出表的名字

  * • obfuscate：加密beacon Dll的导入表，复写未使用的header内容，以及让ReflectiveLoader将beacon拷贝到新的内存区域

  * • rich_header：rich_header是`PE`这个MagicNumber后面的pe数据

  * • sleep_mask：在sleep之前混淆beacon，避免内存查杀

  * • smartinject：Beacon 将关键函数指针（如 GetProcAddress 和 LoadLibrary）嵌入到同架构的post-ex DLL中。这使 post-ex DLL可以在新进程中进行自我引导，而不会出现类似shellcode的行为，如使用PEB寻找kernel32.dll和其中的函数指针。

  * • stomppe：在加载完beacon之后，修改Characteristic值

  * • userwx：是否使用RWX权限，RWX容易引起怀疑

  * • allocator：控制Beacon反射加载器初始化时使用的内存分配函数（VirtualAlloc，HeapAlloc，MapViewOfFile）默认使用VirtualAlloc函数

![]()mark

![]()mark

  

  

## 0x02 C2Profile加载

TeamServer负责加载C2Profile文件，IDEA中找到server/TeamServer.java，在绿色三角启动调试，注意，在debug窗口中，点击扳手按钮，以设置调试选项。依次设置VM
Optional，可以将TeamServer这个文件的设置直接复制进去，我这里设置的是`-XX:ParallelGCThreads=4
-Dcobaltstrike.server_port=50050 -Djavax.net.ssl.keyStore=./cobaltstrike.store
-Djavax.net.ssl.keyStorePassword=123456 -XX:+AggressiveHeap
-XX:+UseParallelGC`。然后添加主类，最后添加要调试的参数即可。

![]()mark

![]()mark

首先判断端口是否有效。然后加载C2Profile文件。

![]()mark![]()mark将C2Profile文件按行读取，以'\n'分割。![]()mark然后对C2Profile进行解析，首先对Profile这个结构体进行初始化，这里面存储的是一些默认的C2Profile设置。然后调用Loader.parse进行解析读取到的C2Profile文件内容，![]()mark![]()mark这就是最后解析结果，最后在`TeamServer.go()`注册监听器(Listeners)，Beacons，Phisher等组件。最后建立监听，等待GUI进行链接。然后建立客户端的认证，然后拉起线程和客户端进行socket链接。![]()mark![]()mark

  

  

## 0x03 beacon差异性分析

### 0x3.1 module_x86

首先，采用默认的C2Profile生成beacon文件，至于生成beacon的流程可以参考这篇文章。然后测试一下`module_x86`属性。修改C2Profile文件的stage块。添加`set
module_x86
"xpsservices.dll";`这个语句。module_x86这个语句的作用是加载module_x86指定的模块，然后将beacon复写到该模块的内存区域。

![]()mark

在MalleablePE.java的`pre_process`会根据c2profile的set命令获取相对于的值，然后根据值对PE进行对于的操作。

![]()mark

![]()mark在`setModuleStomp`这个方法中，首先将设置Characteristic为0x4000。然后在PE偏移为0x40的地方，添加大小为0x40的随机数据，接着在将C2Profile解析到的模块名(xpsservices.dll)写入PE偏移为0x40的地方，相当于重新覆盖直接写的随机数据的一部分。由此完成操作。![]()mark对比一下生成的修改过module_x86属性的beacon.bin,和默认C2Profile生成的beacon.bin数据。有效的差异就是在PE+0x40出存在一个模块名。![]()mark为了方便调试，生成一个beacon.exe程序，执行到beacon的时候，在PE偏移+0x40处下访问断点，以及针对LoadLibrary(Ex)W(A)下断点，程序中断在`003A82DF`处，每个断点都不一样。![]()mark![]()mark之后逻辑就很清楚了，首先加载指定的Module，然后通过函数需要，获取导出函数地址，分别将PE头和节区数据复制到目的地址。![]()mark最后解析入口点，进行跳转即可![]()mark

  

  

### 0x3.2 stomppe

通过`set stomppe "true"`启用stomppe，看官方文档的介绍，stomppe的目的是轻微混淆内存中的Dll
beacon，通过查看源码，通过了解`PEEditor.setCharacteristic`函数发现，该函数目的就是修改PE文件头中的Characteristic属性，这个属性和该文件的类型有关。用PEview查看默认生成的beacon文件，通过该属性值很显然的发现，这是一个32bit的dll可执行文件。感觉这个功能挺鸡肋的。

![]()mark

![]()mark

  

  

### 0x3.3 sleep_mask

sleep_mask的作用是在sleep之前加密beacon以规避基于内存特征的查杀。

中断在`BeaconPayload.setupGargle`方法处，首先，在setupGargle函数中，重新读取了resources/beacon.dll数据，然后将.text段的EndAddr保存在Settings的序号为41的地址处。

![]()mark

然后计算rdata段和.text段之间的间距，如果间距小于256则不行。开始我很迷惑这步的操作，在后面的分析调试过程中，才发现，这个区域存储的是加密函数，而且当时我也好奇，如果要加密几个节区的话，代码段一旦被加密，事后怎么解密呢?其实CobaltStrike的设计者非常巧妙。

![]()mark

最后获取了所有的Sections的起始地址和结束地址，并将其保存在序号为42的Settings处。

![]()mark生成并调试beacon.exe，通过之前的分析，sleep_mask作用就是在beacon
休眠之前加密beacon，所以只需要在beacon所存放的内存区域设置访问断点即可。具体可以这样操作，假设beacon分配在0x00300000处，首先，针对使程序运行到337D06处，这是程序暴力枚举到beacon区域起始地址，也就是0x00300000处。我们首先需要跳过这部分。然后在0x00300000处下访问断点，运行后断在0x003383D8处，![]()mark我们首先需要跳过这部分。然后在0x00300000处下访问断点，运行后断在0x003383D8处，显然可以看到依次将抹去了MZ的PE头和节区数据复制到新的区域。![]()mark![]()mark然后在新的beacon的地址，也就是0x00550000处下访问断点。然后运行，此时中断在0x00575533。因为经过多次调试，图片的断点已经不相符了。显然，beacon被加密之后，进行休眠。![]()mark![]()mark当休眠结束之后，将加密的beacon还原。然后通过交叉引用，将主要执行这一操作的函数定位到DllMain函数的Sub_591391_OptionalFunc的Sub_594267_SleepMask函数。![]()mark![]()mark![]()mark![]()mark在Sub_8C9728_GetValueInC2Profile函数中，首先根据C2Profile的配置信息在偏移为0x29个元素的地方，取的值，如果取到的值为空的话，说明不需要加密，则直接Sleep，否则的话，进入Sub_8C4262_EncodeBeacon。![]()mark![]()markSub_8C4262_EncodeBeacon在经过jmp后，首先，获取10004262h和10004262h的差值，因为从进程中dump出来的beacon的符号信息不完整，我们重新生成beacon.bin文件，查看这部分信息。很显然，sub_10004262和sub_100041D5函数是连续的，其本质就是为了获取sub_100041D5函数地址。![]()mark![]()mark然后通过C2Profile获取.text段末尾的偏移地址，并将100041D5h的数据复制到.text段段尾。在重新生成的beacon.bin中查看100041D5h的内容。很显然其实就是一个加密函数。整个加密逻辑和上面其实是能对应上的。![]()mark![]()mark![]()mark接着逻辑就很简单了，申请一个长度为0x10的Buffer，然后偏移为0x00的地方存储BaseAddress，在+0x04的地方存储获取的那些节区的地址，最后在偏移为0x08的地方保存生成的Key。将加密函数EncodeFunc和保存上述信息的结构体放到两个全局变量中，就可以在函数外面使用了。![]()mark![]()mark将EncodeFunc下硬件执行断点，断下。整体逻辑大概就是这样的，首先去DataStruct的地址，这是一对地址(起始地址和结束地址)，分别取了地址之后，进行判空，以及起始地址要小于结束地址，然后去BaseAddress，因为DataStruct存储的是偏移地址，加上基地址才是各个区段的绝对地址。然后进行加密运算。当加密完成之后，执行Sleep操作，休眠完，在解密相关数据段即可。![]()mark

  

  

总结一下，CobaltStrike通过获取Section的起始地址和结束地址，并获取.text的结束地址，并将加密函数复制到.text的结束地址到rdata段之间空的地方，因为这个地方不会被加密。该加密函数内部会调用Sleep进行休眠，在休眠之前，加密数据，在休眠之后解密数据，以此往复，实现Sleep_Mask的功能。不得不说，设计的非常巧妙。

## 参考文献

  * • [记一次cs bypass卡巴斯基内存查杀] https://xz.aliyun.com/t/9224?page=1

  * • https://wbglil.gitbook.io/cobalt-strike/cobalt-strikekuo-zhan/malleable-c2#malleable-pe-process-injection-and-post-exploitationbeacon-hang-wei

  * • https://github.com/rsmudge/Malleable-C2-Profiles/blob/master/normal/reference.profile#L254-L255

  * • CobaltStrike4.0用户手册_中文翻译

 **推荐阅读：**[ Shiro
历史漏洞分析](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247489093&idx=1&sn=a152d824764fcb8736affbd3ef8b9e5e&chksm=c17107d4f6068ec247834014d3d1b8d30cdad5bbc9f01be1a7f433186f6fdfbf2b86885f4cf0&scene=21#wechat_redirect)[浅谈pyd文件逆向](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247488872&idx=1&sn=6f9049e42e19ff4a04e172750c62e479&chksm=c17104f9f6068deff8184a99c1226d7cf390bdbb87d32e2f8dc4ff05dc5208ad28e19f872a8e&scene=21#wechat_redirect)[CVE-2022-23222漏洞及利用分析](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247488478&idx=1&sn=5eec4a19c6d838dd1dcdc76bd56cd91c&chksm=c171024ff6068b59177daed4601bc84714d2cff52a712ef243fafae08edf42c28af8c261d0a2&scene=21#wechat_redirect)  
[Mimikatz详细使用总结](http://mp.weixin.qq.com/s?__biz=MzkxNDMxMTQyMg==&mid=2247488474&idx=1&sn=3b943fbe21016467df3558278029023d&chksm=c171024bf6068b5d32ac4d4e4f4472b16519a099b3bb3d14dcd6a63c78eb424bfc2097b5c929&scene=21#wechat_redirect)

  

* * *

跳跳糖是一个安全社区，旨在为安全人员提供一个能让思维跳跃起来的交流平台。

  

跳跳糖持续向广大安全从业者征集高质量技术文章，可以是漏洞分析，事件分析，渗透技巧，安全工具等等。通过审核且发布将予以500RMB-1000RMB不等的奖励，具体文章要求可以查看“投稿须知”。阅读更多原创技术文章，戳“阅读全文”

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

