#  UI Automation 控制 PC 端聊天工具

原创 0xcc [ 非尝咸鱼贩 ](javascript:void\(0\);)

**非尝咸鱼贩** ![]()

微信号 awkwardfish1

功能介绍 临渊羡鱼，不如在家咸鱼

____

___发表于_

收录于合集 #逆向工程 10个

_本文为前几天删掉那篇的修订版_

  

在权限管理相对宽松的桌面操作系统如 Windows，如果运行了不受信任的代码，是可以实现被程序读取并发送微信消息的。

Windows 下有一套 UI Automation 机制，可以自动化模拟用户输入。

  

https://learn.microsoft.com/en-us/dotnet/framework/ui-automation/ui-
automation-overview

  

![]()

  

这套机制仅限于同一用户、同一桌面下的进程。实测如果有多个虚拟桌面，会遍历不到其他桌面下的节点。不过一般人也不会用管理员权限运行 PC 版微信。

  

简单来说，UI Automation 将各种控件以树形结构的形式提供接口，可以遍历节点和模拟操作。

  

Windows SDK 里带了一个 inspect.exe 可以分析 UI。

  

https://learn.microsoft.com/en-us/windows/win32/winauto/inspect-objects

  

装了 Visual Studio 和 Windows SDK 就有。直接在 vs 的工具菜单找到开发者命令提示环境，弹出的 shell（VS2022 上有
cmd 和 PowerShell 任选）里输入 inspect 即可。

  

就是下面这玩意儿：

  

![]()

  

![]()

  

有一点像 F12，查看树形结构，并触发一些简单的 Action（模拟交互）。

  

有了 UI 的树形结构，就可以结合 API 里的 TreeWalker 和 Condition 等来定位界面元素（像不像网页里的 DOM？）。

  

微软还单独出了一个工具叫 Accessibility Insights：  

  

https://accessibilityinsights.io/downloads/

  

![]()

  

我试了一下，不仅界面奇丑，还难用得要命。  

  

这套 API 有大佬用 Python 封装好一个包，目前 1.8k 星标：

https://github.com/yinkaisheng/Python-UIAutomation-for-Windows

  

如果你随便搜一下相关关键字，还能找到一个叫 wxauto 的项目，正是基于上面这个 python 库，也正是做本文写的事。代码量不多，可以很快熟悉 UI
Automation 的玩法。搞个机器人绰绰有余……

  

那么本文到此就可以直接结束了。不过用 Python 写感觉怪怪的，在攻防演练的场景里，还要把解释器打包进去不成？

  

用 cpp 写，操作 COM 还需要手动管理内存，代码可不止多一点点。

  

https://github.com/microsoft/Windows-classic-
samples/blob/main/Samples/UIAutomationDocumentClient/cpp/UiaDocumentClient.cpp

  

PowerShell 也可以用 COM，但是看到网上说里面藏了一个和线程限制有关的坑，需要加特殊的 flag 启动
ps，我就懒得试了；再加上用内联托管代码的方式调用，可能还不如直接用 C# 写起来爽快。

  

实际上是否正确响应 Automation，还需要开发者自行实现。比如 PC 版微信，实测用 Automation 可以遍历到各种控件，但模拟输入不响应。

  

还好 Windows API 还提供了另一种方式，就是简单粗暴的模拟鼠标点击和键盘输入。有一定的条件竞争风险，比如半路弹出个别的窗口抢走焦点。又不是不能用。

  

UI Automation 需要用到至少两个系统库：UIAutomationClient.dll 和和 UIAutomationTypes.dll。

  

直接添加依赖有点问题。放 Google 一搜，得到的答案居然是直接引用 WPF，就会自动包含这两个模块……另外模拟键盘和鼠标输入用了一个 NuGet 包
InputSimulator。

  *   *   *   *   *   *   * 

    
    
      <ItemGroup>    <FrameworkReference Include="Microsoft.WindowsDesktop.App.WPF" />  </ItemGroup>  
      <ItemGroup>    <PackageReference Include="InputSimulator" Version="1.0.4" />  </ItemGroup>

  

