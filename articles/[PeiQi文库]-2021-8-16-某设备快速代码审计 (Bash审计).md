##  某设备快速代码审计 (Bash审计)

原创 PeiQi文库 [ PeiQi文库 ](javascript:void\(0\);)

**PeiQi文库** ![]()

微信号 PeiQi_wiki

功能介绍 乌拉乌拉！

____

__

收录于话题

#漏洞分析

75个

![](https://gitee.com/fuli009/images/raw/master/public/20210816092036.png)

**![](https://gitee.com/fuli009/images/raw/master/public/20210816092038.png)**

**  
**

**一** **：挖掘思路🐑**

  

![](https://gitee.com/fuli009/images/raw/master/public/20210816092039.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210816092040.png)

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     #!/bin/sh#This script is created by ssparser automatically. The parser first created by MaoShouyanprintf "Content-type: text/htmlCache-Control: no-cache  
    "echo -n ""; . ../common/common.shmyself="/cgi-bin/Maintain/`basename $0`"  
    echo -n "<script languate=\"javascript\">function Validate(frm){  frm.ntpserver.value = TrimAll(frm.ntpserver.value);  if (frm.ntpserver.value != \"\" && !IsIPAddr(frm.ntpserver.value)) {    alert(\"请输入IP地址!\");    frm.ntpserver.select();    return false;  }  return true;}</script>";if [ "${REQUEST_METHOD}" = "POST" ]; then  operator_check "${myself}"  [ "${CGI_ntpserver}" = "" ] && CGI_ntpserver="0.0.0.0"  echo "ntpserver_ip=${CGI_ntpserver}" > ${PGETC}/ntp.conf  timefmt="${CGI_year}${CGI_month}${CGI_day}${CGI_hour}${CGI_minute}.${CGI_second}"  errmsg=`date ${timefmt}`  [ "${CGI_ntpserver}" != "0.0.0.0" ] && ntpdate -t 10 ${CGI_ntpserver}    afm_dialog_msg "操作成功!"fiyear=`date "+%Y"`month=`date "+%m"`day=`date "+%d"`hour=`date "+%H"`minute=`date "+%M"`second=`date "+%S"`if [ -f ${PGETC}/ntp.conf ]; then  . ${PGETC}/ntp.conf  CGI_ntpserver="${ntpserver_ip}"fi[ "${CGI_ntpserver}" = "" ] && CGI_ntpserver="0.0.0.0"  
    echo -n "<body>"; cgi_show_title "系统管理->系统时间" echo -n "<br><form method=post onsubmit=\"return Validate(this)\" action=\"${myself}\"><table width=700 border=0 cellspacing=1 cellpadding=1 bgcolor=\"#ffffff\"><tr id=row1 height=22>  <td width=40></td>  <td width=90 align=left>NTP服务器</td>  <td width=* align=left>    <input type=text name=ntpserver style=\"width:120px\" value=\"${CGI_ntpserver}\"></input>&nbsp;(请输入IP地址，目前不支持域名解析,0.0.0.0表示关闭NTP)</td></tr></table><br><table width=700 border=0 cellspacing=1 cellpadding=1 bgcolor=\"#ffffff\"><tr id=row1 height=22>  <td width=40></td>  <td width=90 align=left>年/月/日</td>  <td width=* align=left>  <select name=year style=\"width:60px\" value=${year}>  ";    tmpvar=2000    while [ ${tmpvar} -le 2020 ]; do      if [ ${tmpvar} -eq ${year} ]; then        echo "<option value=${tmpvar} selected>${tmpvar}</option>"      else        echo "<option value=${tmpvar}>${tmpvar}</option>"      fi      tmpvar=$((${tmpvar} + 1))    done  
    

 ****

![](https://gitee.com/fuli009/images/raw/master/public/20210816092041.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210816092042.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210816092043.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210816092045.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210816092046.png)

 **** ** **  二:  团队招新🦉****

 **  
**

 **团队SRC组招人，觉得自己能力还可以的，平常有时间一起冲排名赚赏金学习的，SRC排名靠前，简历投至 admin@wgpsec.org  **

  

 **团队内部含有知识库、信息收集平台、工具， 简历审核后会有群面，考察基础能力以及学习能力**

![](https://gitee.com/fuli009/images/raw/master/public/20210816092047.png)

 **  
**  

 ** **  三:  关于文库🦉****

 ** **  
****

  

 **    在线文库：**

 **http://wiki.peiqi.tech**

 **  
**

 **    Github：**

 **https://github.com/PeiQi0/PeiQi-WIKI-POC**

![](https://gitee.com/fuli009/images/raw/master/public/20210816092048.png)

 ****  

## 最后

> 下面就是文库的公众号啦，更新的文章都会在第一时间推送在交流群和公众号
>
> 想要加入交流群的师傅公众号点击交流群加我拉你啦~
>
> 别忘了Github下载完给个小星星⭐

  

 **同时知识星球也开放运营啦，希望师傅们支持支持啦🐟**

 **知识星球里会持续发布一些漏洞公开信息和技术文章~**

![](https://gitee.com/fuli009/images/raw/master/public/20210816092049.png)

  

  

  

  

 ****  

 **由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

  

 **PeiQi文库
拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。**

  

  

![]()

PeiQi文库

支持支持啦

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

支持支持啦

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

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

某设备快速代码审计 (Bash审计)

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

