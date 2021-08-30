#  shiro反序列化绕WAF之未知HTTP请求方法

原创 c0ny1 [ 回忆飘如雪 ](javascript:void\(0\);)

**回忆飘如雪** ![]()

微信号 gv7_me

功能介绍 记录工作生活所思所想

____

__

收录于话题

#绕WAF ,2

#shiro ,1

  

**  0x01 **

 **背景**

  

当下WAF对shiro的防护，确实比较严格。对rememberMe的长度进行限制，甚至解密payload检查反序列化class。本周我遇到一个场景，就是这种情况。使用之前的方法rememberMe=加密payload+==垃圾数据也失败了，[这个方法](https://mp.weixin.qq.com/s?__biz=MzkyMDIxMjE5MA==&mid=2247484124&idx=1&sn=c258b2b5ee70b70b5464f47a975edc52&scene=21#wechat_redirect)之前有大佬分享过，我就不再赘述了。我最终使用未知HTTP请求方法解决战斗。

![](https://gitee.com/fuli009/images/raw/master/public/20210830120625.png)

  

  **0x02  **

 **过程**

  

当时我的思考是shiro的payload在header上，如何修改request header可以导致waf解析不出来，但是后端中间件正常解析呢？

  

第一步，先构造出先绕WAF，哪怕改成不合法的数据包。

第二步，在绕WAF的数据包基础上修正，让后端中间件可以解析。

  

我把被拦截的包发送的repeater模块,尝试切换http版本，添加垃圾header头等等方法均没绕过。在修改GET方法为XXX这样的未知HTTP请求方法时,发现WAF不在拦截，但是后端报错了。

![](https://gitee.com/fuli009/images/raw/master/public/20210830120627.png)

接下来验证下后端是否真正处理了rememberMe。我先请求去掉rememberMe，response对应的rememberMe消失了

![](https://gitee.com/fuli009/images/raw/master/public/20210830120629.png)  

然后再加上rememberMe,repseone的remeberMe又回来了。这说明后端正常处理rememberMe，这么绕WAF没问题！

![](https://gitee.com/fuli009/images/raw/master/public/20210830120632.png)

最后将之前注入内存webshell的payload修改下请求方法，成功拿下Web权限。

  

 **  0x03 **

 **原理**

  

方法简单粗暴，不难推断WAF是通过正常的http方法识别HTTP数据包的。但是为何后端中间件依然能拿到rememberMe的结果呢？

  

于是我在本地代码org.apache.shiro.web.mgt.CookieRememberMeManager#getRememberedSerializedIdentity处下了断点。

![](https://gitee.com/fuli009/images/raw/master/public/20210830120633.png)

通过XXX方法发送数据包，调试发现request.getCookies可以获取到rememberMe值，而且如下方法均可正常使用。说明未知HTTP请求方法不影响各类参数的读取。

![](https://gitee.com/fuli009/images/raw/master/public/20210830120636.png)

那对三大组件的调用是否有影响呢？继续翻阅Tomcat源码，我发现Listener被调用是受行为事件影响，Filter是受请求路径影响，而Servlet是受请求路径和HTTP请求方法影响。一旦遇到未知方法，Servlet不再进入业务代码，直接返回一个http.method_not_implemented报错。具体代码如下：

![](https://gitee.com/fuli009/images/raw/master/public/20210830120637.png)

所以得到一个结论就是 **未知Http方法名绕WAF这个姿势，可以使用在Filter和Listener层出现的漏洞，同时WAF不解析的情况** 。  

![]()

c0ny1

如有帮助，打赏c0ny1一瓶1664吧!

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

如有帮助，打赏c0ny1一瓶1664吧!

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

shiro反序列化绕WAF之未知HTTP请求方法

最多200字，当前共字

__

发送中

写下你的留言

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

