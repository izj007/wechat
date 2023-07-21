#  浅析Smartbi逻辑漏洞

原创 Y4tacker  [ WgpSec狼组安全团队 ](javascript:void\(0\);)

**WgpSec狼组安全团队** ![]()

微信号 wgpsec

功能介绍 WgpSec
狼组安全团队由几位热爱网络安全的年轻人一同组成过去的几年内没来得及让团队发生有效且质的变化这一次，为了我们的slogan：打造信息安全乌托邦。前进！

____

___发表于_

收录于合集

#安全研究 29 个

#漏洞复现 14 个

**点击蓝字**

![]()

 **关注我们**

  

  

 ** _声明  
_**

本文作者：Y4tacker

本文字数：4000字

阅读时长：约10分钟

附件/链接：点击查看原文下载

 **本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  
  
  

## 写在前面

仅分享逻辑漏洞部分思路，全文以无害路由做演示，后续利用部分打码处理

厂商已发布补丁：https://www.smartbi.com.cn/patchinfo

## 分析

最近可以看到smartbi官网突然发布了新的补丁，对比学习了下

利用也仍然和RMIServlet相关，在这个Servlet上还有个Filter(smartbi.freequery.filter.CheckIsLoggedFilter)

如果我们访问调用一些未授权的类方法，就会返回如下字段

![]()image-20230705095525401

我们先来看看如果正常情况程序该怎么走，首先如果调用RMIServlet，则会尝试获取到className与methodName，获取的方式也多种多样

![]()image-20230705095800089

有通过解码windowUnloading参数获取

![]()image-20230705100021973

有通过GET/POST获取

![]()image-20230705100158411

甚至支持从请求体流中解析

![]()image-20230705100240104

后面通过下面这两个判断对类与方法做鉴权，如果为true则会继续判断是否登录，未登录则抛出CLIENT_USER_NOT_LOGIN

![]()image-20230705100351950

这里对于未授权右边部分我们可以不必关心，按照运算符优先级只要`FilterUtil.needToCheck`返回false那么整个结果一定为false，而`FilterUtil.needToCheck`中返回false的都是白名单，代表我们不需要登录都能访问，这也就是为什么上个版本通过利用`loginFromDB`登录默认内置用户

![]()image-20230705101505359

接下来过了过滤器部分，我们在看RMIServlet如何取值，通过parseRMIInfo从request当中取得

![]()image-20230705101708783

在这个方法中首先通过request.getParameter取值，若为空则通过multipart获取参数，如果都不行则通过request.getAttribute从之前保存的属性当中获取

    
    
    public static RMIInfo parseRMIInfo(HttpServletRequest request, boolean forceParse) {  
            if (!"/vision/RMIServlet".equals(request.getServletPath()) && !forceParse) {  
                return null;  
            } else {  
                RMIInfo info = getRMIInfoFromRequest(request);  
                if (info != null) {  
                    return info;  
                } else {  
                    String className = request.getParameter("className");  
                    String methodName = request.getParameter("methodName");  
                    String params = request.getParameter("params");  
                    if (StringUtil.isNullOrEmpty(className) && StringUtil.isNullOrEmpty(methodName) && StringUtil.isNullOrEmpty(params) && request.getContentType() != null && request.getContentType().startsWith("multipart/form-data;")) {  
                        DiskFileItemFactory dfif = new DiskFileItemFactory();  
                        ServletFileUpload upload = new ServletFileUpload(dfif);  
                        String encodeString = null;  
      
                        try {  
                            List<FileItem> fileItems = upload.parseRequest(request);  
                            request.setAttribute("UPLOAD_FILE_ITEMS", fileItems);  
                            Iterator var10 = fileItems.iterator();  
      
                            while(var10.hasNext()) {  
                                FileItem fileItem = (FileItem)var10.next();  
                                if (fileItem.isFormField()) {  
                                    String itemName = fileItem.getFieldName();  
                                    String itemValue = fileItem.getString("UTF-8");  
                                    if ("className".equals(itemName)) {  
                                        className = itemValue;  
                                    } else if ("methodName".equals(itemName)) {  
                                        methodName = itemValue;  
                                    } else if ("params".equals(itemName)) {  
                                        params = itemValue;  
                                    } else if ("encode".equals(itemName)) {  
                                        encodeString = itemValue;  
                                    }  
                                }  
                            }  
                        } catch (UnsupportedEncodingException | FileUploadException var14) {  
                            LOG.error(var14.getMessage(), var14);  
                        }  
      
                        if (!StringUtil.isNullOrEmpty(encodeString)) {  
                            String[] decode = (String[])((String[])CodeEntry.decode(encodeString, true));  
                            className = decode[0];  
                            methodName = decode[1];  
                            params = decode[2];  
                        }  
                    }  
      
                    if (className == null && methodName == null) {  
                        className = (String)request.getAttribute("className");  
                        methodName = (String)request.getAttribute("methodName");  
                        params = (String)request.getAttribute("params");  
                    }  
      
                    info = new RMIInfo();  
                    info.setClassName(className);  
                    info.setMethodName(methodName);  
                    info.setParams(params);  
                    request.setAttribute("ATTR_KEY_RMIINFO", info);  
                    return info;  
                }  
            }  
        }  
    

## 利用

这时候稍微对漏洞敏感的人已经意识到了一些问题

前面提到了有个windowUnloading参数，如果存在则会对值做解码，并将结果赋给className/methodName/params，

![]()image-20230705102704861

那么我们是不是就能首先根据此对参数做污染，让其帮助我们通过FilterUtil.needToCheck的校验，之后等到了RMIServlet，又通过GET/POST/表单当中的值恢复真实调用

![]()image-20230705102814870

关于构造windowUnloading，为了演示方便我选择else分支，因为这样返回的内容是明文，省去一次解码的问题

![]()image-20230705102953412

当然选择上面这个if分支其实更好，这更方便我们使攻击流量更隐蔽，可以通过传入`jsonpCallback`参数去除解码，

![]()image-20230705103219188

当然为了演示方便我还是选择else分支，任意选择FilterUtil.needToCheck当中的类方法

    
    
    className=UserService&methodName=checkVersion&params=y4  
    

下面做演示，未使用windowUnloading前，调用受限方法会提示未登录(这里以无害方法做演示)

![]()image-20230705103836736

使用后成功调用

![]()image-20230705103930047

通过未授权调用，我们可以获取用户敏感信息包括密码

![]()image-20230705104148109

通过上版本中提到的直接比对数据库方式登录

![]()image-20230705104321315

最终可实现RCE，当然RCE也不止这一种

![]()image-20230705104529432

## 后话

上面提到可以配合RMICoder编解码使流量更隐蔽，同样以第一个无害方法getSystemId为例子

![]()image-20230705105227512  
  

 ** _后记_**

  

  

团队正在招新！期待你的加入 发送简历至 admin@wgpsec.org  
  
  

 ** _作者_**

  

  

![]()

Y4tacker

宁静致远，淡泊明志

  
  

 ** _扫描关注公众号回复加群_**

 ** _和师傅们一起讨论研究~_**

  

 **长**

 **按**

 **关**

 **注**

 **WgpSec狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

  

![]()![]()

  

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

