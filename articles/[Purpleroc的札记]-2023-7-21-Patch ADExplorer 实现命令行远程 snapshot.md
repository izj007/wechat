#  Patch ADExplorer 实现命令行远程 snapshot

原创 Purpleroc [ Purpleroc的札记 ](javascript:void\(0\);)

**Purpleroc的札记** ![]()

微信号 purpleroc_0xFA

功能介绍 分享一些网络安全知识，写写自己的感受经历。为了取代博客吧~

____

___发表于_

收录于合集

ADExplorer.exe是微软的 Sysinternals
套装里的一个工具，本意是提供给系统管理员做一些域管理操作的。由于太好用，在横移中也是经常用到，通常用来做快照、查看域设置、定位域内机器、站点、快照处理后可以直接导入BloodHound等等。详情（自行搜：ADExplorer）。

问题来了，这么好用的工具，应该是会提供命令行的吧。看了下，的确有：  

![]()

网上搜了搜，都是本机执行：

adexplorer -snapshot "" c:\\\snapshot.snp

即可获取当前AD的快照：

![]()

于是，我C哥就在想，怎么样远程用命令行把快照给做了？

好问题，搜了一通后，发现，网上的命令行，对于 <connection-string> 都没有解释，都是清一色用的""的例子。那这个 connection-
string 格式到底是怎么样的？能不能带用户名密码？如果想要命令行远程使用，要怎么操作？

那就，打开久违的IDA + x64dbg走起来吧。先从写文件操作往回跟，定位到 createfile
相关函数俩，猜测分别是命令行和UI操作（基址设置的：0x7FF67DE40000）：

  *   * 

    
    
    snapshot_cmd_7FF67DE736F0snapshot_gui_7FF67DE73FF0
    
    
      
    

调试后，发现在 createfile 上面，有疑似 ldap 连接的函数：open_ldap_7FF67DE67C00，下断点调试：  

![]()

得到如下信息：  
![]()

发现传入的第一个参数RCX其实是一个结构体的指针，建立连接需要用到的信息会存在结构体对应的位置上。这个时候，RCX指向地址在 堆 上。

再看看命令行情况下，这个结构体对应的内容，执行: ADExplorer.exe -snapshot "127.0.0.1" test.dat：  

![]()

可以看到，在结构体中，只有host有信息：

![]()

另外，此时RCX指向的地址在 栈 上。

再回到IDA，GUI模式下：

![]()

可以看出是有对应的函数来获取并存储用户名和密码的。

再看CMD模式下：  

![]()

理论上，CMD模式下应该也是有地方来设置对应的信息的。所以，重点在于传入这个函数的第一个参数了。跟上去看函数调用处：

![]()

(图：信息处理.jpg)

  
可以看到传入的实际上是 connect_string 的地址，而这个地址来自栈上：  

![]()

于是，可以推测，在 connect_string 之后，其实放的是 store_name
，再然后就是账号，由于ida伪代码问题，这里直接给并成了一个__int128了，在其后便是密码的存放位置。

那，在哪给他设置值呢？再看上图中 信息处理.jpg，关键代码如下：

  *   * 

    
    
      password_v35 = 0i64;                          // 密码  _mm_storeu_si128((__m128i *)&username_v34, (__m128i)0i64);// 0 和账号
    
    
      
    
    
    会发现，其实命令行模式下，并没有处理账号密码，而是单纯设置为空，也就是说命令行下的远程snapshot模式是被阉割掉了的。

先前以为connection-
string会以某一种连接方式如：ldap://username:password@host来设置，后来发现在open_ldap_7FF67DE67C00
中第一个函数里，有对host进行处理的函数：  

![]()

它的操作，其实就是单纯拼接上"LDAP://"和"rootDSE"，得到最终的地址为：LDAP://{IP:PORT}/rootDSE。

即然已经知道了，对应栈上位置是用来存储账号密码的，那有没有办法把阉割掉的功能实现呢？理论上是可以的，再看上图 信息处理.jpg，关键代码如下：

    
    
      
    

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
      if ( v9 + 1 >= pNumArgs )    host_v17 = 0i64;  else    host_v17 = (const OLECHAR *)*((_QWORD *)hMem + v9 + 1);  if ( v9 + 2 >= pNumArgs )    store_path = 0i64;  else    store_path = (const WCHAR *)*((_QWORD *)hMem + v9 + 2);    ...     if ( !host_v17 || !store_path )         goto LABEL_47;   password_v35 = 0i64;                          // 密码   _mm_storeu_si128((__m128i *)&username_v34, (__m128i)0i64);// 0 和账号  
    

host和store_path的值均来自hMem，而hMem来自命令行处理：

  *   * 

    
    
      v4 = GetCommandLineW();  hMem = CommandLineToArgvW(v4, &pNumArgs);

而且后面对 host 和 store_path 做了判断，如果为空，就退出，否则，就继续，设置账号密码为空。

于是，可以在这里做文章，即然 hMem 对应的是命令行参数，host、store_path能取到，自然账号密码也可以取到并赋值，往后加偏移即可。

![]()

输入启动命令：ADExplorer64.exe -snapshot "127.0.0.1" test.dat username
password，调试看了看，rdx刚好放着password的地址，而rax刚好放着username的地址。可直接赋值了，直接改：

![]()

为了保持ecx的值不变，同时保持字节数。就将store_path的判断给占用掉了。如此，命令行模式下走到open_ldap_7FF67DE67C00，RCX结构体对应如下：

![]()

于是，就能实现命令行远程snapshot了。

再进一步，可以把首次运行的弹框让用户允许的界面屏蔽掉。

有两种方法：  
1、先运行：ADExplorer.exe /accepteula，这样就不会弹框了  
2、过滤掉判断、弹框逻辑，在：handle_accepteula_7FF67DE41D80函数中。  

![]()

对应的处理办法就是把它nop掉  

![]()

之后，C哥测了一下，结合Inline-Execute-PE可在被控机器上内存加载运行来做远程snapshot。

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

