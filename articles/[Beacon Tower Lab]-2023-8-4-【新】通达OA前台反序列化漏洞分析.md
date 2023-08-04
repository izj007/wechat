#  【新】通达OA前台反序列化漏洞分析

原创 烽火台实验室  [ Beacon Tower Lab ](javascript:void\(0\);)

**Beacon Tower Lab** ![]()

微信号 WebRAY_BTL

功能介绍 "海上千烽火，沙中百战场"，烽火台实验室将为您持续输出前沿的安全攻防技术

____

___发表于_

收录于合集

**0x01 前言**

  

  

注：本文仅以安全研究为目的，分享对该漏洞的挖掘过程，文中涉及的所有漏洞均已报送给国家单位，请勿用做非法用途。

  

通达OA作为历史上出现漏洞较多的OA，在经过多轮的迭代之后已经很少前台的RCE漏洞了。一般来说通达OA是通过auth.inc.php文件来进行鉴权，如图1.1所示。整个通达全部的代码看下来很少有未鉴权的代码。  

![]()

图1.1 通达中一般文件的鉴权方式

  

在通达中有一个模块

/general/appbuilder/web/index.php采用了yii框架实现，其鉴权逻辑与其他模块存在显著差异。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    $url = $_SERVER["REQUEST_URI"];    $strurl = substr($url, 0, strpos($url, "?"));  
        if (strpos($strurl, "/portal/") !== false) {      if (strpos($strurl, "/gateway/") === false) {        header("Location:/index.php");        sess_close();        exit();      }      else if (strpos($strurl, "/gateway/saveportal") !== false) {        header("Location:/index.php");        sess_close();        exit();      }      else if (strpos($url, "edit") !== false) {        header("Location:/index.php");        sess_close();        exit();      }    }    else if (strpos($url, "/appdata/doprint") !== false) {      $_GET["csrf"] = urldecode($_GET["csrf"]);      $b_check_csrf = false;      if (!empty($_GET["csrf"]) && preg_match("/^\{([0-9A-Z]|-){36}\}$/", $_GET["csrf"])) {        $s_tmp = __DIR__ . "/../../../../logs/appbuilder/logs";        $s_tmp .= "/" . $_GET["csrf"];  
            if (file_exists($s_tmp)) {          $b_check_csrf = true;          $b_dir_priv = true;        }      }  
          if (!$b_check_csrf) {        header("Location:/index.php");        sess_close();        exit();      }    }    else {      header("Location:/index.php");      sess_close();      exit();    }

  

从上面的代码可以看出，如果访问的目标地址是/general/appbuilder/web/portal/gateway/?，则不需要授权就能访问对应的接口。

  

再来看一下通达中使用的yii版本，如图1.2所示。Yii2 < 2.0.38是存在反序列化利用链的，网上已经有很多分析文章，感兴趣的小伙伴可以关注。

https://www.anquanke.com/post/id/254429

  

![]()

图1.2通达OA中的yii版本  

  

虽然有了利用链，但如何利用一直是个难题。本文的目的是寻找反序列化利用点并构造反序列化利用链达到RCE效果。

  

 **0x02 反序列化点**  

  

  

在appbuilder模块中多数情况下会加载视图views/layouts/main.php。视图中会调用csrfMetaTags方法。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?php  
    $this->beginPage();echo "<!DOCTYPE html>\n<html lang=\"";echo Yii::$app->language;echo "\">\n<head>\n    <meta charset=\"";echo Yii::$app->charset;echo "\">\n    <meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">\n    ";echo yii\helpers\Html::csrfMetaTags();echo "    <title>";echo yii\helpers\Html::encode($this->title);echo "</title>\n\t<link rel=\"stylesheet\" type=\"text/css\" href=\"";echo Yii::$app->params["MYOA_STATIC_SERVER"];echo "/static/theme/";echo Yii::$app->params["LOGIN_THEME"] == "" ? "1" : Yii::$app->params["LOGIN_THEME"];echo "/style.css\" />\n    <link href=\"/static/js/bootstrap/css/bootstrap.css\" rel=\"stylesheet\">\n<!--    <link href=\"/module/appbuilder/css/bootstrap.css\" rel=\"stylesheet\">-->\n    <link href=\"/module/appbuilder/css/site.css\" rel=\"stylesheet\"></head>\n\t<style>\n\ta.btn.btn-danger {\n\t\tcolor: #fff;\n\t}\n\t</style>\n    ";$this->head();echo "</head>\n<body class=\"bodycolor\">\n";$this->beginBody();echo "\n<div><!--class=\"wrap\"-->\n    ";

  

