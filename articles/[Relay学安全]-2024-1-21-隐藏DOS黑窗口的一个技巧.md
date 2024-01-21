#  隐藏DOS黑窗口的一个技巧

原创 relaysec  [ Relay学安全 ](javascript:void\(0\);)

**Relay学安全** ![]()

微信号 gh_8d57319ec39c

功能介绍
这是一个纯分享技术的公众号，只想做安全圈的一股清流，不会发任何广告，不会接受任何广告，只会分享纯技术文章，欢迎各行各业的小伙伴关注。让我们一起提升技术。

____

___发表于_

我们一般写的loader或者自己挖的白程序有时候会弹出一个DOS黑窗口，其实自己的写的话很好规避掉，隐藏掉即可。  

但是如果我们挖到一个白程序，也是会弹出一个DOS黑框，卡在那不动得话就很难受，我们又没有源码，不能直接使用如下方式给他规避掉。

  * 

    
    
    #pragma comment(linker, "/subsystem:\"windows\" /entry:\"mainCRTStartup\"")

  * 

    
    
    FreeConsole();

  *   * 

    
    
    HWND handos = GetForegroundWindow();ShoWindow(handos,SW_HIDE);

  * 

    
    
    xxxxxxxxxx int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow);

其实我们除了上面得方式也可以设置Subsystem这个字段，这个字段在可选PE头中，如下微软介绍。

  * 

    
    
    https://learn.microsoft.com/zh-cn/cpp/build/reference/subsystem-specify-subsystem?view=msvc-170

那么我们就可以直接在PE结构中进行更改了。  

可选PE头如下:  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    typedef struct _IMAGE_OPTIONAL_HEADER64 {  WORD                 Magic;  BYTE                 MajorLinkerVersion;  BYTE                 MinorLinkerVersion;  DWORD                SizeOfCode;  DWORD                SizeOfInitializedData;  DWORD                SizeOfUninitializedData;  DWORD                AddressOfEntryPoint;  DWORD                BaseOfCode;  ULONGLONG            ImageBase;  DWORD                SectionAlignment;  DWORD                FileAlignment;  WORD                 MajorOperatingSystemVersion;  WORD                 MinorOperatingSystemVersion;  WORD                 MajorImageVersion;  WORD                 MinorImageVersion;  WORD                 MajorSubsystemVersion;  WORD                 MinorSubsystemVersion;  DWORD                Win32VersionValue;  DWORD                SizeOfImage;  DWORD                SizeOfHeaders;  DWORD                CheckSum;  WORD                 Subsystem;  WORD                 DllCharacteristics;  ULONGLONG            SizeOfStackReserve;  ULONGLONG            SizeOfStackCommit;  ULONGLONG            SizeOfHeapReserve;  ULONGLONG            SizeOfHeapCommit;  DWORD                LoaderFlags;  DWORD                NumberOfRvaAndSizes;  IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];} IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;

微软介绍:  

  * 

    
    
    https://learn.microsoft.com/zh-cn/windows/win32/api/winnt/ns-winnt-image_optional_header64

使用DIE工具打开EXE。选择高级选项，点击PE结构，找到可选PE头，然后找到Subsystem字段，将右上角得只读取消掉。  

![]()

然后将Windows CUI更改为Windows GUI即可。  

![]()

然后我们来看一下效果，没有设置之前:

![]()

设置之后，可以看到DOS框已经规避掉了，这样的话你挖到白名单的一些程序或者如果没有源码的。

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 隐藏DOS黑窗口的一个技巧

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

