#  开源项目Dolphin-网络资产风险监测系统

汤青松  [ 渗透云笔记 ](javascript:void\(0\);)

**渗透云笔记** ![]()

微信号 shentouyun

功能介绍
分享学习网络安全的路上，把自己所学的分享上来，帮助一些网络小白避免“踩坑”，既节省了学习时间，少走了弯道，我们也可以去回顾自己所学，分享自己所得与经验，为此我们感到光荣。

____

___发表于_

收录于合集

## 文章来源于 QingScan  

## 项目简介

Dolphin 是一个的资产风险分析系统,用户仅需将一个主域名添加到系统中,dolphin会自动抓取与该域名相关的信息进行分析;

例如同ICP域名,子域名,对应IP,端口,URL地址,站点截图,端口协议,邮箱地址,泄露信息等.

前端使用了bootstrap框架,控制台使用的ThinkPHP; 底层数据来自于蜻蜓平台的数据聚合系统,调用了各类框架和API.

GitHub地址: https://github.com/StarCrossPortal/dolphin

## 使用方法

将公司的主域名填进去,dolphin能采集到`同ICP域名`,`子域名`,`IP`,`端口`,`URL`,`站点截图`,`漏洞`,`产品指纹`,`泄露信息`,`邮箱`,`证书`

## 安装方法

  1. 下载代码:`git clone --depth=1 https://github.com/StarCrossPortal/dolphin.git && cd dolphin`

  2. 启动docker`docker run -it -d -p 80:80 -v $(pwd):/root/code --name dolphin daxia/qingting:dolphin`

  3. 安装依赖`docker exec -it dolphin bash -c 'cd /root/code && composer install'`

  4. 新建MySQL数据库,把`xinxishouji.sql`文件导入进去

  5. 复制`.example.env`为`.env`,并修改数据库地址信息

  6. 浏览器打开地址:`http://xxxx/admin/home/index`

## 感谢

  1. 项目UI体验,灵感来自于0xbug大佬的biu系统`https://github.com/0xbug/Biu`

  2. 数据由蜻蜓驱动,地址`http://qingting.starcross.cn/scenario/detail?id=2054`

## 效果图

![](https://gitee.com/fuli009/images/raw/master/public/20230309135937.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140000.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140001.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140003.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140005.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140006.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140007.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140009.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230309140010.png)

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