如下的示例代码遍历最近的几个聊天窗口，找到“文件传输助手”，唱几句《只因你太美》。你也可以把最下面的那个 if
语句注释掉，这样最近聊过天的几个好友和群聊都知道你会 rap 了。  

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    using System.Windows;using System.Windows.Automation;using WindowsInput;using WindowsInput.Native;  
    internal class Program{    public static InputSimulator inputSim = new();  
        private static void SimulateClick(AutomationElement element)    {        var rect = element.Current.BoundingRectangle;        var x = 65535 * (rect.Left + rect.Width / 2) / SystemParameters.PrimaryScreenWidth;        var y = 65535 * (rect.Top + rect.Height / 2) / SystemParameters.PrimaryScreenHeight;  
            inputSim.Mouse.MoveMouseTo(x, y);        inputSim.Mouse.LeftButtonClick();    }  
        private static void Text(string content)    {        inputSim.Keyboard.ModifiedKeyStroke(VirtualKeyCode.CONTROL, VirtualKeyCode.VK_A);        inputSim.Keyboard.KeyPress(WindowsInput.Native.VirtualKeyCode.DELETE);        inputSim.Keyboard.TextEntry(content);        inputSim.Keyboard.ModifiedKeyStroke(VirtualKeyCode.MENU, VirtualKeyCode.VK_S);    }  
        private static void Main(string[] args)    {        Console.OutputEncoding = System.Text.Encoding.UTF8;  
            var selectorWindow = new PropertyCondition(AutomationElement.ClassNameProperty, "WeChatMainWndForPC");        var winWeChat = AutomationElement.RootElement.FindFirst(TreeScope.Children, selectorWindow);        if (winWeChat == null) return;  
            var selectorRecents = new AndCondition(            new PropertyCondition[] {                    new PropertyCondition(AutomationElement.ControlTypeProperty, ControlType.List),                    new PropertyCondition(AutomationElement.NameProperty, "会话")            }        );  
            var selectorIsListItem = new PropertyCondition(AutomationElement.ControlTypeProperty, ControlType.ListItem);        var lstRecents = new TreeWalker(selectorRecents).GetFirstChild(winWeChat);        var listItems = lstRecents?.FindAll(TreeScope.Children, selectorIsListItem);        if (listItems == null) return;  
            WindowPattern windowPattern = (WindowPattern)winWeChat.GetCurrentPattern(WindowPattern.Pattern);        windowPattern.SetWindowVisualState(WindowVisualState.Normal);  
            foreach (AutomationElement element in listItems)        {            Console.WriteLine(element.Current.Name);  
                if (element.Current.Name == "文件传输助手")            {                SimulateClick(element);  
                    new List<string> {                    "迎面走来的你让我如此蠢蠢欲动",                    "这种感觉我从未有",                    "Cause I got a crush on you",                    "who you"                }                .ForEach(spam => Text(spam));            }        }    }}  
    

  

上面只是纯文字。想自动发图片或者表情包，直接用 Clipboard.SetImage 然后模拟 Ctrl+V 即可。

  

本文的示例有涉及汉字输入，暂时没碰到乱码的问题，不过可能换个环境会有。另外代码没有处理很多边界条件，比如微信窗口被最小化了，系统存在多个显示器，甚至多个桌面等。

  

到这里机智的读者肯定想到，只要是通过鼠标键盘可以做的，这个 API 都支持。比如读取联系人和聊天记录，也只是遍历多几个元素的事情。PC
版现在可以拉群、朋友圈互动……不难想象可能会有人拿来做机器人什么的。

  

这种方式不依赖进程注入，有没有办法检测？

  

有一个系统 API 可以检查读屏软件是否运行，用 SystemParametersInfo 函数
SPI_GETSCREENREADER。可惜这个似乎需要读屏软件主动调用 SPI_SETSCREENREADER，例如 inspect.exe
运行了就可以检测到。但上面的 C# 示例程序并不影响这个函数的返回结果。

  

有文章提到用 manifest 设置 requestedExecutionLevel 元素的 uiAccess 属性禁用 UI
Automation。未做测试。

  

  * 

    
    
    <requestedExecutionLevel level="asInvoker" uiAccess="false" />

  

https://techcommunity.microsoft.com/t5/windows-blog-archive/using-the-
uiaccess-attribute-of-requestedexecutionlevel-to/ba-p/228641

  

禁用掉之后读屏等可访问性软件肯定受影响，看开发者如何取舍了。除了 FANNG 这类国际大厂，还没见到几个产品会考虑 accessibility。

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

