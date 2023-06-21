#  【供应链安全】Bootcss CDN 疑似被投毒

[ 渗透测试网络安全 ](javascript:void\(0\);)

**渗透测试网络安全** ![]()

微信号 STCSWLAQSEC

功能介绍 号主是一名网络安全行业的资深爱好者，在这里主要分享一些安全工具，应急响应，代码审计，漏洞挖掘，安全资讯等文章与技术。
请勿利用本公众号文章内的相关所有技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

____

___发表于_

收录于合集

#钓鱼攻击 1 个

#嗅探 1 个

**0x01 现象**

使用手机 Chrome 访问某数码论坛时会自动跳转到奇怪的网站，而且是在帖子加载完成后才动态跳转。不稳定复现，仅在 Chrome 上偶现，Edge
无法复现，桌面浏览器无法复现。

主要涉及的两个恶意域名均注册于 5 月底：

![](https://gitee.com/fuli009/images/raw/master/public/20230621120620.png)

 **0x02  分析过程**

检查跳转网络请求的调用堆栈，发现一个名为`jquery.min-4.0.2.js`的可疑文件。

![](https://gitee.com/fuli009/images/raw/master/public/20230621120621.png)

检查此文件，发现它并不是 jQuery ，而是一段混淆过的恶意代码，推测是用来跳转到恶意网站。可以从原链接或者Internet Archive获取：

![](https://gitee.com/fuli009/images/raw/master/public/20230621120622.png)

这个 evil jQuery 是由托管在 cdn.bootcss.com 上的highlight.js引入的。当对此 highlight.js 的请求具有
**特定的 Referer** 和 **移动端 UA** 时，服务器才会返回带有恶意代码的 highlight.js
，否则返回正常代码，伪装性极强。而且，这段代码执行时是否引入恶意 jQuery 的操作 **也具有特定条件** ，目前我测试时使用的链接已经无法复现。

恶意的 highlight.js 可以从这里获取到，恶意代码位于文件尾部，粗略看了一下，大概是如果浏览器不是桌面端，就在 head 部分放置一个
script 标签，应该就是引入恶意 jQuery 的方法了。下图是恶意 highlight.js 与同版本正常 highlight.js 的 diff 。

![](https://gitee.com/fuli009/images/raw/master/public/20230621120624.png)

发出如下网络请求，可以获取到带有恶意代码的 highlight.js ，截至写作时依然有效：

  *   *   *   *   *   *   * 

    
    
    curl 'https://cdn.bootcss.com/highlight.js/9.7.0/highlight.min.js' \  -H 'sec-ch-ua: "Not.A/Brand";v="8", "Chromium";v="114", "Google Chrome";v="114"' \  -H 'Referer: https://bbs.letitfly.me/' \  -H 'sec-ch-ua-mobile: ?1' \  -H 'User-Agent: Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Mobile Safari/537.36' \  -H 'sec-ch-ua-platform: "Android"' \  --compressed

 **0x03  总结**

由于连接是 https 连接且证书正确，可以推测 Bootcss 服务器大概率已经被攻击者掌控。建议在项目中尽快换掉 Bootcss/Bootcdn 。

原文地址:https://www.v2ex.com/t/950163

  

>  **免责声明**  
>
>
>
> 由于传播、利用本公众号渗透测试网络安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号渗透测试网络安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！
> ****

  

 **进交流群 请添加管理员**

备注：进群，将会自动邀请您加入 渗透测试网络安全 技术 官方 交流群

![](https://gitee.com/fuli009/images/raw/master/public/20230621120625.png)

好文分享收藏赞一下最美点在看哦

  

![](https://gitee.com/fuli009/images/raw/master/public/20230621120626.png)
还在等什么？赶紧点击下方名片开始学习吧！![](https://gitee.com/fuli009/images/raw/master/public/20230621120626.png)

  

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

