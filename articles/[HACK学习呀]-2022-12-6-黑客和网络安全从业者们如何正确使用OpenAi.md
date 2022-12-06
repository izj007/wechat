#  黑客和网络安全从业者们如何正确使用OpenAi

原创 HACK学习  [ HACK学习呀 ](javascript:void\(0\);)

**HACK学习呀** ![]()

微信号 Hacker1961X

功能介绍
HACK学习，专注于互联网安全与黑客精神；渗透测试，社会工程学，Python黑客编程，资源分享，Web渗透培训，电脑技巧，渗透技巧等，为广大网络安全爱好者一个交流分享学习的平台！

____

___发表于_

收录于合集

#黑客 12 个

#OpenAi 1 个

#渗透测试 103 个

**0X00 如何注册**

##

##  **准备工作**

  *   *   * 

    
    
     1.代理要求韩国，日本，印度，新加坡均可。香港的不行。2.准备接码平台，sms-activate.org3.准备一个浏览器

  1.  **接码**

  

###  第一步 准备接码，打开接码平台 sms-activate.org，注册一个账号

![](https://gitee.com/fuli009/images/raw/master/public/20221206143001.png)

注册后选择充值，可以选择支付宝充值，接码OpenAi的一次费用是大概11卢布，人民币来看差不多是1块钱，就先充个1美金吧

![](https://gitee.com/fuli009/images/raw/master/public/20221206143018.png)

 **2.注册OpenAi账号**

如果挂了代理，还遇到这个页面，注意看你的代理地区

![](https://gitee.com/fuli009/images/raw/master/public/20221206143020.png)

####

#### 解决挂代理后还有地区问题

首先，你要把你的代理切换到不是香港的地区，我这里选韩国。

然后，先复制下面这段代码

  * 

    
    
    window.localStorage.removeItem(Object.keys(window.localStorage).find(i=>i.startsWith('@@auth0spajs')))

 **注意，这里一定要输入javascript:，然后粘贴上面的代码，如下图所示**

![](https://gitee.com/fuli009/images/raw/master/public/20221206143021.png)

首先是打开ChatGPT的账户注册页面。谷歌注册或者邮箱注册都可以，无所谓，这里用邮箱注册作为例子。

 **https://beta.openai.com/signup**

![](https://gitee.com/fuli009/images/raw/master/public/20221206143022.png)

我是直接选择Google账号登录的，然后下一步

![](https://gitee.com/fuli009/images/raw/master/public/20221206143023.png)

  

再下一步会让你接收验证码，你选择地区为印度+91  

![](https://gitee.com/fuli009/images/raw/master/public/20221206143024.png)

然后打开解码平台，选择openai

![](https://gitee.com/fuli009/images/raw/master/public/20221206143026.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143028.png)

然后复制这个91880.....的号码，点击接收验证码

![](https://gitee.com/fuli009/images/raw/master/public/20221206143024.png)

等一会解码网站会提示验证码，我们复制粘贴验证码即可

填写完验证码后，就会到达最后一步，下面这个页面随便选一项即可  

![](https://gitee.com/fuli009/images/raw/master/public/20221206143030.png)

至此，就注册完成了  

 **0X01  Chrome插件**

 **下载地址：**

  * 

    
    
     https://github.com/wong2/chat-gpt-google-extension

![](https://gitee.com/fuli009/images/raw/master/public/20221206143031.png)

点击下载，然后在Chrome浏览器里面选择拓展程序，加载已解压的拓展程序

![](https://gitee.com/fuli009/images/raw/master/public/20221206143033.png)

安装后效果，搜索的时候，右边会出现一个ChatGPT  

![](https://gitee.com/fuli009/images/raw/master/public/20221206143034.png)

 **0X02 如何使用**

注册完成后，点击登录  

 **https://chat.openai.com/auth/login**

![](https://gitee.com/fuli009/images/raw/master/public/20221206143036.png)

 **示例：**

![](https://gitee.com/fuli009/images/raw/master/public/20221206143037.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143038.png)

需要使用英文去输入，然后对话也是英文的，记得右键翻译即可  

对于咒语的选择，可以使用deepl.com，将你的需求翻译成英文，然后再去输入

![](https://gitee.com/fuli009/images/raw/master/public/20221206143039.png)

 **0X03如何利用OpenAI提供安全从业者的工作效率**

 **Tips：将你的需求翻译成英文去和OpenAi对话即可，善用关键词和Deepl**

 **  
**1.帮我写提高工作效率的小脚本(原文是英文，我右键翻译了)  

![](https://gitee.com/fuli009/images/raw/master/public/20221206143040.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143043.png)

复制出来的脚本如下，亲测可用

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # Open the input file containing the domain nameswith open('domains.txt') as input_file:  # Read the lines from the input file  lines = input_file.readlines()  
      # Open the output files for HTTP and HTTPS domain names  with open('http_domains.txt', 'w') as http_file, open('https_domains.txt', 'w') as https_file:    # Loop through the lines in the input file    for line in lines:      # Check if the line starts with 'http://' or 'https://'      if line.startswith('http://'):        # Write the line to the HTTP output file        http_file.write(line)      elif line.startswith('https://'):        # Write the line to the HTTPS output file        https_file.write(line)

2.让Openai给我写一个shellcode加载器，并不断完善，不断PUA OpenAi，哈哈哈  

![](https://gitee.com/fuli009/images/raw/master/public/20221206143044.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143045.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143046.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143048.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143049.png)

3.写一份公司成立信息安全委员会的文件和章程

![](https://gitee.com/fuli009/images/raw/master/public/20221206143050.png)

4.如果让OpenAi来面试安全岗位，问他会怎么样呢(下面的图来源朋友圈)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143051.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143052.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143053.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143054.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20221206143055.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143056.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143057.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143058.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143059.png)

5.信息安全工程师年度述职PPT

![](https://gitee.com/fuli009/images/raw/master/public/20221206143100.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143102.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143106.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221206143108.png)

**![](https://gitee.com/fuli009/images/raw/master/public/20221206143109.png)**

  

 **推荐阅读：**

 **  
**

 **[实战 |
记一次SSRF攻击内网的实战案例](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247504980&idx=1&sn=d8ee8cc63ce8bb937891c0942c59d2e0&chksm=ec1c816bdb6b087d4b4315f81c87c6a8b8a7a9cfe60f42f79bbb9a14877016b6b607f95d962e&scene=21#wechat_redirect)  
**

  

 **[实战 |
记两次应急响应小记](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247508721&idx=1&sn=6bd3d0e1354da8b22cb12cd42587c3d8&chksm=ec1cf7cedb6b7ed8a701c0f460539e5039ea6e35e22c87cd34a6b738651c5d6a9040048faa66&scene=21#wechat_redirect)**
**  
**

  

 **[干货 |
Wordpress网站渗透方法指南](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247508079&idx=1&sn=668535e1e2e29683403cf6dea91d2ad6&chksm=ec1cf550db6b7c46859ff1ab8129eaf46f82a9c9e427be6a3a94ffd9827a0873bf5afc8d209f&scene=21#wechat_redirect)  
**

  

[ **实战 |
记一次CTF题引发的0day挖掘**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247509402&idx=1&sn=af99609cb6685118b96616a8011ac252&chksm=ec1cf0a5db6b79b38ea240c755ccf10b94f3b0c19326297c569ab69abba57e7e8f8093063e66&scene=21#wechat_redirect)  

  

[ **2022年零基础+进阶系统化白帽黑客学习 |
全新版本**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247509409&idx=1&sn=33fe4e4fe791d8d2fa99dc784f47dad6&chksm=ec1cf09edb6b79889157a342133a9f517602787be0033995cd1fe163f660ab97062e45126f63&scene=21#wechat_redirect)  

  

[ **实战 |
记一次邮件系统C段引发的SQL手注和内网渗透**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247509307&idx=1&sn=3b09b33609b5fb4476dae0e84e489d4a&chksm=ec1cf004db6b79127cc7b5fb1a3606054440feb48208a60a759f29ffff0f0659f19826b9ad48&scene=21#wechat_redirect)  

  

 **点赞，转发，在看**

  

![](https://gitee.com/fuli009/images/raw/master/public/20221206143110.png)

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

