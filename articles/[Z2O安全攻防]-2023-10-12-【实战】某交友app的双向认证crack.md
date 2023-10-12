#  【实战】某交友app的双向认证crack

Ch3nYe  [ Z2O安全攻防 ](javascript:void\(0\);)

**Z2O安全攻防** ![]()

微信号 Z2O_SEC

功能介绍 From zero to one

____

___发表于_

收录于合集

#审计 3 个

#实战 14 个

#APP渗透 2 个

点击上方[蓝字]，关注我们

 **建议大家把公众号“Z2O安全攻防”设为星标，否则可能就看不到啦！**
因为公众号现在只对常读和星标的公众号才能展示大图推送。操作方法：点击右上角的【...】，然后点击【设为星标】即可。

![]()

# 免责声明

本文仅用于技术讨论与学习，利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者及本公众号团队不为此承担任何责任。

# 文章正文

## 0x0 前言

前几天刚总结了Android平台HTTPS的证书绑定和绕过方法（[Android HTTPS认证的N种方式和对抗方法总结](Android
HTTPS认证的N种方式和对抗方法总结
(ch3nye.top))），这就遇到了一个双向认证的APP，crack完了才发现2018年就有人搞过它了，不过当前版本和之前的代码逻辑有点变化，这里就简要记录一下思路。

 **环境** ：

RedmiK305G Android10 root with magisk

BurpSuite v2.0.11

frida 14.2.16

jadx v1.2.0

APP 版本v3.80.0

## 0x1 检查证书校验方式

手机添加代理证书到系统证书目录，抓包，抓到请求报文，响应总是400：

![]()

说明可能是做了双向认证。

查一下assets目录，发现client.p12证书，和我写的双向认证demo app中用的证书名字都一样，赶紧到反编译代码里搜一波：

![]()

到这里看一下代码逻辑就实锤了，就是双向认证。

要想抓包必须将客户端证书client.p12导入BurpSuite，所以要获取该客户端证书的密码。

## 0x2 获取客户端证书密码

从理论上来说，APP要使用客户端证书client.p12，就必须读取该证书文件，这个过程中需要输入证书密码，所以必然能在本地逆向出证书密码。

### a.分析代码

