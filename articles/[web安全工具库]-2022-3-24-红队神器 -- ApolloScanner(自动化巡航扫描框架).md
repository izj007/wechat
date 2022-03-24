#  红队神器 -- ApolloScanner(自动化巡航扫描框架)

[ web安全工具库 ](javascript:void\(0\);)

**web安全工具库** ![]()

微信号 websec-tools

功能介绍 将一些好用的web安全工具和自己的学习笔记分享给大家。。。

____

__

收录于话题

#自动化 1 个

#神器 2 个

#红队工具 1 个

#网络安全 30 个

#渗透测试 29 个

文末有定制礼品福利

  

一、软件界面

![](https://gitee.com/fuli009/images/raw/master/public/20220324084947.png)

  

二、功能介绍

1、资产收集（需要主域名，资产对象可直接在爆破和漏扫过程中调用）

子域名收集（需要virustotal-api-
token）、cname收集、ip地址（a记录）收集、开放端口扫描（基于masscan）、端口对应服务、组件指纹版本探测（基于nmap）、http标题探测、http框架组件探测

  

2、github敏感信息收集

基于域名和关键字的敏感信息收集（需要github-token）

  

3、暴力破解（基于exp的暴力破解）

exp注册模块：代码动态编辑、代码动态调试、支持资产对象

破解任务模块：支持exp对象调用、支持资产对象、支持批量资产、支持多线程（可配置）

破解结果模块：支持结果显示、支持钉钉通知  

敏感路径探测任务

敏感路径探测结果

  

4、漏洞扫描模块

exp注册模块：代码动态编辑、代码动态调试、支持资产对象

漏扫任务模块：支持exp对象调用、支持资产对象、支持批量资产、支持多线程（可配置)

结果显示模块：支持结果显示、支持钉钉通知

  

5、配置模块

支持常用系统配置（各类token、线程数）

支持用户、用户组、权限配置模块

支持启动服务模块：HTTP服务（支持HTTP请求记录）、DNS服务（支持DNS请求记录）

  

三、exp编写规范

1、暴力破解

  *   *   *   *   *   * 

    
    
    def brute_scan_function_name(ipaddress, port, username, password, logger):      import xx_module # 引入模块全部在函数内容写    # ...     # ...是爆破exp核心代码    logger.log("xxxxx") # 代替print    return True  # 返回必须是true/false

2、漏扫扫描  

  *   *   *   *   *   * 

    
    
    def brute_scan_function_name(ipaddress, port, logger):      import xx_module # 引入模块全部在函数内容写    # ...     # ...是漏扫exp核心代码    logger.log("xxxxx") # 代替print    return True  # 返回必须是true/false

四、安装方式  

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    python版本：3.8.x 或 3.9.xdjango版本：4.0.1nmap：需要masscan：需要mysql前端：基于simple-ui支持操作系统：MacOS Monterey 12.3 / Ubuntu 18.04 LTSsudo python3 -m pip install -r requirments.txtsudo python3 manage.py migratesudo python3 manage.py createsuperusersudo python3 manage.py runserver

五、下载地址  

1、https://github.com/b0bac/ApolloScanner

2、关注公众号：逆向有你，回复20220323

  

好书推荐

![](https://gitee.com/fuli009/images/raw/master/public/20220324084958.png)

>
> 《C++码农日记（全程视频讲解）》共9章。第1章讲述程序员入职前的准备以及C++跨平台开发入门知识，着重介绍求职面试相关知识，以及Qt的安装配置、开发环境搭建、第三方跨平台库基础知识、配套资源等内容；第2~8章通过50多个实际案例讲述命令行程序的开发、DLL（动态链接库）的开发与第三方库的使用、跨平台文件操作、多线程和进程内（多线程间）通信、进程间通信、异步串口通信、数据库访问等常用开发技能；第9章通过一个数据中心的案例介绍C/S模式（Client/Server模式，客户端/服务器模式）软件的综合开发技能。本书提供的案例覆盖了C/S模式软件开发工作的常见场景。

  

加我微信(ivu123ivu)，发送本篇文章的‘点赞’‘在看’及分享朋友圈的截图，  

获取抽奖福利（少客联盟定制鼠标垫*5+少客联盟邀请码*5），仅当天有效。

  

![](https://gitee.com/fuli009/images/raw/master/public/20220324084959.png)

  

预览时标签不可点

收录于话题 #

 个

上一篇 下一篇

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

