#  云演 | 失业利器ChatGPT之助你逆向CTF赛题

原创 x0xKiKi  [ 小草培养创研中心 ](javascript:void\(0\);)

**小草培养创研中心** ![]()

微信号 gh_a824093cc3ce

功能介绍 数字时代网安人才培养的践行者

____

___发表于_

收录于合集

#网络安全 3 个

#云演 10 个

![](https://gitee.com/fuli009/images/raw/master/public/20230302142959.png)

ChatGPT似乎是无所不能的，它可以帮你完成很多事情，放在二进制逆向方面也不例外，本文将用ChatGPT来解决一个CTF赛事中的逆向题，让我们来看看它能帮助我们多少。

  

首先拿出一个CTF逆向程序，运行之后要求找到flag，如图1：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143001.png)

<图1>

  

通过程序的字符串来定位程序的关键代码部分，当然这部分操作需要我们人工进行，如果ChatGPT能够分析程序就好了，对吧~

我们通过步步分析的方式找到了程序获取输入之后会将内容逆序排列，例如输入”impassword”之后会变为”drowssapmi”，如图2，图3：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143002.png)

<图2>

  

![](https://gitee.com/fuli009/images/raw/master/public/20230302143003.png)

<图3>

  

接下来拿出这个关键函数的代码部分，问问ChatGPT，看看它能帮我分析一下这个函数所做的事情，由于代码过长，这里只截取部分，如图4：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143005.png)

<图4>

  

将这段话翻译下，图5：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143006.png)

<图5>

  

这个回答是有一些笼统，如果我们直接提供伪代码，它又会怎么回答。

接下来用IDA载入程序，随后生成伪代码，之后问一下ChatGPT，图6：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143007.png)

<图6>

  

可以看到ChatGPT帮助我们分析了这个函数，即便最后它似乎有一些话没有说完。但它的分析也帮助我们了解了一下，这个程序会进行异或，同时会有对比。如图7：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143009.png)

<图7>

  

这个程序存在反调试和部分代码混淆，希望在未来的某一天ChatGPT可以帮助我们解决这部分的问题。不过接下来我们要继续进行一些分析，例如找到unk_5E3248存的是什么内容。如图8：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143010.png)

<图8>

  

找到了这些数据之后，再来咨询ChatGPT，如图9：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143011.png)

<图9>

  

再问过之后，好像没有提供有价值的内容，可能是我问的姿势不对。那么我换个方法问问它，如图10：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143012.png)

<图10>

  

不得不说ChatGPT的上下文功能是真的很厉害，可以通过之前的对话内容准确的知道我需要什么，并提供了我需要输入的内容。

程序是需要进行异或的，所以接下来我们需要ChatGPT对这些数据进行异或，接着问问它相关的内容。如图11：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143013.png)

<图11>

  

它为我们提供了异或的代码，我们使用VS对其进行编译，并查看结果，如图12：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143015.png)

<图12>

  

这个异或以及代码结果是没有问题的，不过题解需要的是完整的内容，于是可以继续让ChatGPT帮助我们去修改代码，如图13：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143016.png)

<图13>

  

将ChatGPT修改后的代码进行编译，不过我所使用的VS编译器则会报错，不过是一个小错误，如图14：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143017.png)

<图14>

  

接下来我们重新要求ChatGPT进行代码的修改，如图15：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143018.png)

<图15>

  

让人再次叹服，它再次修改了代码，但我的VS依然会报错。算了还是自己手动修改一下吧。修改过了ChatGPT提供的代码之后，输出的内容离我们真正的flag是越来越近了，如图16：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143019.png)

<图16>

  

由于我们输入的时候，程序会将我们的输入内容进行逆序，所以在这里如果加上逆序输出的话就没有问题了，我们再次与ChatGPT对话修改代码，如图17：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143020.png)

<图17>

  

我认为ChatGPT所在强大之处在于，它不仅很快速的生成代码，而且代码也非常的简洁，是一个非常合格的程序员，在此之前我自己写的C语言代码则要比ChatGPT生成的代码要复杂，图18是我之前写的C代码，图19是ChatGPT生成的C++代码：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143021.png)

<图18>

  

![](https://gitee.com/fuli009/images/raw/master/public/20230302143022.png)

<图19>

  

下面我们对ChatGPT生成的代码进行编译运行，可以成功的输出了字符串内容。如图20：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143023.png)

<图20>

  

输出字符串内容之后，我们对其进行验证，将生成的字符串写入到程序中，可以看到输出的内容就是真正的flag，如图21：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143024.png)

<图21>

  

同时，也可以让ChatGPT帮助我们生成Python代码，图22：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143025.png)

<图22>

  

使用了Python进行编译运行后，也输出了正确的flag，如图23：

![](https://gitee.com/fuli009/images/raw/master/public/20230302143026.png)

<图23>

  

  

以上内容就是我们使用ChatGPT帮助我们去完成一个CTF逆向的程序。我们可以通过这个例子看出ChatGPT得到强大。

  

作为一款对话类的AI，虽然它并不能直接去分析一个程序，但我觉得这只是个时间问题，目前我们可以在网上找到很多在线分析程序运行过程的沙盒平台，在失业利器ChatGPT高速发展的趋势下，引用ChatGPT的分析和代码生成功能，势必会让我们的科技更进一步。

![](https://gitee.com/fuli009/images/raw/master/public/20230302143027.png)

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