去看看上述搜索到的代码逻辑，jadx对此处代码反编译不全，需要看smali，证书载入的逻辑在构造函数init中，代码片段：

    
    
    const-string v5, "client.p12"  
      
    invoke-virtual {v0, v5}, Landroid/content/res/AssetManager;->open(Ljava/lang/String;)Ljava/io/InputStream;  
      
    move-result-object v0  
    :try_end_50  
    .catch Ljava/lang/Exception; {:try_start_3c .. :try_end_50} :catch_ee  
      
    .line 10  
    :try_start_50  
    invoke-virtual {v1}, Ljava/lang/String;->toCharArray()[C  
      
    move-result-object v5  
      
    invoke-virtual {v4, v0, v5}, Ljava/security/KeyStore;->load(Ljava/io/InputStream;[C)V // 此处参数v5字符数组就是密钥  
    :try_end_57  
    .catch Ljava/lang/Exception; {:try_start_50 .. :try_end_57} :catch_57  
    .catchall {:try_start_50 .. :try_end_57} :catchall_5b  
      
    .line 11  
    :catch_57  
    :try_start_57  
    invoke-virtual {v0}, Ljava/io/InputStream;->close()V  
    :try_end_5a  
    .catch Ljava/lang/Exception; {:try_start_57 .. :try_end_5a} :catch_60

分析一下上述代码，密钥v5是从v1字符串转CharArray来的，向上翻找v1：

    
    
    invoke-virtual {v1, v2}, Lcn/xxxxapp/android/net/XxxxNetworkSDK;->b(Ljava/lang/String;)Ljava/lang/String;  
    move-result-object v1

发现v1是从cn.xxxxapp.android.net.xxxxNetworkSDK中的方法b(String str)得到的。

再去翻看cn.xxxxapp.android.net.xxxxNetworkSDK.b(String str)：

    
    
    public String b(String str) {  
        return XxxxPowerful.a(str);  
    }

再去翻看cn.xxxxapp.android.xxxxpower.xxxxPowerful.a(String str)：

    
    
    public class xxxxPowerful {  
        static {  
            System.loadLibrary("xxxxpower");  
        }  
      
        public static String a(String str) {  
            return c(str);  
        }  
      
        public static String b() {  
            return i();  
        }  
      
        public static native String c(String str);  
        ...  
    }

现在了然了，从libxxxxpower.so中获取client.p12证书密码。

### b.hook获取

    
    
    frida -U -f cn.xxxxapp.android -l .\hook_Key.js --no-pause

hook_Key.js

    
    
    setTimeout(function(){  
        Java.perform(function (){  
            var KeyStore = Java.use("java.security.KeyStore");  
            KeyStore.load.overload('java.io.InputStream', '[C').implementation = function(a,pass) {  
                console.log("\n============keystore.load()============");  
                console.log(pass);  
                return this.load(a,pass);  
            };  
            var XxxxNetworkSDK = Java.use("cn.xxxxapp.android.net.XxxxNetworkSDK");  
            XxxxNetworkSDK.b.overload('java.lang.String').implementation = function(str) {  
                var res = this.b(str);  
                console.log("\n============XxxxNetworkSDK.b()============");  
                console.log(str);  
                console.log(res);  
                return str;  
            }  
      
        });  
    },0);

我一开始想直接hook java.security.KeyStore.load方法获取key但是结果总为空，后来通过hook
XxxxNetworkSDK中b方法成功获取密钥：

frida logcat：

    
    
    [Redmi K30 5G::cn.xxxxapp.android]->  
    ============keystore.load()============  
    null  
      
    ============keystore.load()============  
    null  
      
    ============XxxxNetworkSDK.b()============  
    10000003  
    }%3R-\0SsjpP1w%X  
      
    ============keystore.load()============  
    [object Object]

### c.导入证书

![]()

### d.成功抓包

![]()

而且奇怪的是这个app似乎对请求没有限速，5线程疯狂访问某个用户主页1000次，居然没ban我😁🙏。

  *   * 

    
    
    https://ch3nye.top/%E3%80%90%E5%AE%9E%E6%88%98%E3%80%91%E6%9F%90%E4%BA%A4%E5%8F%8Bapp%E7%9A%84%E5%8F%8C%E5%90%91%E8%AE%A4%E8%AF%81crack/author:Ch3nYe

 ****

  

# 技术交流

### 知识星球

致力于红蓝对抗，实战攻防，星球不定时更新内外网攻防渗透技巧，以及最新学习研究成果等。常态化更新最新安全动态。专题更新奇技淫巧小Tips及实战案例。

涉及方向包括Web渗透、免杀绕过、内网攻防、代码审计、应急响应、云安全。星球中已发布  **550+** ** **
安全资源，不定时更新未公开或者小范围公开的漏洞，并为星友提供了教程、工具、POC&EXP以及各种学习笔记等等。([点我了解详情](http://mp.weixin.qq.com/s?__biz=Mzg2ODYxMzY3OQ==&mid=2247502558&idx=1&sn=6510627a91776bced518fa5199f2e5af&chksm=ceab219ef9dca88849f8e230cf17caddc5c59b4ee3dd2e55950ee091d7c2a5ce107151709544&scene=21#wechat_redirect))

 **（先到先得）**

![]()

![]()

### 交流群

关注公众号回复“ **加群** ”，添加Z2OBot好友，自动拉你加入 **Z2O安全攻防交流群(微信群)** 分享更多好东西。
**（QQ群可直接扫码添加）**

![]()

![]()

### 关注我们

 **关注福利：**

 **回复“** **app** **" 获取   app渗透和app抓包教程**

 **回复“** **渗透字典** **" 获取 针对一些字典重新划分处理，收集了几个密码管理字典生成器用来扩展更多字典的仓库。**

 **回复“** **书籍** **"  获取 网络安全相关经典书籍电子版pdf**

 ** **回复“ 资料** **"  获取 网络安全、渗透测试相关资料文档****

 ** **  
****

点个【 在看 】，你最好看

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

