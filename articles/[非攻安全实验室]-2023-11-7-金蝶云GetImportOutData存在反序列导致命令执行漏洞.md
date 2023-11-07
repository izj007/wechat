#  金蝶云GetImportOutData存在反序列导致命令执行漏洞

原创 非攻安全实验室  [ 非攻安全实验室 ](javascript:void\(0\);)

**非攻安全实验室** ![]()

微信号 gh_9c3b7f864fba

功能介绍 一个不会编程、挖 SRC、代码审计的安全爱好者。

____

___发表于_

收录于合集 #漏洞复现 27个

**导读  
**



      主要分享学习日常中的web渗透，内网渗透、漏洞复现、工具开发相关等。希望以技术共享、交流等不断赋能自己，为网络安全发展贡献自己的微薄之力！    

![]()  
 **0x00免责声明** **本次测试仅供学习使用，** **如若非法他用，与平台和本文作者无关，需自行负责！**![]()  
 **0x01漏洞描述**

  

    金蝶云GetImportOutData存在反序列导致命令执行漏洞,攻击者可通过此漏洞获取服务器权限。

  

![]()  
 **0x02漏洞复现**  

1、fofa

  * 

    
    
    app="Kingdee-K3-cloud"

![]()

2、界面如下

![]()

3、复现如下（利用方式请加入帮会获取）

![]()

  

![]()  
 **0x03修复建议**  
  

  * 

    
    
     升级到安全版本

  

![]()  
 **0x04知识大陆**  
  

    因为很多未公开或者小范围公开的漏洞不能直接发公众号，所以后面部分漏洞会直接整理发表在知识大陆，持续更新，目前进入需要19.9元。（目前帮会已更新漏洞80+，有兴趣的小伙伴可以加入帮会）

  * 

    
    
    https://wiki.freebuf.com/front/societyFront?invitation_code=2f34044b&society_id=107&source_data=2

![]()

更新漏洞清单如下：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    广联达OA存在信息泄露致远OA 帆软组件 ReportServer目录遍历漏洞万户协同办公平台 ezoffice存在未授权访问漏洞北京万户软件技术有限公司存在sql注入360 新天擎终端安全管理系统信息泄露漏洞金蝶产品-任意文件读取启明天钥安全网关多个漏洞亿赛通update.jsp存在SQL注入锐捷网络股份有限公司校园网自助服务系统存在目录穿越漏洞深圳市朗驰欣创科技股份有限公司视频监控系统绕过登录漏洞用友U8 slbmbygr.jsp 存在sql注入Panabit panalog 任意用户创建漏洞和后台命令执行NUUO 摄像头远程命令执行漏洞泛微 do.php 存在未授权访问北京金盘鹏图软件技术有限公司金盘移动图书馆系统存在任意文件下载亿赛通 UploadFileList 任意文件读取用友U8 bx_dj_check.jsp sql注入泛微e-office存在前台文件上传漏洞泛微e-office—任意文件读取广州图创计算机软件开发有限公司图书馆集群管理系统存在逻辑绕过用友-移动系统管理任意文件读取用友-移动系统管理任意文件上传济南上邦电子科技有限公司电子文档安全管理系统 V6.0存在任意文件下载广东飞企互联科技股份有限公司企业运营管理平台存在登录绕过郑州天迈科技股份有限公司天迈科技网络视频监控系统存在命令执行北京亚鸿世纪科技发展有限公司企业侧互联网综合管理平台存在远程命令执行成都任我行软件股份有限公司管家婆分销ERP系统存在sql注入漏洞速达软件技术（广州）有限公司多款产品存在代码执行上海普华科技发展股份有限公司PowerPMS存在信息泄露大华智慧园区 attachment_downloadByUrlAtt.action 任意文件读取用友移动系统管理存在两处SQL注入漏洞泛微 emessage 管理界面 存在任意文件读取深圳市强鸿电子有限公司鸿运主动安全云平台存在任意文件下载漏洞深圳市强鸿电子有限公司鸿运主动安全云平台存在sql注入北京南北天地科技股份有限公司ERP系统存在未授权访问北京南北天地科技股份有限公司ERP系统存在sql注入e-office协同办公平台Init.php存在SQL注入漏洞广东飞企互联科技股份有限公司企业运营管理平台存在任意文件读取成都海翔软件有限公司海翔药业云平台存在sql注入网神下一代极速防火墙存在任意文件下载华测监测预警系统任意读取漏洞江苏叁拾叁信息技术有限公司OA存在sql注入用友GRP-UP-U8 listSelectDialogServlet 存在sql注入金和OA GetTreeDate.aspx SQL注入用友NC cloud uploadChunk 存在任意文件上传EasyCVR 视频管理平台存在信息泄露北京宏景世纪软件股份有限公司人力资源信息管理系统存在xxe漏洞e-office协同办公平台json_common.php存在SQL注入漏洞深信服NGAF下一代防火墙loadfile.php存在任意文件读取漏洞红帆HFOffice是广泛应用于医院的OA系统list接口存在SQL注入漏洞福建博思软件股份有限公司博斯软件V6.0存在sql注入金蝶云星空存在任意文件读取用友致远A6 operaFileActionController.jsp 任意文件读取漏洞泛微 download.php 任意文件读取金和 clobfield 存在sql注入蓝凌EIS智慧协同平台 api.aspx 接口存在任意文件上传H2db console 未授权访问万户OA DocumentEdit_unite.jsp 存在sql注入泛微 e-Mobile 移动管理平台文件上传漏洞红帆OA ioFileDown.aspx 存在任意文件下载漏洞用友-U8-Cloud upload.jsp 存在任意文件上传用友GRP-U8 forgetPassword_old.jsp 存在sql注入浙江大华技术股份有限公司智能物联综合管理平台(ICC)存在逻辑漏洞Casdoor系统 存在信息泄露广联达 test.aspx 存在信息漏洞致远oa-getAjaxDataServlet-xxe任意文件读取漏洞大为计算机软件开发有限公司知识产权协同创新管理系统任意密码重置Saber企业级开发平台任意登录浙江大华技术股份有限公司智能物联综合管理平台(ICC)存在任意文件读取多个厂商AC集中管理平台存在信息泄露某司酒店智慧营销系统存在sql注入用友 com.ufida.web.action.ActionServlet 未授权访问华为Auth-Http Server 1.0任意文件读取xxl-job-默认accessToken权限绕过漏洞金蝶云SaveUserPassport存在反序列导致命令执行漏洞金蝶云GetImportOutData存在反序列导致命令执行漏洞Samsung-WLAN-AP路由器RCE

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

