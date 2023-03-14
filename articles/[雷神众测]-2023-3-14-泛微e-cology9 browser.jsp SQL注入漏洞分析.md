#  泛微e-cology9 browser.jsp SQL注入漏洞分析

原创 F9Hѕυ  [ 雷神众测 ](javascript:void\(0\);)

**雷神众测** ![]()

微信号 bounty_team

功能介绍 雷神众测，专注于渗透测试技术及全球最新网络攻击技术的分析。

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20230314160520.png)

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160549.png)

引子

2023-02-23，某站发布了一个关于泛微e-cology9 SQL注入的漏洞通告。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160550.png)

如上图所示，根据其说明，受影响的版本范围是<=10.55版本。

另外，他们还提到该漏洞无权限要求，并不是后台洞。

  

![](https://gitee.com/fuli009/images/raw/master/public/20230314160549.png)

补丁包对比

E-COLOGY安全补丁下载网址如下：

https://www.weaver.com.cn/cs/securityDownload.html?src=cn

通过如下两个链接，下载该次漏洞的以及上一个版本的补丁包：

v10.55：

https://www.weaver.com.cn/cs/package/Ecology_security_20221014_v10.55.zip

v10.56：

https://www.weaver.com.cn/cs/package/Ecology_security_20230213_v10.56.zip

将两个补丁压缩包分别解压，然后使用IDEA工具对比差异。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160553.png)

这里对比看了很久，但是却没有看出有价值的内容。

嗯？先了解下web.xml文件中的内容。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160556.png)

在开头存在一个SecurityFilter的过滤器，SecurityFilter在初始化时会调用weaver.security.filter.SecurityMain中的initFilterBean方法初始化安全规则。而在weaver.security.rules.ruleImp包中的每个类差不多就是每次打的补丁，此包中的类将被重点关注。

所以，缩小范围去补丁包的WEB-
INF/myclasses/weaver/security/rules/ruleImp目录寻找。并且我们还可以根据文件时间戳，将2022年10月前的补丁文件都排除在外，继续过滤一遍。

    
    
    security/rules/ruleImp» stat -f %SB----%N *.class | grep -v "2021----" | grep -v -E '^[Jan|Mar|Apr|May|Jul|Aug|Sep].*2022' | wc -l  
         125

这样还剩下125个补丁文件，然后通过jadx这款反编译工具一次性打开这些补丁文件，然后搜索Xss(Validate
failed关键词，不断寻找，最终找到了一段SQL注入漏洞的补丁代码，下图所框疑似就是本次漏洞的位置。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160557.png)

上图所示的补丁代码，所处如下位置。

    
    
    $ Ecology_security_20230213_v10.56 » fd SecurityRuleForMobileBrowser  
    WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleForMobileBrowser.class

然而在v10.55补丁包中也同样发现该文件所在，并且是一摸一样的。

    
    
    $ Ecology_security_20221014_v10.55 » fd SecurityRuleForMobileBrowser  
    WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleForMobileBrowser.class
    
    
    $ patch » md5 Ecology_security_20230213_v10.56/WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleForMobileBrowser.class Ecology_security_20221014_v10.55/WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleForMobileBrowser.class | awk -F "=" '{print $2}'  
     391d53bb28cffa1bf6974e21eac16b7d  
     391d53bb28cffa1bf6974e21eac16b7d

查看这两个文件的时间戳，发现最后修改时间也是一致，都是2022年12月8日。

    
    
    $ patch » stat -f %SB Ecology_security_20230213_v10.56/WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleForMobileBrowser.class Ecology_security_20221014_v10.55/WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleForMobileBrowser.class  
    Dec  8 19:55:26 2022  
    Dec  8 19:55:26 2022

但是在下载v10.55补丁包的时候，通过其下载链接，可以得知此版本补丁包的发布日期是2022年10月14日。是早于其中的SecurityRuleForMobileBrowser.class文件的最后修改时间的。

    
    
    https://www.weaver.com.cn/cs/package/Ecology_security_20221014_v10.55.zip

在通过关于这个漏洞的通告中的时间线中，大致可以猜测出来，在2022年9月，正值演练期间收到该漏洞，在2022月12月上报给监管单位，监管单位应该是收到了该漏洞就立马通知给厂商，在2022年12月8日厂商就已经开发出本次SQL注入漏洞的补丁代码，也就是SecurityRuleForMobileBrowser.class文件中的内容。但厂商在开发出该漏洞的补丁代码后并未立即发布v10.56补丁包，而是先将其更新至v10.55补丁包中了。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160600.png)

这也就是为什么用IDEA对比v10.55和v10.56补丁包，却一无所获的原因。

