#  Frida开启PC端小程序调试模式

原创 A20  [ Lambda小队 ](javascript:void\(0\);)

**Lambda小队** ![]()

微信号 LambdaTeam

功能介绍 一群热爱网络安全的热血青年，一支做好事不留名的Lambda小队

____

___发表于_

收录于合集

#SRC挖掘 3 个

#小程序 1 个

_**免责声明：**_

 __
_本公众号致力于安全研究和红队攻防技术分享等内容，本文中所有涉及的内容均不针对任何厂商或个人，同时由于传播、利用本公众号所发布的技术或工具造成的任何直接或者间接的后果及损失，均由使用者本人承担。请遵守中华人民共和国相关法律法规，切勿利用本公众号发布的技术或工具从事违法犯罪活动。最后，文中提及的图文若无意间导致了侵权问题，请在公众号后台私信联系作者，进行删除操作。_

  
  
 **0x00  前言**

最近网上公开了用frida动态修改内存，开启Windows
PC端小程序调试模式的方法，该方法支持`RadiumWMPF`为8447版本的运行环境（截至发文时间是最新版本）
![]()为了方便学习核心代码的作用，本文对代码进行了一定的省略和修改

 **0x01 开启调试模式**

以下frida代码HOOK了加载小程序的函数（函数地址`0x1B3FF48`），并且从内存中搜索`"enable_vconsole":false`，替换成`"enable_vconsole":
true`

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //HOOK 小程序加载//"LaunchAppletBegin": "0x1B3FF48"Interceptor.attach(address.LaunchAppletBegin, {    onEnter(args) {        send("HOOK到小程序加载! " + readStdString(args[1]))        //搜索内存中的字符串并进行替换        for (var i = 0; i < 0x1000; i+=8) {            try {                //读取数据                var s = readStdString(args[2].add(i))                var s1 = s.replaceAll('"enable_vconsole":false', '"enable_vconsole": true')                if (s !== s1) {                    //写入数据                    writeStdString(args[2].add(i), s1)                }            } catch (a) {            }        }    }})

我们知道PC端老版本的小程序，可以通过命令行参数`--enable-
vconsole`打开调试模式，后续高版本中这个参数已经不可使用了，现在看来这个参数隐藏在了内存当中，接下来根据代码的思想，内存中搜索`enable_vconsole`，手工开启调试模式

 **0x02  手工开启调试模式**

打开Cheat Engine，选择打开的小程序

![]()

“数组类型”选择`字符串`，文本输入`enable_vconsole`，点击“首次扫描”

![]()

搜索到了一处结果，右键“浏览相关内存区域”

![]()

可以看到`enable_vconsole`被设置为了`false`

![]()

我们直接修改内存把`false`修改为`true`，注意`false`比`true`多1个字节，所以在`true`前面加个空格符

![]()

回到小程序，点击右上角`···`，再点击重新进入小程序

![]()

出现`devTools`了！赶紧打开看看

![]()

发现是个阉割版的`devTools`，无法查看网络和代码

## ![]()##

 **0x03  开启完整版Devtools**

frida代码中HOOK了`0x2EC9FBD`处，并且把`rdx`寄存器赋值成`0x7C0D6BD`

  *   *   *   *   *   *   *   *   * 

    
    
    //HOOK F12配置 替换原本内容//"WechatAppHtml":"0x2EC9FBD"//"WechatWebHtml":"0x7C0D6BD"Interceptor.attach(address.WechatAppHtml, {    onEnter(args) {        this.context.rdx = address.WechatWebHtml;        send("已还原完整F12")    }})

使用IDA查看这两处地址都是什么，`0x7C0D6BD`指向了一个完整版调试页面URL（`https://applet-
debug.com/devtools/wechat_web.html`）

![]()

`0x2EC9FBD`上面的`0x2EC9FB6`处把阉割版的调试页面URL（`https://applet-
debug.com/devtools/wechat_app.html`）赋值给了`rdx`

![]()

查看伪代码，可以看出该函数是通过参数`a3`判断是把完整版还是阉割版的调试模式赋值给变量`v8`

![]()

因此`this.context.rdx = address.WechatWebHtml`就是在这个判断之后，把完整版的赋值给变量`v8`

最终效果如下：

![]()![]()##

##

**0x04  题外话**

原版代码中使用的是nodejs binding，在安装依赖模块时报错提示要安装Visual Studio C++，这玩意好几G

![]()简单分析了原作者写的启动代码，发现只是为了获取小程序的pid传给frida，我照猫画虎使用python的win32库获取pid也能实现一样的效果，还免去了安装VS，这里就不再展开。公开代码https://github.com/x0tools/WeChatOpenDevTools

 **0x05  加入我们**

后台回复“加群”或“小助手”，或扫描下方二维码加入我们的付费圈子，一起进步吧

  

![]()

![]()

##

  

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

