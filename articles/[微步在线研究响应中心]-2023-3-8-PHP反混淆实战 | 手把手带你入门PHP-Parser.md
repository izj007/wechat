#  PHP反混淆实战 | 手把手带你入门PHP-Parser

原创 子推  [ 微步在线研究响应中心 ](javascript:void\(0\);)

**微步在线研究响应中心** ![]()

微信号 gh_280024a09930

功能介绍 微步情报局最新威胁事件分析、漏洞分析、安全研究成果共享，探究网络攻击的真相

____

___发表于_

收录于合集 #安全报告 105个

![](https://gitee.com/fuli009/images/raw/master/public/20230308191039.png)1  
  

 **            ****什么是PHP-Parser ?**  

PHP-
Parser是由Nikic开发的一款PHP抽象语法树（AST）解析工具。能够将PHP代码转换为抽象语法树，安全研究员可以通过生成的语法树对PHP样本进行控制流图生成、静态分析和污点检测等操作，同时其组合模式的设计使得每个节点操作的处理相互独立，后期维护十分方便。因此在PHP
Webshell检测领域中被广泛使用。同时对于生成的抽象语法树也可以进行操作，从代码节点的角度对PHP文件进行修改，所以通过PHP-
Parser进行代码的混淆和反混淆是十分方便的。在非安全领域，PHP-
Parser也可以自动帮你补全单元测试框架、检查代码问题。接下来从笔者从事相关工作的经验来探讨下通过PHP-
Parser进行反混淆的方法，通过对某CTF混淆样本进行反混淆让大家初步掌握使用PHP-Parser反混淆的方法。2  
  

 **            ****PHP-Parser入门**

此章主要介绍PHP-Parser的创建过程以及在实战中需要用的方法、节点类型等。 **1.创建解析器实例** 要使用PHP-
Parser，首先需要创建实例，在创建时选择解析的语言版本，声明格式如下：  

  *   * 

    
    
    use PhpParser\ParserFactory;$parser = (new ParserFactory)->create(ParserFactory::PREFER_PHP7);

其中ParserFactory的参数有以下四种： **参** **数**|  **效果**  
---|---  
ParserFactory::PREFER_PHP7| 优先解析PHP7，如果PHP7解析失败则将脚本解析成PHP5  
ParserFactory::PREFER_PHP5| 优先解析PHP5，如果PHP5解析失败则将脚本解析成PHP7  
ParserFactory::ONLY_PHP7| 只解析成PHP7  
ParserFactory::ONLY_PHP5| 只解析成PHP5  
 **2.解析PHP代码**

通过解析器的parse方法将PHP代码解析成抽象语法树：

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?phpuse PhpParser\Error;use PhpParser\ParserFactory;        require 'vendor/autoload.php';         $code = file_get_contents("./test.php");$parser = (new ParserFactory)->create(ParserFactory::PREFER_PHP7);try {    $ast = $parser->parse($code);} catch (Error $error) {    echo "Parse error: {$error->getMessage()}\n";}

 **3.输出抽象语法树** 通过Node Dumping我们可以生成一个直观的AST，例如我们使用view.php来解析sample.php：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    //view.php<?php           
    require 'vendor/autoload.php';use PhpParser\Error;use PhpParser\NodeDumper;use PhpParser\ParserFactory;//获取sample.php的代码内容$code = file_get_contents('sample.php');//初始化解析器$parser = (new ParserFactory)->create(ParserFactory::PREFER_PHP7);try {    //解析sample.php内容，转换为ast   $ast = $parser->parse($code);} catch (Error $error) {    echo "Parse error: {$error->getMessage()}\n";    return;}            
    $dumper = new NodeDumper;//优化ast并dumpecho $dumper->dump($ast) . "\n";

sample.php的解析效果如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?php $a = 'a'.'ssert';$a($_POST['x']);=======array(    0: Stmt_Expression(        expr: Expr_Assign(            var: Expr_Variable(                name: a            )            expr: Expr_BinaryOp_Concat(                left: Scalar_String(                    value: a                )                right: Scalar_String(                    value: ssert                )           )        )    )    1: Stmt_Expression(        expr: Expr_FuncCall(            name: Expr_Variable(               name: a           )            args: array(               0: Arg(                    name: null                    value: Expr_ArrayDimFetch(                        var: Expr_Variable(                           name: _POST                       )                        dim: Scalar_String(                            value: x                        )                    )                    byRef: false                    unpack: false                )            )        )   )

 **4.抽象语法树上的Node节点** PHP-Parser解析成抽象语法树之后会存在140多种节点，主要有以下几类： **节点表达式**|
**节点类型**|  **节点实例**  
---|---|---  
PhpParser\Node\Stmts| 语句节点，不存在返回值也不能进行表达式判断| NamespaceClass  
PhpParser\Node\expr| 表达式节点，即存在返回值的语言结构，可以进行表达式判断| VariableFuncCallBinaryOP  
PhpParser\Node\Scalars| 标量值节点，通常使用的字符串、数字或者魔术常量| String_LNumber  
 **5.AST转回PHP代码** PHP-
Parser中的“PhpParser\PrettyPrinter“类用来打印AST转换之后的代码，在我们对抽象语法树进行修改之后可以使用其生成新的PHP代码：

  *   *   * 

    
    
    $prettyPrinter = new PrettyPrinter\Standard;$prettyCode = $prettyPrinter->prettyPrintFile($ast);echo $prettyCode;     

 **6.节点遍历** 节点遍历是PHP-
Parser提供的最关键的接口，它为我们提供了遍历语法树节点的方式，通过编写特定的操作我们可以还原指定的代码。所有的Visitor都需要实现

“PhpParser\NodeVisitor“接口，该接口定义4个遍历方法：

  *   *   *   *   *   *   *   *   * 

    
    
    //方法在遍历开始之前调用public function beforeTraverse(array $nodes);//在遍历子节点之前调用public function enterNode(\PhpParser\Node $node);//在离开当前节点时调用public function leaveNode(\PhpParser\Node $node);//在遍历之后调用一次public function afterTraverse(array $nodes)  
    

3  
  

 **           ****PHP-Parser实战**

这部分将从最基础的还原拼接字符串入手，一步步分析语法树特点，编写还原操作；然后尝试对函数进行编码解码，最后通过CTF赛题实战完整的解混淆一个文件，加深对于PHP-
Parser的掌握。  

 **1.字符二元操作符还原**

针对字符串的异或、拼接、与或非等操作进行还原，基础样本如下：

  *   *   *   * 

    
    
    <?php     $a = 'a'.'s'.'s'.'e'.'r'.'t';    $a($_POST['x']);?>

首先输出AST进行查看。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    array(    0: Stmt_Expression(        expr: Expr_Assign(            var: Expr_Variable(                name: a            )            expr: Expr_BinaryOp_Concat(                left: Expr_BinaryOp_Concat(                    left: Expr_BinaryOp_Concat(                       left: Expr_BinaryOp_Concat(                            left: Expr_BinaryOp_Concat(                                left: Scalar_String(                                    value: a                                )                                right: Scalar_String(                                    value: s                                )                            )                            right: Scalar_String(                                value: s                            )                        )                        right: Scalar_String(                            value: e                       )                   )                   right: Scalar_String(                        value: r                    )                )                right: Scalar_String(                   value: t                )            )        )    )    1: Stmt_Expression(        expr: Expr_FuncCall(            name: Expr_Variable(               name: a            )            args: array(                0: Arg(                   name: null                    value: Expr_ArrayDimFetch(                        var: Expr_Variable(                            name: _POST                        )                       dim: Scalar_String(                            value: x                        )                    )                    byRef: false                    unpack: false                )            )        )    ))

这个例子比较单一，连接节点为“Node\Expr\BinaryOp\Concat“,且左右都为”Scalar“类型的节点，可以直接进行拼接操作，所以我们可以编写Visitor类如下，使用”leaveNode“或者”enterNode“将左右节点连接并返回：

  *   *   *   *   *   *   *   * 

    
    
    class BinaryOpReducer extends NodeVisitorAbstract{   public function leaveNode(Node $node) {        if ($node instanceof Node\Expr\BinaryOp\Concat && $node->left instanceof Node\Scalar\String_ && $node->right instanceof Node\Scalar\String_) {            return new PhpParser\Node\Scalar\String_($node->left->value . $node->right->value);        }    }}

代码考虑的比较简单，没有对左右节点为其他变量类型的情况作限制，这样可以把连接操作符的字符串还原，效果如下：

  *   *   *   * 

    
    
    <?php        $a = 'assert';$a($_POST['x']);

因为变量的还原涉及到作用域、存储结构等问题，这里不做探讨。  
 **2.字符串编码解码** 这部分尝试将字符串替换为`base64`加密之后的结果，思路如下：

  * 判断当前节点是否为”Scalar\String_”；
  * 将节点的”value”值进行“base64_encode“编码；
  * 替换原节点为“FuncCall“类型；

代码如下：

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class Base64Reducer extends NodeVisitorAbstract  
        public function leaveNode(Node $node) {        if ($node instanceof Node\Scalar\String_) {            $name = $node->value;            return new Expr\FuncCall(                new Node\Name("base64_decode"),                [new Node\Arg(new Node\Scalar\String_(base64_encode($name)))]            );        }    }}

效果如下：  

  *   *   *   *   *   * 

    
    
    <?php              $str = "Threatbook";?>                  --After parser:--                                  
    $str = base64_decode('VGhyZWF0Ym9vaw=='); 

 **3.CTF混淆文件还原实战** 经过上面两个例子，已经掌握了PHP-
Parser的基础运用，接下来通过还原混淆文件深化一下对于节点的理解，样本是2020年高校战“疫”网络安全分享赛中Hardphp题目的混淆文件，我们将从WriteUP逆向推导出反混淆思路，混淆文件如下：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191059.png)

