##  openrasp-iast 灰盒扫描工具

原创 Bypass [ Bypass ](javascript:void\(0\);)

**Bypass** ![]()

微信号 Bypass--

功能介绍 致力于分享原创高质量干货，包括但不限于：渗透测试、WAF绕过、代码审计、安全运维。 闻道有先后，术业有专攻，如是而已。

____

__

收录于话题

openrasp-iast 是一款灰盒扫描工具，能够结合应用内部hook点信息精确的检测漏洞，需安装Agent和扫描器，支持java、PHP等应用程序。

在这里，我们通过docker部署控制台，接入一个PHP应用进行测试体验。

* * *

 **1、快速搭建环境**

使用容器快速搭建一整套的测试环境，包含 IAST 扫描器、OpenRASP 管理后台 以及 漏洞测试用例。

  *   *   *   *   *   * 

    
    
    sudo sysctl -w vm.max_map_count=262144git clone https://github.com/baidu-security/openrasp-iast.gitcd openrasp-iast/docker/iast-clouddocker-compose up#http://127.0.0.1:18662/vulns/ 触发PHP测试用例#http://127.0.0.1:18660/ 云控后台(账号openrasp/admin@123)

 **2、接入PHP应用程序**

（1）使用lnmp一键安装PHP环境，导入dvwa源码运行。

  * 

    
    
    wget http://soft.vpser.net/lnmp/lnmp1.7.tar.gz -cO lnmp1.7.tar.gz && tar zxf lnmp1.7.tar.gz && cd lnmp1.7 && ./install.sh lnmp

（2）使用默安账户登录控制台，添加一个新应用。

![](https://gitee.com/fuli009/images/raw/master/public/20210823082029.png)

（3）切换到新的应用test，点击右上角的添加主机，安装agent。

![]()

PS：遇到重启PHP服务报错，需要查看openrasp.so 路径，如复制不成功，手工复制过去即可。

![](https://gitee.com/fuli009/images/raw/master/public/20210823082030.png)

 （4）安装iast扫描器

  * 

    
    
    wget https://packages.baidu.com/app/openrasp/openrasp-iast-latest -O /usr/bin/openrasp-iast

![](https://gitee.com/fuli009/images/raw/master/public/20210823082031.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210823082032.png)

复制上面的最后一个url，设置为Fuzz 服务器地址。

![](https://gitee.com/fuli009/images/raw/master/public/20210823082033.png)

（5）启动扫描，访问站点触发。

![](https://gitee.com/fuli009/images/raw/master/public/20210823082034.png)

(6) 查看漏洞列表

![](https://gitee.com/fuli009/images/raw/master/public/20210823082035.png)

![]()

Bypass

如有帮助，请随意打赏。

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

如有帮助，请随意打赏。

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

openrasp-iast 灰盒扫描工具

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

