#  红帆OA ioFileDown.aspx 存在任意文件下载漏洞

原创 非攻安全实验室  [ 非攻安全实验室 ](javascript:void\(0\);)

**非攻安全实验室** ![]()

微信号 gh_9c3b7f864fba

功能介绍 一个不会编程、挖 SRC、代码审计的安全爱好者。

____

___发表于_

收录于合集 #漏洞复现 20个

**导读  
**



      主要分享学习日常中的web渗透，内网渗透、漏洞复现、工具开发相关等。希望以技术共享、交流等不断赋能自己，为网络安全发展贡献自己的微薄之力！    

![]()  
 **0x00免责声明** **本次测试仅供学习使用，** **如若非法他用，与平台和本文作者无关，需自行负责！**![]()  
 **0x01漏洞描述**

  

   红帆OA ioFileDown.aspx 存在任意文件下载漏洞，攻击者可通过此漏洞获取敏感信息。

  

![]()  
 **0x02漏洞复现**  

1、fofa

  * 

    
    
    app="红帆-ioffice"

![]()

2、复现如下（大部分都存在，利用方式请加入帮会获取）

![]()

![]()

  

![]()  
 **0x03修复建议**  
  

  *   *   * 

    
    
     1.对用户传过来的文件名参数进行硬编码或统一编码，对文件类型进行白名单控制，对包含恶意字符或者空字符的参数进行拒绝；2.文件路径保存至数据库，让用户提交文件对应ID下载文件；3.用户下载文件之前需要进行权限判断；

  

![]()  
 **0x04知识大陆**  
  

    因为很多未公开或者小范围公开的漏洞不能直接发公众号，所以后面部分漏洞会直接整理发表在知识大陆，持续更新，目前进入需要19.9元。（目前帮会已更新漏洞61+，每天更新1-3个漏洞，有兴趣的小伙伴可以加入帮会或者群一起讨论交流），访问链接或者扫描二维码如下：

  * 

    
    
    https://wiki.freebuf.com/front/societyFront?invitation_code=2f34044b&society_id=107&source_data=2

![]()

10月更新内容如下（活动期间会持续更新一些0和1）

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    1、广联达OA存在信息泄露2、致远OA 帆软组件 ReportServer目录遍历漏洞3、万户协同办公平台 ezoffice存在未授权访问漏洞4、北京万户软件技术有限公司存在sql注入5、360 新天擎终端安全管理系统信息泄露漏洞6、金蝶产品-任意文件读取7、启明天钥安全网关多个漏洞8、亿赛通update.jsp存在SQL注入9、锐捷网络股份有限公司校园网自助服务系统存在目录穿越漏洞10、深圳市朗驰欣创科技股份有限公司视频监控系统绕过登录漏洞11、用友U8 slbmbygr.jsp 存在sql注入12、Panabit panalog 任意用户创建漏洞和后台命令执行13、NUUO 摄像头远程命令执行漏洞14、泛微 do.php 存在未授权访问15、北京金盘鹏图软件技术有限公司金盘移动图书馆系统存在任意文件下载16、亿赛通 UploadFileList 任意文件读取17、用友U8 bx_dj_check.jsp sql注入18、泛微e-office存在前台文件上传漏洞19、泛微e-office—任意文件读取20、广州图创计算机软件开发有限公司图书馆集群管理系统存在逻辑绕过21、用友-移动系统管理任意文件读取22、用友-移动系统管理任意文件上传23、济南上邦电子科技有限公司电子文档安全管理系统 V6.0存在任意文件下载24、广东飞企互联科技股份有限公司企业运营管理平台存在登录绕过25、郑州天迈科技股份有限公司天迈科技网络视频监控系统存在命令执行26、北京亚鸿世纪科技发展有限公司企业侧互联网综合管理平台存在远程命令执行27、成都任我行软件股份有限公司管家婆分销ERP系统存在sql注入漏洞28、速达软件技术（广州）有限公司多款产品存在代码执行29、上海普华科技发展股份有限公司PowerPMS存在信息泄露30、大华智慧园区 attachment_downloadByUrlAtt.action 任意文件读取31、用友移动系统管理存在两处SQL注入漏洞32、泛微 emessage 管理界面 存在任意文件读取33、深圳市强鸿电子有限公司鸿运主动安全云平台存在任意文件下载漏洞34、深圳市强鸿电子有限公司鸿运主动安全云平台存在sql注入35、北京南北天地科技股份有限公司ERP系统存在未授权访问36、北京南北天地科技股份有限公司ERP系统存在sql注入37、e-office协同办公平台Init.php存在SQL注入漏洞38、广东飞企互联科技股份有限公司企业运营管理平台存在任意文件读取39、成都海翔软件有限公司海翔药业云平台存在sql注入40、网神下一代极速防火墙存在任意文件下载41、华测监测预警系统任意读取漏洞42、江苏叁拾叁信息技术有限公司OA存在sql注入43、用友GRP-UP-U8 listSelectDialogServlet 存在sql注入44、金和OA GetTreeDate.aspx SQL注入46、用友NC cloud uploadChunk 存在任意文件上传47、EasyCVR 视频管理平台存在信息泄露48、北京宏景世纪软件股份有限公司人力资源信息管理系统存在xxe漏洞49、e-office协同办公平台json_common.php存在SQL注入漏洞50、深信服NGAF下一代防火墙loadfile.php存在任意文件读取漏洞51、红帆HFOffice是广泛应用于医院的OA系统list接口存在SQL注入漏洞52、福建博思软件股份有限公司博斯软件V6.0存在sql注入53、金蝶云星空存在任意文件读取54、用友致远A6 operaFileActionController.jsp 任意文件读取漏洞55、泛微 download.php 任意文件读取56、金和 clobfield 存在sql注入57、蓝凌EIS智慧协同平台 api.aspx 接口存在任意文件上传58、H2db console 未授权访问59、万户OA DocumentEdit_unite.jsp 存在sql注入60、泛微 e-Mobile 移动管理平台文件上传漏洞61、红帆OA ioFileDown.aspx 存在任意文件下载漏洞

