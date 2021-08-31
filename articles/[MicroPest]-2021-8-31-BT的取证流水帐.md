#  BT的取证流水帐

原创 MicroPest [ MicroPest ](javascript:void\(0\);)

**MicroPest** ![]()

微信号 gh_696c36c5382b

功能介绍 个人开发的小工具

____

__

收录于话题

    没什么技术含量，只是备查。

  

    接连碰到几个关于“宝塔”镜像文件的勘验取证工作，每次都是上一次的重复回忆，觉得有点烦，特记录下操作流水帐，方便自己。

  

1、仿真起来，一般都是CentOS7.5的；

  

2、root/123456 登录进入；

  

3、一般都配有固定IP，我们要修改成dhcp；

vi /etc/sysconfig/network-scripts/ifcfg-eth0;

将里面的BOOTPROTO=static修改成dhcp，并屏蔽掉相关的静态IP设置即可；

![](https://gitee.com/fuli009/images/raw/master/public/20210831134915.png)

  

  

4、查看防火墙状态，就用 firewall-cmd --status;

![](https://gitee.com/fuli009/images/raw/master/public/20210831134935.png)

关闭防火墙 systemctl stop firewalld；

![]()

  

5、打开ssh，xshell连入；

systemctl start sshd;

![](https://gitee.com/fuli009/images/raw/master/public/20210831134936.png)

  

6、查看bt，生成了url/username/password

bt default

![]()

  

7、关闭BT中的某些限制

bt

![](https://gitee.com/fuli009/images/raw/master/public/20210831134937.png)

  

8、进入bt平台

![](https://gitee.com/fuli009/images/raw/master/public/20210831134938.png)

  

9、有2个网站  

![](https://gitee.com/fuli009/images/raw/master/public/20210831134939.png)

并配置了几个同网站的域名；

  

10、将网站重定向到本地，修改hosts；

位于c:\windows\system32\drivers\etc\hosts

![]()

将域名指向仿真的IP;

  

11、打开网站

![](https://gitee.com/fuli009/images/raw/master/public/20210831134940.png)

  

12、平台中的数据库

![](https://gitee.com/fuli009/images/raw/master/public/20210831134941.png)

知道了帐号密码，用navicat连接；

  

13、连接数据库

![](https://gitee.com/fuli009/images/raw/master/public/20210831134942.png)

  

这里有个经常性的错误,提示“Access denied for user 'root'@'localhost' (using password:
YES)”，如下解决:

  

1) mysql -u root -p

  

2) mysql> use mysql;

  

3) mysql> update user set password=password('你的密码') where user='root' and
host='%';

  

  

4) mysql> flush privileges;

  

  

5) quit;

  

  

6) bt restart

  

  

14、找到管理员的帐号密码；

![](https://gitee.com/fuli009/images/raw/master/public/20210831134943.png)

  

14、找到后台，进入；

![]()

  

15、网站代码  

一般都放在/www/下，宝塔在/www/server下，网站在/www/wwwroot下；如果要修改，可以在代码下；  

![](https://gitee.com/fuli009/images/raw/master/public/20210831134944.png)

  

  

对服务器日志、进程的勘验，不多说了，这是其它的内容了，跟BT的内容无关了。

  

以上内容，一套走完，估计半小时够了。

![]()

MicroPest

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

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

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

BT的取证流水帐

最多200字，当前共字

__

发送中

写下你的留言

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

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