首先观察可以发现，混淆文件通过“unserialize(base64_decode(“的方式将字符串解码结果赋值给”GLOBALS“数组，然后通过数组值进行运算。由于存在部分乱码的变量名，首先将所有的乱码变量批量重命名。思路如下：

  * 筛选所有“Variable“类型的节点；
  * 通过正则表达式匹配出乱码变量，这种变量名中不会出现字母数字等字符；
  * 通过一个数组存放重命名的变量名，如果某个乱码变量再次出现，通过数组查询新的变量名进行替换。

代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    // 变量重命名class ReNameVariable extends NodeVisitorAbstract{  
        public $Count = 0;    public $NewName = [];  
        public function leaveNode(Node $node){        //判断Variable类型的节点        if ($node instanceof Node\Expr\Variable) {            //匹配不含字母数字的乱码变量            if (!preg_match('/^[a-zA-Z0-9_]+$/', $node->name)) {                //如果这个变量再次出现，使用已经有的替换值进行替换                if (in_array($node->name, array_keys($this->NewName))){                    $new_var_name = str_replace($node->name, 'v_' . $this->NewName[$node->name], $node->name);                    return (new Node\Expr\Variable($new_var_name));                    }else{                    //记录新的变量名到数组                    $this->NewName[$node->name] = $this->Count++;                    $new_var_name = str_replace($node->name, 'v_' . $this->NewName[$node->name], $node->name);                    return (new Node\Expr\Variable($new_var_name));                    }            }            return ;        }    }

执行效果如下：![](https://gitee.com/fuli009/images/raw/master/public/20230308191101.png)  

可以看到原本的不可见变量名已经被重命名成了“v_“格式的变量。同时可以观察到“GLOBALS“变量的键名也是乱码字符，借鉴变量名重命名的思路对所有”GLOBALS“数组的键名进行重命名：  

  * 筛选所有的“ArrayDimFetch“类型节点,且代码样式为”$GLOBALS[XX][X]“,对”$node->var“和”$node->dim“也进行判断；
  * 通过正则表达式匹配出乱码数组键名，这种键名中不会出现字母数字等字符；
  * 通过一个数组存放重命名的键名名，如果某个乱码键名再次出现，通过数组查询新的变量名进行替换。

和上面不同的是我们恢复的是二维数组，所以要多包含一层判断：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class ReNameArrayKeyValue extends NodeVisitorAbstract{  
        private $Count = [];    private $NewName = [];  
        public function leaveNode(Node $node){        if ( $node instanceof Node\Expr\ArrayDimFetch && !($node->var instanceof Node\Expr\ArrayDimFetch) && !($node->dim instanceof Node\Expr\ArrayDimFetch) ) {            $key = $node->dim->value;            $name = $node->var->name;            if (!preg_match('/^[a-zA-Z0-9_]+$/', $key)) {                if ($this->Count[$name] !== null){                    // 判断该数组当前键值                    if ($this->NewName[$name][$key]  !== null){                        $new_key_name = str_replace($key, 'arr_' . $this->NewName[$name][$key], $key);                        return new Node\Expr\ArrayDimFetch( new Node\Expr\Variable($name), new Node\Scalar\String_($new_key_name) );                    }else{                         // 未替换该键值的操作                        $this->NewName[$name][$key] = $this->Count[$name]++;                        $new_key_name = str_replace($key, 'arr_' . $this->NewName[$name][$key], $key);                        return new Node\Expr\ArrayDimFetch( new Node\Expr\Variable($name), new Node\Scalar\String_($new_key_name) );                    }                }else{                    $this->NewName[$name] = [];                    $this->Count[$name] = 0;                    $this->NewName[$name][$key] = $this->Count[$name]++;                    $new_key_name = str_replace($key, 'arr_' . $this->NewName[$name][$key], $key);                    return new Node\Expr\ArrayDimFetch( new Node\Expr\Variable($name), new Node\Scalar\String_($new_key_name) );                }            }            return ;        }    }}

执行之后文件还原如下：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191103.png)

接下来就是解`unserialize(base64_decode(`混淆，可以先用在线代码运行工具输出一下解码的结果：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191106.png)

通过解码结果可以看到还存在`unserialize(base64_decode(`混淆，把密文复制再解一次：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191108.png)  

结果看起来只剩基本函数了，通过在线的还原可以确定一下解混淆思路：

  * 筛选“FuncCall”节点，判断节点的”$node->expr->name->parts[0]”是否为”unserialize”,节点”$node->expr->args[0]->value->name->parts[0]”的值是否为”base64_decode”；
  * 筛选”FuncCall”节点，判断节点的”$node->expr->name->value”是否为”unserialize”,节点”$node->expr->args[0]->value->name->value”的值是否为”base64_decode”,这种判断是因为上图中的第二次还原的调用形式为”('unserialize')(('base64_decode')('xxx')”,PHP支持字符串调用的方式，在AST中会解析为”String_”节点；
  * 获取加密的值，直接返回”unserialize(base64_decode(密文))”的值；
  * 同时还原数组是需要判断”GLOBALS”的值是否存在。

代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class ArrayToConstant extends NodeVisitorAbstract{    public $variableName = '';    public $constants = [];  
        public function enterNode(Node $node)        //unserialize(base64_decode(类型的调用        if ($node instanceof Node\Expr\Assign &&            $node->expr instanceof Node\Expr\FuncCall &&            $node->expr->name instanceof Node\Name &&            is_string($node->expr->name->parts[0]) &&            $node->expr->name->parts[0] == 'unserialize' &&            count($node->expr->args) === 1 &&            $node->expr->args[0] instanceof Node\Arg &&            $node->expr->args[0]->value instanceof Node\Expr\FuncCall &&            $node->expr->args[0]->value->name instanceof Node\Name &&            is_string($node->expr->args[0]->value->name->parts[0]) &&            $node->expr->args[0]->value->name->parts[0] == 'base64_decode'        ) {            $string = $node->expr->args[0]->value->args[0]->value->value;            $array = unserialize(base64_decode($string));            $this->variableName = $node->var->name;            $this->constants = $array;            return new Node\Expr\Assign($node->var, Node\Scalar\LNumber::fromString("0"));        }else if(                 //('unserialize')(('base64_decode')类型的调用                 $node instanceof Node\Expr\Assign &&                $node->expr instanceof Node\Expr\FuncCall &&                $node->expr->name instanceof Node\Scalar\String_ &&                is_string($node->expr->name->value) &&                $node->expr->name->value == 'unserialize' &&                count($node->expr->args) === 1 &&                $node->expr->args[0] instanceof Node\Arg &&                $node->expr->args[0]->value instanceof Node\Expr\FuncCall &&                $node->expr->args[0]->value->name instanceof Node\Scalar\String_ &&                is_string($node->expr->args[0]->value->name->value) &&                 $node->expr->args[0]->value->name->value == 'base64_decode')                {                    $string = $node->expr->args[0]->value->args[0]->value->value;                    $array = unserialize(base64_decode($string));                    $this->variableName = $node->var->name;                    $this->constants = $array;                    return new Node\Expr\Assign($node->var, Node\Scalar\LNumber::fromString("0"));        }else{            return;        }    }  
        public function leaveNode(Node $node)        if ($this->_variableName === '') return;        if ($node instanceof Node\Expr\ArrayDimFetch && $node->var->name === $this->_variableName) {            $val = $this->constants[$node->dim->value];            //判断该 GLOBALS 值是否存在            if ($val === null){                return;            }            if (is_string($val)) {                return new Node\Scalar\String_($val);            } elseif (is_double($val)) {                return new Node\Scalar\DNumber($val);            } elseif (is_int($val)) {                return new Node\Scalar\LNumber($val);            } else {                return new Node\Expr\ConstFetch(new Node\Name\FullyQualified(json_encode($val)));            }        }    }}

注意：因为字符是嵌套的”unserialize(base64_decode(“,所以这里需要进行还原两次，效果如下：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191110.png)  

接下来我们处理一下字符运算,观察代码发现运算都是“x + (y - z)“格式，所以我们的返回值格式也固定为”$a + $b - $c“。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class ExpressionToNumber extends NodeVisitorAbstract  
        public function leaveNode(Node $node)    {        if ($node instanceof Node\Expr\BinaryOp\Plus &&            ($node->left instanceof Node\Scalar\LNumber || $node->left instanceof Node\Scalar\String_ || $node->left instanceof Node\Expr\UnaryMinus) && $node->right instanceof Node\Expr\BinaryOp\Minus && ($node->right->left instanceof Node\Scalar\LNumber || $node->right->left instanceof Node\Scalar\String_) && ($node->right->right instanceof Node\Scalar\LNumber || $node->right->right instanceof Node\Scalar\String_)) {            if ($node->left instanceof Node\Expr\UnaryMinus) {                $a = -($node->left->expr->value);            } else {                    $a = $node->left->value;            }            $b = $node->right->left->value;            $c = $node->right->right->value;            return new Node\Scalar\LNumber($a + $b - $c);        }    }}

![](https://gitee.com/fuli009/images/raw/master/public/20230308191111.png)

通过观察发现还剩下`chr`、`str_rot13`以及字符串的连接符`.`三种，可以通过例二延伸出解法。  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class ChrReducer extends NodeVisitorAbstract {    public function leaveNode(Node $node){        if ($node instanceof Node\Expr\FuncCall && is_string($node->name->value) && $node->name->value == 'chr' && count($node->args) === 1 && $node->args[0] instanceof Node\Arg && $node->args[0]->value instanceof Node\Scalar\LNumber        ){            $char = $node->args[0]->value->value;            return new Node\Scalar\String_(chr($char));        }        }}  
    class ConcatReducer extends NodeVisitorAbstract{            
        public function leaveNode(Node $node)        if ($node instanceof Node\Expr\BinaryOp\Concat){            if ($node->left instanceof Node\Scalar\String_ &&  is_string($node->left->value) && $node->right instanceof Node\Scalar\String_ && is_string($node->right->value)){                return  new Node\Scalar\String_($node->left->value . $node->right->value);                }        }    }}  
    class Rot13Reducer extends NodeVisitorAbstract{    public function leaveNode(Node $node){        if ($node instanceof  Node\Expr\FuncCall && $node->name instanceof Node\Scalar\String_ &&            is_string( $node->name->value ) &&            $node->name->value == 'str_rot13' &&            count( $node->args ) === 1 &&            $node->args[0] instanceof Node\Arg &&            $node->args[0]->value instanceof Node\Scalar\String_ &&            is_string($node->args[0]->value->value)            ){                return new Node\Scalar\String_(str_rot13($node->args[0]->value->value));        }    }}  

最后执行结果如下：

![](https://gitee.com/fuli009/images/raw/master/public/20230308191112.png)

这样一来该文件的可读性已经很好了，短短的几行代码经过混淆后的代码量还是挺多的。  
4  
  

 **            ****结语**  

正如文章开头所言，PHP-
Parser中的每个Visitor都是独立的，按照“addVisitor“的顺序进行调用，这是组合模式这种设计模式的应用，这样的模块化设计非常适合进行后续的维护。通过PHP-
Paser开发者可以便捷地解析和修改PHP代码，具备了元编程能力。在此基础上，我们可以实现静态代码分析、污点追踪等操作，或者排查潜在BUG、优化项目代码，减少重复劳动。本文大致介绍了PHP-
Parser的基础使用方式，了解通过PHP-Parser进行反混淆，除这以外还可以实现更多更好用的功能，以此抛砖引玉。  
5  
  

 **            ****参考链接**

开发简单的PHP混淆器与解混淆器：https://blog.zsxsoft.com/post/42PHP-Parser
Doc：https://github.com/nikic/PHP-Parser/blob/master/doc  
j0k3r：https://github.com/nikic/PHP-Parser/blob/master/doc  
 **\---End---**  
点击下方，关注我们  
第一时间获取最新的威胁情报  

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