目前帮会正在参与FreeBuf网安知识大陆的【白帽比武大会】活动，进入帮会或者进入群的好兄弟帮忙点点人气（1人1天5票），点得越多更新越快。

（漏洞盒子十月企业SRC榜单前50，公益前30的有两个免费加入名额）

![]()

  

    加入帮会后扫描二维码加入群聊，（超200人需要一个一个拉比较麻烦，索性直接建立个3群，已加入的就不要重复加了，感谢）。

![]()

![]()  
 **0x05往期内容**  
  

[1、京信通信系统中国有限公司CPE-
WiFi存在任意用户添加&命令执行](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483674&idx=1&sn=09cde828189029b86ad532ee6e61c084&chksm=c32276f1f455ffe74c7a5e97099a4033dc169b880075f536eed6970b8a500901d43edf6d74af&scene=21#wechat_redirect)

[2、泛微 emessage
管理界面存在任意文件读取漏洞](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483696&idx=1&sn=51eadf07551d81ef0387484402745d00&chksm=c32276dbf455ffcdd4a07631f69beef9990e220ffde81b6575c67c7cf0bc468c535af7b594bc&scene=21#wechat_redirect)  

[3、飞企互联企业运营管理平台存在任意文件读取](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483715&idx=1&sn=9f63136b2a7ff2a174e48538eb78f1a8&chksm=c32276a8f455ffbe5c95068cb2ee3af5c71e6cbb5bd1946d792550917a51c6b3f4371c1e6f11&scene=21#wechat_redirect)  

[4、泛微e-office OfficeServer2.php
存在任意文件读取漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483731&idx=1&sn=56e9cff8aa7f6a1ad9504e12abdfe018&chksm=c32276b8f455ffaea93653dbb3d0273896e4c264b1a920b2355b793ef7f93904f3bb70c15884&scene=21#wechat_redirect)

[5、广联达OA存在信息泄露漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483755&idx=1&sn=ab7de9d29375c732d7b919ff16c760a4&chksm=c3227680f455ff9639e5a435a896bdb26f51ebabce80bbf62fcaa1dfe93a3e65bae8e167807f&scene=21#wechat_redirect)

[6、NUUO
摄像头远程命令执行漏洞](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483866&idx=1&sn=4ce87b5403f541fd2c16ad1be05bba6d&chksm=c3227631f455ff2792c8995e7f412df4a49a41d82db4d443ba1a5cda9fac71fe89303d787781&scene=21#wechat_redirect)  

[7、Panabit panalog
任意用户创建漏洞和后台命令执行](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483873&idx=1&sn=684f8dcaa777a7da71be7c038024ca10&chksm=c322760af455ff1c2486c153f3752df434a078dfe0ee46126b9bacb18f1612067898dbdba025&scene=21#wechat_redirect)  

[8、用友U8 slbmbygr.jsp
存在sql注入](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483874&idx=1&sn=f7f2c4932b369f33527571f0aeafe2c2&chksm=c3227609f455ff1f94521e6fa7d95a2f6999388e1a8b5b17d2af5afbc4f790f27433d42acdf7&scene=21#wechat_redirect)  

[9、用友移动系统管理存在前台SQL注入漏洞](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483883&idx=1&sn=ba8610c59c524eb24fcfdf558f166697&chksm=c3227600f455ff16d5e0c449af6de1e9554e5b505d78f51c64b3996e3e14280f61523249dae5&scene=21#wechat_redirect)

[10、金盘移动图书馆系统 download.jsp
任意文件下载](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483891&idx=1&sn=a637fa6bc6947263d6537359cfde4d01&chksm=c3227618f455ff0e32cfdab58f4d6f87ee4aaa4f8be552ba90bb44267db48b19029438911fdc&scene=21#wechat_redirect)

[11、大华智慧园区管理平台存在任意文件读取](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483912&idx=1&sn=b4527f4cb8c3ebc7600f9399cf492cf4&chksm=c32275e3f455fcf5ea5e471c0868821668ce9dd81ac27a2d95e868d8a9c101f870e4edca26d8&scene=21#wechat_redirect)

12、[广州图创计算机软件开发有限公司图书馆集群管理系统存在逻辑绕过](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483931&idx=1&sn=3d2996689f48f0eaf84b5701131fad60&chksm=c32275f0f455fce61002c0cae83b18dbcdc42e027537a7c8446586634702c6baca19d6532572&scene=21#wechat_redirect)  

[13、深圳市强鸿电子有限公司鸿运主动安全云平台存在任意文件下载漏洞](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483954&idx=1&sn=0410d4bcc1b9bc5a69521f3bbf942109&chksm=c32275d9f455fccf736a69fdb79be810cf76e4f6cb675d0f08607d088e7515d3c19d4a3b8934&scene=21#wechat_redirect)

14、[深圳市北京南北天地科技股份有限公司ERP系统存在sql注入](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247483981&idx=1&sn=daaddd165b48a541970bed8a666b6bce&chksm=c32275a6f455fcb0e4282e515f98a56997ad8523d5f04f864041b8630e72062690c2cdd16345&scene=21#wechat_redirect)  

15、[北京通达信科科技有限公司通达OA2016网络智能办公系统 handle.php
存在sql注入](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484003&idx=1&sn=c01aa198911ea47e82f40019c939ebb4&chksm=c3227588f455fc9eec286f0f5a107fbb85a82ef2769da9f0cf052e928ecff71ad4ec357a289c&scene=21#wechat_redirect)

16、[成都海翔软件有限公司海翔药业云平台存在sql注入](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484028&idx=1&sn=b16cd36209f0b930cc1cce777b4e3216&chksm=c3227597f455fc819400251a3433608b85f164c078e10754dafd1c4137dee4a2eab84ebd70f4&scene=21#wechat_redirect)

[17、用友NC cloud uploadChunk
存在任意文件上传](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484058&idx=1&sn=eacfaf5fa130a5aa910203483c7e988f&chksm=c3227571f455fc67e6ab0432d973e40dc4762a1914faffd52f429bc25592c80a222df43b45c0&scene=21#wechat_redirect)

[18、e-office协同办公平台Init.php存在SQL注入漏洞](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484085&idx=1&sn=1cb753ce0da610214934acbd4f8cb212&chksm=c322755ef455fc482529d35b957c4e83714efb68a94cc2dbd0182b89ac385b693b9c987be669&scene=21#wechat_redirect)

[19、e-office协同办公平台json_common.php存在SQL注入](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484105&idx=1&sn=25acedbe14551d52b064b3bc87c1f32f&chksm=c3227522f455fc3455f707b8f00f9ed7c0d99ecd27cd27bbcba0be50b900f73b3129d4cb94cd&scene=21#wechat_redirect)

[20、用友 GRP-U8 license_check.jsp
存在sql注入](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484127&idx=1&sn=eef5d2cb86a6ffdb98a1e699899bfcca&chksm=c3227534f455fc22b02bc79877d35a40c53526b63f030cd81e70c4b3eb35e1f6bbc6066e0d0d&scene=21#wechat_redirect)

21、[蓝凌EIS智慧协同平台 api.aspx
接口存在任意文件上传](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484147&idx=1&sn=ad2152b9d0dedbd2c6b32f4565e4d213&chksm=c3227518f455fc0ed75bbd49977eec7ff104678fa1faf7adeeb0689d9b6776ecb19643f9567e&scene=21#wechat_redirect)

22、[万户OA DocumentEdit_unite.jsp
存在sql注入](http://mp.weixin.qq.com/s?__biz=Mzk0NDUzMDA1Mg==&mid=2247484165&idx=1&sn=feaf763be0d68f58483b0737229b8e35&chksm=c32274eef455fdf850aa584c102b52c20d9fab1c060f20a56b5ed67c44b6e7ce0c8876b9892c&scene=21#wechat_redirect)  

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