在yii框架中存在yii\helpers\Html::csrfMetaTags()方法，该方法的主要作用时用于生成csrf校验需要的meta标签。

  *   *   *   *   *   *   *   *   *   * 

    
    
     public static function csrfMetaTags(){        $request = Yii::$app->getRequest();        if ($request instanceof Request && $request->enableCsrfValidation) {            return static::tag('meta', '', ['name' => 'csrf-param', 'content' => $request->csrfParam]) . "\n"                . static::tag('meta', '', ['name' => 'csrf-token', 'content' => $request->getCsrfToken()]) . "\n";        }  
            return '';    }

  

在方法中调用了$request->getCsrfToken()，跟踪该方法。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public function getCsrfToken($regenerate = false){        if ($this->_csrfToken === null || $regenerate) {            $token = $this->loadCsrfToken();            if ($regenerate || empty($token)) {                $token = $this->generateCsrfToken();            }            $this->_csrfToken = Yii::$app->security->maskToken($token);        }  
            return $this->_csrfToken;    }   public function getCsrfToken($regenerate = false){        if ($this->_csrfToken === null || $regenerate) {            $token = $this->loadCsrfToken();            if ($regenerate || empty($token)) {                $token = $this->generateCsrfToken();            }            $this->_csrfToken = Yii::$app->security->maskToken($token);        }  
            return $this->_csrfToken;    }

  

在该方法中继续调用了loadCsrfToken方法，跟踪该方法。

  *   *   *   *   *   *   *   * 

    
    
    protected function loadCsrfToken(){        if ($this->enableCsrfCookie) {            return $this->getCookies()->getValue($this->csrfParam);        }  
            return Yii::$app->getSession()->get($this->csrfParam);    }

  

继续跟踪getCookies方法。

  *   *   *   *   *   *   *   *   * 

    
    
    public function getCookies(){    if ($this->_cookies === null) {        $this->_cookies = new CookieCollection($this->loadCookies(), [            'readOnly' => true,        ]);    }    return $this->_cookies;}

  