那么既然如此，拿v10.54补丁包与v10.56补丁包对比呢？v10.54补丁包下载地址如下：

    
    
    https://www.weaver.com.cn/cs/package/Ecology_security_20220805_v10.54.zip

如下图所示，确实对比出了该文件存在于V10.56中，而不存在于V10.54中。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160601.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230314160549.png)

确定漏洞位置

虽然通过如上的补丁包对比分析找到了一个疑似的漏洞路径，但是未必就能肯定这是真正的漏洞位置。

我们先来简单看看/mobile/plugin/browser.jsp的内容。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160609.png)

参数很多，继续往下看，发现一个isDis参数及其判断语句。

    
    
    boolean isDis = "1".equals(Util.null2String(request.getParameter("isDis"))) ? true : false;  
    if (!isDis) {  
    	request.getRequestDispatcher("/mobile/plugin/dialog.jsp").forward(request, response);  
    	return;  
    }

如果isDis参数值不为1的话，则进入if条件语句之中，RequestDispatcher的作用是将请求分配给另一个资源，后面使用的是forward方法，此处就是将请求转发到/mobile/plugin/dialog.jsp处理。那么就看看/mobile/plugin/dialog.jsp的内容。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160611.png)

发现开头的HrmUserVarify.getUser，该方法部分代码如下：

![](https://gitee.com/fuli009/images/raw/master/public/20230314160616.png)

可以看出此处有个登录判断，根据我们的情况肯定会返回null到dialog.jsp就直接返回空了。死路一条，弃之。

回到上面，现在可以确定的是该参数是必须需要存在的，且参数值还得必须为1。不妨构造一个请求发送看看。

    
    
    POST /mobile/plugin/browser.jsp HTTP/1.1  
    Host:   
    Accept-Encoding: gzip, deflate  
    Accept: */*  
    Accept-Language: en-US;q=0.9,en;q=0.8  
    Connection: close  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 7  
      
    isDis=1

![](https://gitee.com/fuli009/images/raw/master/public/20230314160617.png)

与网站放出的测试图相比，是不是就对上了。那么也就能够确定漏洞的路径就是：/mobile/plugin/browser.jsp。

在使用BurpSuite Intruder多个不同的目标时，发现除了200的状态码，还有很多404的，这种情况我们最后再说。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160621.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230314160549.png)

补丁代码分析

通过前面两段内容的相互印证，可以确定本次SQL注入的漏洞补丁代码就是SecurityRuleForMobileBrowser.class文件的内容。

完整补丁代码内容如下。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160623.png)

对该段补丁代码作简单分析。从第5行的第一个if条件语句开始，判断如果../、\、十六进制的00都不存在于URI中，则进入第6行下一个if条件语句判断，如果/mobile/、/plugin/、/browser.jsp都存在于URI中，那么获取keyword参数值；接着进入到第9行try语句，首先对keyword参数值进行了一次URL解码，并判断其中是否有恶意SQL注入payload
'，如果有的话，则进行拦截、拉黑IP，返回false；紧接着对keyword参数值进行二次URL解码，并将二次URL解码后的值与第一次URL解码的值将比较，如果不一致也会进行拦截、拉黑IP，返回false，此处判断是为了防止多层URL编码Bypass的，即第一次URL解码的结果必须是最终的的解码结果。

通过分析可以得知存在注入的参数就是/mobile/plugin/browser.jsp中的keyword参数。

  

![](https://gitee.com/fuli009/images/raw/master/public/20230314160549.png)

SQL注入分析

现在继续关注/mobile/plugin/browser.jsp中的内容。刚刚说到对isDis的判断，继续往下看。

    
    
    String f_weaver_belongto_userid=Util.null2String(request.getParameter("f_weaver_belongto_userid"));//需要增加的代码  
    String f_weaver_belongto_usertype=Util.null2String(request.getParameter("f_weaver_belongto_usertype"));//需要增加的代码  
    User user  = HrmUserVarify.getUser(request, response, f_weaver_belongto_userid, f_weaver_belongto_usertype) ;//需要增加的代码  
      
    BrowserAction braction = new BrowserAction(user, browserTypeId, pageNo, pageSize);

跟进HrmUserVarify.getUser，代码如下，这里肯定会返回null。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160625.png)

起初这个地方让我感到很迷惑，误以为这个鉴权会被用到，实际并不会用到，这里返回的null作为browser.jsp中的user变量的值，但是在browser.jsp中并未对user做检查。如果需要达到鉴权的效果，那么正确的写法应该是增加如下代码片段：

    
    
    if(user == null)  return ;

如下图所示，SearchSubDept.jsp正是采用的这种写法。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160627.png)

继续往下，设置了很多参数值，但不过大部分参数都不是必须的。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160632.png)

最后到braction.getBrowserData()方法，这个方法位于classbean/weaver/mobile/webservices/common/BrowserAction.class文件。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160638.png)

最开始有对browserTypeId参数值进行判断，以及很多list开头的方法，接着还对method参数值进行判断，根据不同的值执行不同的list开头的方法，那么注入很大可能就存在某个list开头的方法之中。

先简单尝试注入一下，请求如下：

    
    
    POST /mobile/plugin/browser.jsp HTTP/1.1  
    Host:   
    Accept-Encoding: gzip, deflate  
    Accept: */*  
    Accept-Language: en  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36  
    Connection: close  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 54  
      
    isDis=1&browserTypeId=160&keyword=a%' union select 1,'

![](https://gitee.com/fuli009/images/raw/master/public/20230314160643.png)

可以发现一些与SQL注入相关的敏感关键词（'、select）均被全角化了。那尝试URL编码一下，一层URL编码的请求如下，服务器直接返回500了。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160646.png)

双层URL编码试试，如下图所示，服务器端依旧做了两次URL解码，然后发现'，将其转换成全角字符了，但不过select关键词却没有被全角化。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160648.png)

继续三层URL编码，通过下图可以发现，经过三层编码后的字符串被URL解码两次后顺利到达BrowserAction.getBrowserData()，而在每一个list开头的方法中都有一次URL解码操作，那么就能确保我们的SQL注入payload顺利传递到SQL查询语句中。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160650.png)

下面依次对各个list开头的方法进行审计，最后发现当browserTypeId等于269时，执行的listRemindType()方法中存在一个有回显的SQL注入漏洞。

    
    
    public void listRemindType() {  
        try {  
            new MeetingBrowser();  
            this.keyword = URLDecoder.decode(this.keyword, "UTF-8");  
            if (this.pageInfo.getPageNo() > 0) {  
                this.pageInfo.setPageNo(this.pageInfo.getPageNo());  
            }  
      
            RecordSet var1 = new RecordSet();  
            String var2 = " select count(0) as count from meeting_remind_type t1  where isuse=1 ";  
            if (StringUtils.isNotEmpty(this.keyword)) {  
                var2 = var2 + " and name like '%" + this.keyword + "%'";  
            }  
      
            var1.executeSql(var2);  
            int var3 = 0;  
            if (var1.next()) {  
                var3 = Util.getIntValue(var1.getString("count"), 0);  
            }  
      
            this.pageInfo.setTotalCount(var3);  
            String var4 = " from  meeting_remind_type t1  ";  
            String var5 = "t1.id as id,t1.name as name";  
            String var6 = " where isuse=1 ";  
            if (StringUtils.isNotEmpty(this.keyword)) {  
                var6 = var6 + " and t1.name like '%" + this.keyword + "%' ";  
            }  
      
            String var7 = "t1.id";  
            String var8 = "t1.id";  
            log.info("select " + var5 + var4 + " where " + var6);  
            this.pageInfo.setResult(this.getLimitPageData(var5, var4, var6, var7, var8, 1, this.pageInfo.getPageNo(), this.pageInfo.getPageSize(), 3));  
        } catch (Exception var9) {  
            var9.printStackTrace();  
        }  
      
    }

看到这里就可以直接构造注入payload了，最终注入的效果如下所示。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160653.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230314160657.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230314160549.png)

 认证绕过分析

