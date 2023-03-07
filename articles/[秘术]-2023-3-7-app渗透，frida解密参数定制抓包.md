#  app渗透，frida解密参数定制抓包

原创 Yaoヽ药药  [ 秘术 ](javascript:void\(0\);)

**秘术** ![]()

微信号 gh_0f1ee576e0ac

功能介绍 法于阴阳，和于术数。

____

___发表于_

收录于合集

使用小黄鸟进行抓包发现所有接口都是通过paras进行传递  

paras参数值是被加密了的  
  

需要解密后才能进行正常抓包  

![](https://gitee.com/fuli009/images/raw/master/public/20230307082000.png)

  

通过mt对apk进行反编译发现是360加固，这是使用万能脱壳法  

![](https://gitee.com/fuli009/images/raw/master/public/20230307082022.png)

通过对dex代码反编译进行分析，然后使用瞎几把搜索乱下段定位法  

定位到了以下代码块  

  *   *   *   * 

    
    
    HashMap v4 = new HashMap();v4.put("mobile", StringsKt.trim(arg3.getBinding().etPhone.getText().toString()).toString());v4.put("password", StringsKt.trim(arg3.getBinding().etLoginPwd.getText().toString()).toString());String v4_1 = GsonUtil.INSTANCE.mapToJsonStrObjEncryption(v4);

  

![](https://gitee.com/fuli009/images/raw/master/public/20230307082023.png)

通过分析发现所有接口都是先定义了hashmap  

然后所有参数传递给hashmap  

然后调用mapToJsonStrObjEncryption  
通过函数定义的命名就能知道是将对象加密 然后返回一段json  

![](https://gitee.com/fuli009/images/raw/master/public/20230307082024.png)

这里接着往下跟

发现是调用的encryptionData

此时的hashmap已经是json字符串了

![](https://gitee.com/fuli009/images/raw/master/public/20230307082025.png)  

![](https://gitee.com/fuli009/images/raw/master/public/20230307082026.png)

同类下发现decryptData

那么这里 encryptionData decryptData便是关键函数  

准备一个真机进行调试，先将frida-server push到手机然后执行起来  

  *   *   * 

    
    
    adb push /data/local/tmp frida-serverchmod 777 /data/local/tmp/frida-server./data/local/tmp/frida-serve

写好frida hook关键函数并且log出参数  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    function main(){    Java.perform(function(){          var LoginActivity= Java.use("com.xxxxxxxx.utils.DecryptUtils");        LoginActivity.encryptionData.implementation=function(str){            console.log("待加密的数据:" + str);            console.log("解密后的数据" + this.encryptionData(str));            return this.encryptionData(str);        }        LoginActivity.decryptData.implementation=function(str){            console.log("待解密的数据:" + str);            console.log("解密后的数据" + this.decryptData(str));            return this.decryptData(str);        }            });}  
    setTimeout(() => {    main()});

然后将代码注入到指定进程  

  * 

    
    
    frida -U -F 包名 -l hook.js

![](https://gitee.com/fuli009/images/raw/master/public/20230307082028.png)

此时，便明文了，可以进行抓包了

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    var LoginActivity= Java.use("com.xxxxxx.utils.DecryptUtils");LoginActivity.encryptionData.implementation=function(str){            console.log("待加密的数据:" + str);            if (str.indexOf("被改的内容") > -1)            {                str = str.replace("被改的内容","待改的内容");                console.log("改改改改改包:" + str);            }            return this.encryptionData(str);        }

![](https://gitee.com/fuli009/images/raw/master/public/20230307082029.png)

通过抓管理员userid，然后到个人页面刷新，将自己的userid改成管理员的便越权了，可以进行改包了  

![](https://gitee.com/fuli009/images/raw/master/public/20230307082031.png)

  

发现阿里云oss  

![](https://gitee.com/fuli009/images/raw/master/public/20230307082034.png)

成功接管！！！  

  

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