继续跟踪loadCookies方法，在这个方法中会调用unserialize方法对传入的Cookie的值进行反序列化。这也就会造成反序列化漏洞。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    protected function loadCookies(){        $cookies = [];        if ($this->enableCookieValidation) {            if ($this->cookieValidationKey == '') {                throw new InvalidConfigException(get_class($this) . '::cookieValidationKey must be configured with a secret key.');            }            foreach ($_COOKIE as $name => $value) {                if (!is_string($value)) {                    continue;                }                $data = Yii::$app->getSecurity()->validateData($value, $this->cookieValidationKey);                if ($data === false) {                    continue;                }                if (defined('PHP_VERSION_ID') && PHP_VERSION_ID >= 70000) {                    $data = @unserialize($data, ['allowed_classes' => false]);                } else {                    $data = @unserialize($data); //这里是反序列化点                }                if (is_array($data) && isset($data[0], $data[1]) && $data[0] === $name) {                    $cookies[$name] = Yii::createObject([                        'class' => 'yii\web\Cookie',                        'name' => $name,                        'value' => $data[1],                        'expire' => null,                    ]);                }            }        } else {            foreach ($_COOKIE as $name => $value) {                $cookies[$name] = Yii::createObject([                    'class' => 'yii\web\Cookie',                    'name' => $name,                    'value' => $value,                    'expire' => null,                ]);            }        }  
            return $cookies;    }

  

 **0x03 绕过限制**  

  

  

在上面的方法中会对传入的Cookie值进行签名校验，校验的方法是validateData，其中$this->cookieValidationKey是签名的key。

  * 

    
    
    Yii::$app->getSecurity()->validateData($value, $this->cookieValidationKey)

  

在validateData方法中会通过hash_hmac对传入的key和value进行签名校验。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public function validateData($data, $key, $rawHash = false){    $test = @hash_hmac($this->macHash, '', '', $rawHash);    if (!$test) {        throw new InvalidConfigException('Failed to generate HMAC with hash algorithm: ' . $this->macHash);    }    $hashLength = StringHelper::byteLength($test);    if (StringHelper::byteLength($data) >= $hashLength) {        $hash = StringHelper::byteSubstr($data, 0, $hashLength);        $pureData = StringHelper::byteSubstr($data, $hashLength, null);        $calculatedHash = hash_hmac($this->macHash, $pureData, $key, $rawHash);        if ($this->compareString($hash, $calculatedHash)) {            return $pureData;        }    }    return false;}

  

由于通达OA中的$this->cookieValidationKey来自于配置文件general/appbuilder/config/web.php。其中值是固定的。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <?php  
    $params = require __DIR__ . "/params.php";$config = array(  "id"          => "appbuilder",  "basePath"    => dirname(__DIR__),  "bootstrap"   => array("log"),  "charset"     => "GB2312",  "language"    => "zh-CN",  "runtimePath" => "@app/../../../logs/appbuilder",  "components"  => array(    "request"      => array("cookieValidationKey" => "tdide2"), //这是固定的密钥tdide2    "cache"        => array("class" => "app\\td\base\TDRedisCache"),    "redis"        => array("class" => "yii\\redis\Connection", "hostname" => $MYOA_REDIS_SERVERS[0]["host"], "port" => $MYOA_REDIS_SERVERS[0]["port"], "database" => $MYOA_REDIS_DB_ID + 1, "password" => $MYOA_REDIS_PASS),    "errorHandler" => array("errorAction" => "site/error"),    "log"          => array(      "traceLevel" => 1,      "targets"    => array(        array(          "class"  => "yii\log\FileTarget",          "levels" => array("error")          )        )

  

另外通达OA有全局的addslashes过滤，包括Cookie中的值。由于PHP反序列化中有大量的双引号，如果直接通过Cookie传递则会因为双引号被转义而失败。但是令人开心的是通达在进行全局addslashes的时候对部分值进行了例外排查。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    if (0 < count($_COOKIE)) {  foreach ($_COOKIE as $s_key => $s_value ) {    if ((substr($s_key, 0, 7) == "_SERVER") || (substr($s_key, 0, 8) == "_SESSION") || (substr($s_key, 0, 7) == "_COOKIE") || (substr($s_key, 0, 4) == "_GET") || (substr($s_key, 0, 5) == "_POST") || (substr($s_key, 0, 6) == "_FILES")) {      continue;    }  
        if (!is_array($s_value)) {      $_COOKIE[$s_key] = addslashes(strip_tags($s_value));    }  
        $s_key = $_COOKIE[$s_key];  }  
      reset($_COOKIE);}

  

如果Cookie中字段名称的前面几位字符为_GET这种，则不进行addslashes操作。这也就给漏洞利用提供了便利。

  

 **0x04结论**  

  

  

反序列化利用链是直接采用yii中已经公开的反序列化链，本文作者调好的poc（实际是exp）如下所示， **完整的poc可从ddpoc平台查看 。**

 **  
**

 **https://www.ddpoc.com/poc/DVB-2023-4705.html**

  

![]()

使用该poc之后可以会在网站根目录生成哥斯拉的webshell。

  * 

    
    
    在根目录生成文件/logon.php 111/111 哥斯拉

本漏洞已向相关单位报送并已推出补丁，可适用于通达11.X全版本。对应12系列未做过多测试，本文提到的所有漏洞在最新版的通达12.4中已修复，使用此漏洞造成的任何攻击影响均与本文作者无关。

  

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