在上面有埋下一个坑，就是在使用BurpSuite
Intruder请求多个不同的目标的/mobile/plugin/browser.jsp时，发现有很多返回404状态码的站。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160702.png)

我们需要再次对比v10.54和v10.56的补丁包。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160704.png)

找到SecurityRuleMobile29.class这么一个补丁文件，再次对比，首先可以发现在v10.54补丁包中的
SecurityRuleMobile29.class文件的最后修改时间是2020年9月10日，那么如果ecology没有打过这个补丁则无需考虑绕过的情况，/mobile/plugin/browser.jsp路径可以被直接访问，这也就是为什么有一些站直接访问该路径不会出现404的原因。

    
    
    # stat -f %SB Ecology_security_20220805_v10.54/WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleMobile29.class Ecology_security_20230213_v10.56/WEB-INF/myclasses/weaver/security/rules/ruleImp/SecurityRuleMobile29.class  
    Sep 10 09:51:04 2020  
    Dec  7 20:26:20 2022

![](https://gitee.com/fuli009/images/raw/master/public/20230314160708.png)

未更新该补丁之前的validate方法内容如下图。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160710.png)

我们先从153行的else语句开始看起。毫无疑问，当请求路径中存在/mobilemode/或/mobile/或/cpt/其中一个，并且请求路径的结尾是.jsp时，顺利进入到154行if分支。

    
    
    else {  
        if (path.indexOf("/mobilemode/") != -1 || path.indexOf("/mobile/") != -1 || path.indexOf("/cpt/") != -1 && path.endsWith(".jsp")) {  
            List<String> mobileNoLoginUrlList = (List)sc.getRule().get("mobile-no-login-urls");  
            if (mobileNoLoginUrlList != null && !mobileNoLoginUrlList.isEmpty()) {  
                Iterator var7 = mobileNoLoginUrlList.iterator();  
      
                while(var7.hasNext()) {  
                    String url = (String)var7.next();  
                    if (path.indexOf(url) != -1) {  
                        return true;  
                    }  
                }  
            }  
      
            List<String> mobileNeedLoginUrlList = (List)sc.getRule().get("mobile-need-login-urls");  
            if (mobileNeedLoginUrlList != null && !mobileNeedLoginUrlList.isEmpty()) {  
                Iterator var8 = mobileNeedLoginUrlList.iterator();  
      
                while(var8.hasNext()) {  
                    String url = (String)var8.next();  
                    if (path.indexOf(url) != -1) {  
                        boolean isLogin = this.isLogin(req, res);  
                        if (!isLogin) {  
                            sc.writeLog(">>>>Xss(Validate failed[Not Login]) validateClass=weaver.security.rules.SecurityRuleMobile29  path=" + req.getRequestURI() + " security validate failed!  source ip:" + ThreadVarManager.getIp());  
                            return false;  
                        }  
                    }  
                }  
            }  
        }  
      
        return true;  
    }

