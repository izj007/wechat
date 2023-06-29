#  NginxWebUI runCmd远程命令执行漏洞 附检测POC

原创 unknown  [ 极致攻防实验室 ](javascript:void\(0\);)

**极致攻防实验室** ![]()

微信号 jz_sec

功能介绍 极致攻防实验室专注于最前沿，最基础，最实际的红蓝对抗黑客技术。

____

___发表于_

收录于合集

#nuclei 10 个

#poc 10 个

#漏洞 10 个

**免责声明：**
本文章或工具仅供安全研究使用，请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，极致攻防实验室及文章作者不为此承担任何责任。  
 **简介：**

NginxWebUI 是一款图形化管理 nginx 配置的工具，可以使用网页来快速配置 nginx单机与集群的各项功能，包括
http协议转发，tcp协议转发，反向代理，负载均衡，静态 html服务器，ssl证书自动申请、续签、配置等，配置好后可一建生成
nginx.conf文件，同时可控制 nginx使用此文件进行启动与重载，完成对 nginx的图形化控制闭环。

  
 **利用条件：** NginxWebUI <= 3.5.0  
 **漏洞原理：**
NginxWebUI未对后台功能做有效的身份认证与用户的输入进行安全过滤，导致在权限绕过后可直接访问后台执行任意命令，最终可以达到无条件远程命令执行的效果。  
 **漏洞检测：**  

![](https://gitee.com/fuli009/images/raw/master/public/20230629083212.png)

nuclei检测：

  * 

    
    
    ./nuclei -u http://www.xxx.com/ -t ./ultimaste-nuclei-templates/nginxwebui/nginxwebui-runcmd-rce.yaml

‍

![](https://gitee.com/fuli009/images/raw/master/public/20230629083214.png)

nuclei-templates:https://github.com/UltimateSec/ultimaste-nuclei-
templates/blob/main/nginxwebui/nginxwebui-runcmd-rce.yaml  
  
 **修复方式：**

  * 收敛系统访问网络
  * 安装官方对应版本的更新补丁  

  
 **参考：**

  * https://mp.weixin.qq.com/s?__biz=MzIwMDk1MjMyMg==&mid=2247491574&idx=1&sn=043729e27577a851856c002cfb0409b8

  

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

