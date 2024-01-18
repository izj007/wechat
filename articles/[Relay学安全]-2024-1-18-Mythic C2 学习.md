#  Mythic C2 学习

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

**Relay学安全** ![]()

微信号 gh_8d57319ec39c

功能介绍
这是一个纯分享技术的公众号，只想做安全圈的一股清流，不会发任何广告，不会接受任何广告，只会分享纯技术文章，欢迎各行各业的小伙伴关注。让我们一起提升技术。

____

___发表于_

##### 安装

从github上拉下来。

  * 

    
    
    git clone https://github.com/its-a-feature/Mythic

更新kali:

  *   * 

    
    
    apt install updateapt install upgrade //更新会很慢 耐心等待 建议挂上梯子之后再去更新

记住一定要更新

  *   * 

    
    
    cd Mythicsudo ./install-docker-kali.sh

  * 

    
    
    sudo make

配置安装:

  * 

    
    
    sudo ./mythic-cli start 启动

  * 

    
    
    sudo ./mythic-cli stop 停止

查看状态:

  * 

    
    
    sudo ./mythic-cli status

![]()

需要注意的是在我们使用start之后会生成一个.env文件，记住是.env，前面没有什么名称.env

这是我们的配置文件。

这里需要注意的是:

  *   *   *   * 

    
    
    MYTHIC_SERVER_PORT 表示你Mythic服务器上开放的端口。NGINX_PORT是由Nginx开放的充当所有其他服务的反向代理。NGINX_PORT表示你连接到web用户页面的端口。这是唯一的端口。ALLOWED_IP_BLOCKS表示对Mythic login页面的访问，这里可以设置一个ip段也可以直接设置一个固定的IP,比如192.168.0.1/24

如果你要运行Mythic服务器注意以外的的服务，则需要确保RABBITMQ_BIND_LOCALHOST_ONLY
和MYTHIC_SERVER_BIND_LOCALHOST_ONLY 均设置为false。

![]()

当使用start启动之后，会生成账号和密码，账号是mythic_admin，密码是随机的，我们直接在.env配置文件中的MYTHIC_ADMIN_PASSWORD来找到随机的密码。

![]()

现在我们就可以直接访问7443端口了。

![]()

会发现这里生成payload没有可使用的payload。

![]()

这是因为Mythic默认不附带任何有效载荷或C2配置文件我们可以使用如下命令进行添加。

  *   *   *   * 

    
    
    安装payload:sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo安装C2配置文件sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http

如下是它的一些payload相关的，我们完全可以使用如上命令来进行安装。

如下这些项目均可以使用。

  * 

    
    
    https://github.com/MythicAgents

![]()

直接把github的地址复制下来安装即可。

![]()

##### 使用

安装成功我们点击

C2 Profile And Payload Types。可以看到这些C2以及payload支持的操作系统和项目地址。

![]()

我们可以点击最右边的Documentation。

![]()

这里会对这个项目进行介绍以及使用。可以看到它支持的一些命令。

![]()

现在我们来生成一个payload上线。

选择Create Payload

![]()

选择Windows 当然你也可以选择Linux。

![]()

这里有两种方式一种是直接生成EXE一种是生成shellcode，如果要做免杀的话可以生成shellcode，我这里先选择EXE。

![]()

这里是你要配置的一些命令：这里点击>>>即可。

![]()

下一步:

![]()

最后点击CREATE PAYLOAD生成即可。

![]()

生成之后点击最上面菜单栏的第三个也就是payloads。

这里就是我们生成的马，我们点击download下载下来 放到目标机器上运行即可。

![]()

运行之后可以看到就已经上线了。点击Active Callbacks

这里可以看到有目标的IP，用户名，以及HostName，还有PID等等，感觉跟Cobalt Strike差不多。

![]()

如果我们想执行命令的话直接点击如下即可

![]()

我们都知道在Cobalt Strike中执行命令的话都是通过shell run等一些来执行的，这里其实也是使用shell来执行的。

![]()

也可以使用它自带的一些命令，我们之前在文档中可以看到它支持了很多命令。

比如PS命令可以查看进程列表。

![]()

我们可以使用help命令来查看这些命令如何使用：

![]()

比如说我们使用mimikatz导出密码：

  * 

    
    
    mimikatz coffee privilege::debug sekurlsa::logonpasswords

这里是因为它的mimikatz版本太低了。

![]()

但是我们会发现这个窗口太难受了，命令和这个参杂在一起很难受。

这里可以选择SPLIT Tasking，来将这两个窗口分开。

此时我们左边窗口来输入命令右边来回显我们的执行命令的结果。

![]()

![]()

当然我们都知道Cobalt Strike有一个图形化的界面，也就是很清除的看到那些机器上线了。

Mythic也是有的。点击加号 选择第二个即可。

![]()

我们都知道Cobalt Strike中执行命令不止shell，还有run，在这里也可以使用powerpick来执行命令。

使用powerpick命令来执行或许更OPSEC一些。它更难检测到。

  

![]()

我们也可以执行dotnot程序，可以使用execute-assembly来进行执行，和Cobalt Strike差不多。

接下来介绍下Search Operation。

首先是TASKS，这里我们可以搜索我们使用过的历史命令以及结果。

![]()

还有就是FILES选项，这里的话主要是团队作战的话，可以上传问题供你的队员去下载。

CREDENTIALS选项是如果你使用了mimikatz命令转储的凭据都会保存在这里。

![]()

接下来就是Token这个选项，这个选项中存储的令牌相关的信息。

![]()

还有就是报告功能，我们可以直接输出报告。

![]()

报告里面有我们上线主机的详细信息，还有我们执行操作的详细内容。

![]()

还有你做的一些操作会映射到MITRE中。

![]()

如上就是基础介绍了。

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# Mythic C2 学习

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

轻触阅读原文

![]()

Relay学安全

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

