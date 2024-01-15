#  基于Spring Boot的在线免杀生成平台

Mr.x  [ SecHub网络安全社区 ](javascript:void\(0\);)

**SecHub网络安全社区** ![]()

微信号 secevery0x01

功能介绍
隶属于SecHub网络安全社区，致力于研究渗透测试、红蓝对抗、应急响应、实战案例、钓鱼社工、和漏洞分析，定期分享实战案例、技术教程、安全工具等前沿网络安全资源。

____

___发表于_

**点击蓝字 关注我们**

![]()

 **免责声明**

本文发布的工具和脚本，仅用作测试和学习研究，禁止用于商业用途，不能保证其合法性，准确性，完整性和有效性，请根据情况自行判断。

如果任何单位或个人认为该项目的脚本可能涉嫌侵犯其权利，则应及时通知并提供身份证明，所有权证明，我们将在收到认证文件后删除相关内容。

文中所涉及的技术、思路及工具等相关知识仅供安全为目的的学习使用，任何人不得将其应用于非法用途及盈利等目的，间接使用文章中的任何工具、思路及技术，我方对于由此引起的法律后果概不负责。

 **添加星标不迷路  
**

由于公众号推送规则改变，微信头条公众号信息会被折叠，为了避免错过公众号推送，请大家动动手指设置“星标”，设置之后就可以和从前一样收到推送啦![]()

  

## 关于

一个基于 Spring Boot 的在线免杀生成平台

# BypassAV-Online

一个基于 Spring Boot 的在线免杀生成平台，还在初期，功能待完善。未来将增加更多编程语言的支持。

![]()

## 特点

  * 在线生成

  * 可多个人使用

  * 随机变量名

  * 随机图标

  * 反沙箱

## 首页

![]()

# 快速体验

## 初次安装

    
    
    wget https://github.com/yutianqaq/BypassAV-Online/releases/download/v1.2/bypassAVOnline.zip  
      
    unzip bypassAVOnline.zip  
      
    cd bypassAVOnline  
      
    curl https://nim-lang.org/choosenim/init.sh -sSf | sh  
    # 输入 y  
    echo "PATH=/home/kali/.nimble/bin:$PATH" >> ~/.zshrc  
    source ~/.zshrc  
      
    # winim 依赖  
    nimble install winim  
      
    sudo apt install mingw-w64  
      
    java -jar bypass-0.0.1-SNAPSHOT.jar  
    

## 前端配置

    
    
    # 前端  
    sudo cp -r dist/assets dist/index.html dist/logo.ico /var/www/html/  
    sudo chown -R www-data:www-data /var/www/html  
    

输入以下命令，开启模块

    
    
    sudo a2enmod proxy  
    sudo a2enmod proxy_http  
    sudo a2enmod rewrite  
    

编辑 `/etc/apache2/sites-available/000-default.conf` 增加以下配置

    
    
            ProxyPass /api http://localhost:8080  
            ProxyPassReverse /api http://localhost:8080  
            <Directory /var/www/html>  
                    Options Indexes FollowSymLinks  
                    AllowOverride All  
                    Require all granted  
            </Directory>  
      
    

![]()

重启 Apache 服务器

    
    
    sudo systemctl restart apache2  
    

至此完成后端，前端配置

## 非初次安装

从 https://github.com/yutianqaq/BypassAV-Online/releases/latest 下载最新发行版

    
    
    unzip bypassAVOnline.zip  
    cd bypassAVOnline  
      
    java -jar bypass-0.0.1-SNAPSHOT.jar  
    sudo cp -r dist/assets dist/index.html dist/logo.ico /var/www/html/  
    sudo chown -R www-data:www-data /var/www/html  
    sudo systemctl restart apache2  
    

# 从源码构建搭建文档

Spring Boot 后端、Vue 前端。

## 部署

### 后端

使用 idea 打开，点击 右侧 maven，点击 package

![]()

然后在 target 目录会生成一份 `bypass-0.0.1-SNAPSHOT.jar`。

输入下面的命令

    
    
    cd /home/kali  
    mkdir bypassAVOnline  
    cd bypassAVOnline  
    mkdir template  
    mkdir download  
    

新建一个名为 application.yaml 的文件，输入以下内容

    
    
    bypassav:  
      templatesDirectory: /home/kali/bypassAVOnline/template/  
      templateCMapping:  
        v1: v1.c  
      templateNIMMapping:  
        v1: v1.nim  
        v2: x2Ldr-Plus.nim  
      compilerC: x86_64-w64-mingw32-gcc  
      compilerNIM: nim  
      storageDirector: /home/kali/bypassAVOnline/download  
    spring:  
      servlet:  
        multipart:  
          enabled: true  
          max-file-size: 1MB  
          max-request-size: 1MB  
    

此时后端工作目录结构如下：

    
    
    ┌──(kali㉿kali)-[~/bypassAVOnline]  
    └─$ tree                                                                                  
    .  
    ├── application.yaml  
    ├── bypass-0.0.1-SNAPSHOT.jar  
    ├── download  
    └── template  
        ├── v1.c  
        └── v1.nim  
      
    3 directories, 4 files  
      
    

下载编译器

输入下面的命令

    
    
    curl https://nim-lang.org/choosenim/init.sh -sSf | sh  
    # 输入 y  
    export PATH=/home/kali/.nimble/bin:$PATH  
      
    sudo apt install mingw-w64  
    

此状态为安装成功。

![]()

启动后端，成功状态如下

    
    
    curl -X POST -H "Content-Type: application/json" -d '{"code":"0x90", "templateName":"v1"}' http://localhost:8080/v1/compileC  
      
    wget http://localhost:8080/v1/download/c7af340b-8358-44a3-9985-82189468c36d.exe  
    

![]()

## 前端页面部署

输入 `yarn build`，然后输入以下命令，复制到 Apache 目录

    
    
    sudo cp -r dist/assets dist/index.html dist/logo.ico /var/www/html/  
    sudo chown -R www-data:www-data /var/www/html  
    

输入以下命令，开启模块

    
    
    sudo a2enmod proxy  
    sudo a2enmod proxy_http  
    sudo a2enmod rewrite  
    

编辑 `/etc/apache2/sites-available/000-default.conf` 增加以下配置

    
    
            ProxyPass /api http://localhost:8080  
            ProxyPassReverse /api http://localhost:8080  
            <Directory /var/www/html>  
                    Options Indexes FollowSymLinks  
                    AllowOverride All  
                    Require all granted  
            </Directory>  
      
    

![]()

重启 Apache 服务器

    
    
    sudo systemctl restart apache2  
    

至此完成后端，前端配置

# TODO

  *  优化前端交互

  *  增加更多模板

  *  增加免杀性

 **项目地址**  

  * 

    
    
     https://github.com/yutianqaq/BypassAV-Online

  

  

欢迎关注SecHub网络安全社区，SecHub网络安全社区目前邀请式注册，邀请码获取见公众号菜单【邀请码】

![]()

 **联系方式**

电话｜010-86460828

官网｜http://www.secevery.com

![]()

 **关注我们**

![]()![]()![]()

 **公众号：** sechub安全

 **哔哩号：** SecHub官方账号

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 基于Spring Boot的在线免杀生成平台

Mr.x  [ SecHub网络安全社区 ](javascript:void\(0\);)

轻触阅读原文

![]()

SecHub网络安全社区

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