接下来，mobileNoLoginUrlList这个列表中的路径意味着无需登录即可直接访问，如果请求的路径在该列表中，则会返回true。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160712.png)

mobileNeedLoginUrlList列表顾名思义，当请求的路径在该列表中，则是需要登录才能访问的，否则就会返回false。而/mobile/plugin/browser.jsp恰巧在其之中。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160715.png)

当请求的路径既不属于mobileNoLoginUrlList，也不属于mobileNeedLoginUrlList，也是会返回true的。

那么我们继续看validate方法中新增的补丁代码片段：

![](https://gitee.com/fuli009/images/raw/master/public/20230314160718.png)

    
    
    else if (StringUtil.matches(path, "\\s") && StringUtil.matches(path, "/\\s+/")) {  
      sc.writeLog(">>>>Xss(Validate failed[invalidate url]) validateClass=weaver.security.rules.SecurityRuleMobile29  path=" + req.getRequestURL() + " security validate failed!  source ip:" + ThreadVarManager.getIp());  
      return false;  
    }

这里做了一个正则匹配，如果请求路径中存在空白字符，就会触发安全补丁的警告，并返回false。

在此之前，需要留意super.path方法，这个是父类ParentRule中的方法。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160720.png)

path方法会对请求路径中出现的一些特殊字符如;、//，那么则会做正则replace。并且最后还会去除路径中出现的空白字符。最后返回path.toLowerCase()。

但是这里过滤的并不全，不然就不会出现补丁代码中的又一遍的检查了。

    
    
    StringUtil.matches(path, "\\s") && StringUtil.matches(path, "/\\s+/")

所以这里必定是存在绕过的，别忘了path方法中开始会使用uriDecode方法对请求路径做URL解码的哦。

最后梳理下，原始的请求路径需要先经过URL解码，解码后其中不要存在有;、//字符，然后还要经过一遍去空白字符操作，再然后就会返回最终的路径，最后的路径能到达/mobile/plugin/browser.jsp。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160723.png)

成功绕过后，就能对未打这次最新补丁的ecology进行SQL注入了。

![](https://gitee.com/fuli009/images/raw/master/public/20230314160724.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230314160549.png)

修复建议

  

目前厂商已经发布了升级补丁以修复这个问题，请到厂商的主页下载：

https://www.weaver.com.cn/cs/securityDownload.html?src=cn

  

  

 **安恒信息**

✦

杭州亚运会网络安全服务官方合作伙伴

成都大运会网络信息安全类官方赞助商

武汉军运会、北京一带一路峰会

青岛上合峰会、上海进博会

厦门金砖峰会、G20杭州峰会

支撑单位北京奥运会等近百场国家级

重大活动网络安保支撑单位

![](https://gitee.com/fuli009/images/raw/master/public/20230314160727.png)

END

![](https://gitee.com/fuli009/images/raw/master/public/20230314160728.png)![](https://gitee.com/fuli009/images/raw/master/public/20230314160729.png)![](https://gitee.com/fuli009/images/raw/master/public/20230314160730.png)

 **长按识别二维码关注我**

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

