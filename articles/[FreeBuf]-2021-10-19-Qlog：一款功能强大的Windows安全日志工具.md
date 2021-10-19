#  Qlog：一款功能强大的Windows安全日志工具

原创 Alpha_h4ck  [ FreeBuf ](javascript:void\(0\);)

**FreeBuf** ![]()

微信号 freebuf

功能介绍 国内网络安全行业门户

____

__

收录于话题

## 关于Qlog

Qlog是一款功能强大的Windows安全日志工具，该工具可以为Windows操作系统上的安全相关事件提供丰富的事件日志记录功能。该工具目前仍处于积极开发状态，当前版本为Alpha版本。Qlog没有使用API钩子技术，也不需要在目标系统上安装驱动程序，Qlog指挥使用ETW检索遥测数据。当前版本的Qlog仅支持“进程创建”事件，之后还会添加更多丰富的事件支持。Qlog可以看作为Windows服务运行，但也可以在控制台模式下运行，因此我们可以将丰富的事件信息直接传输到控制台进行处理。

## 工作机制

Qlog可以从ETW读取数据，并将丰富的事件信息写入Qlog的事件通道，工具将会创建并使用名为“QMonitor”的新事件源，并写入Windows事件日志中。

 **以下是Qlog的事件处理顺序：**

> 创建ETW会话，并订阅相关内核和用户区ETW Provider；
>
> 从ETW提供程序读取事件；
>
> 丰富的事件支持；
>
> 将丰富的事件写入事件日志通道QLOG；

## 工具依赖&安装&使用

Qlog的运行需要在本地系统中安装并配置好.NET Framework >= 4.7.2环境。

接下来，我们需要使用下列命令将该项目克隆至本地：

    
          * 
    
    
    
    git clone https://github.com/threathunters-io/QLOG.git

接下来，我们可以使用下列命令以交互式终端模式运行Qlog：

    
          * 
    
    
    
    qlog.exe

或者，以Windows服务的方式运行：

    
          *   *   *   * 
    
    
    
    #安装服务qlog.exe -i#卸载服务qlog.exe -u

## 进程处理事件数据输出

    
          *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 
    
    
    
    {  
      "EventGuid": "68795fe8-67e7-410b-a5c0-8364746d7ffe",  
      "StartTime": "2021-07-11T11:06:56.9621746+02:00",  
      "QEventID": 100,  
      "QType": "Process Create",  
      "Username": "TESTOS\\TESTUSER",  
      "Imagefilename": "TEAMS.EXE",  
      "KernelImagefilename": "TEAMS.EXE",  
      "OriginalFilename": "TEAMS.EXE",  
      "Fullpath": "C:\\Users\\TESTUSER\\AppData\\Local\\Microsoft\\Teams\\current\\Teams.exe",  
      "PID": 21740,  
      "Commandline": "\"C:\\Users\\TESTUSER\\AppData\\Local\\Microsoft\\Teams\\current\\Teams.exe\" --type=renderer --autoplay-policy=no-user-gesture-required --disable-background-timer-throttling --field-trial-handle=1668,499009601563875864,12511830007210419647,131072 --enable-features=WebComponentsV0Enabled --disable-features=CookiesWithoutSameSiteMustBeSecure,SameSiteByDefaultCookies,SpareRendererForSitePerProcess --lang=de --enable-wer --ms-teams-less-cors=522133263 --app-user-model-id=com.squirrel.Teams.Teams --app-path=\"C:\\Users\\jocke",  
      "Modulecount": 41,  
      "TTPHash": "42AC63285408F5FD91668B16F8E9157FD97046AB63E84117A14E31A188DDC62F",  
      "Imphash": "F14F00FA1D4C82B933279C1A28957252",  
      "sha256": "155625190ECAA90E596CB258A07382184DB738F6EDB626FEE4B9652FA4EC1CC2",  
      "md5": "9453BC2A9CC489505320312F4E6EC21E",  
      "sha1": "7219CB54AC535BA55BC1B202335A6291FDC2D76E",  
      "ProcessIntegrityLevel": "None",  
      "isOndisk": true,  
      "isRunning": true,  
      "Signed": "Signature valid",  
      "AuthenticodeHash": "B8AD58EE5C35B3F80C026A318EEA34BABF6609C077CB3D45AEE69BF5C9CF8E11",  
      "Signatures": [  
        {  
          "Subject": "CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
          "Issuer": "CN=Microsoft Code Signing PCA 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
          "NotBefore": "15.12.2020 22:24:20",  
          "NotAfter": "02.12.2021 22:24:20",  
          "DigestAlgorithmName": "SHA256",  
          "Thumbprint": "E8C15B4C98AD91E051EE5AF5F524A8729050B2A2",  
          "TimestampSignatures": [  
            {  
              "Subject": "CN=Microsoft Time-Stamp Service, OU=Thales TSS ESN:3BBD-E338-E9A1, OU=Microsoft America Operations, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
              "Issuer": "CN=Microsoft Time-Stamp PCA 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
              "NotBefore": "12.11.2020 19:26:02",  
              "NotAfter": "11.02.2022 19:26:02",  
              "DigestAlgorithmName": "SHA256",  
              "Thumbprint": "E8220CE2AAD2073A9C8CD78752775E29782AABE8",  
              "Timestamp": "15.06.2021 00:39:50 +02:00"  
            }  
          ]  
        },  
        {  
          "Subject": "CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
          "Issuer": "CN=Microsoft Code Signing PCA 2011, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
          "NotBefore": "15.12.2020 22:31:47",  
          "NotAfter": "02.12.2021 22:31:47",  
          "DigestAlgorithmName": "SHA256",  
          "Thumbprint": "C774204049D25D30AF9AC2F116B3C1FB88EE00A4",  
          "TimestampSignatures": [  
            {  
              "Subject": "CN=Microsoft Time-Stamp Service, OU=Thales TSS ESN:F87A-E374-D7B9, OU=Microsoft Operations Puerto Rico, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
              "Issuer": "CN=Microsoft Time-Stamp PCA 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
              "NotBefore": "14.01.2021 20:02:23",  
              "NotAfter": "11.04.2022 21:02:23",  
              "DigestAlgorithmName": "SHA256",  
              "Thumbprint": "ED2C601EDD49DD2A934D2AB32DCACC19940161EF",  
              "Timestamp": "15.06.2021 00:39:53 +02:00"  
            }  
          ]  
        }  
      ],  
      "ParentProcess": {  
        "EventGuid": null,  
        "StartTime": "2021-07-11T09:54:28.9558001+02:00",  
        "QEventID": 100,  
        "QType": "Process Create",  
        "Username": "TEST-OS\\TESTUSER",  
        "Imagefilename": "",  
        "KernelImagefilename": "",  
        "OriginalFilename": "TEAMS.EXE",  
        "Fullpath": "C:\\Users\\TESTUSER\\AppData\\Local\\Microsoft\\Teams\\current\\Teams.exe",  
        "PID": 16232,  
        "Commandline": "C:\\Users\\TESTUSER\\AppData\\Local\\Microsoft\\Teams\\current\\Teams.exe ",  
        "Modulecount": 162,  
        "TTPHash": "",  
        "Imphash": "F14F00FA1D4C82B933279C1A28957252",  
        "sha256": "155625190ECAA90E596CB258A07382184DB738F6EDB626FEE4B9652FA4EC1CC2",  
        "md5": "9453BC2A9CC489505320312F4E6EC21E",  
        "sha1": "7219CB54AC535BA55BC1B202335A6291FDC2D76E",  
        "ProcessIntegrityLevel": "Medium",  
        "isOndisk": true,  
        "isRunning": true,  
        "Signed": "Signature valid",  
        "AuthenticodeHash": "B8AD58EE5C35B3F80C026A318EEA34BABF6609C077CB3D45AEE69BF5C9CF8E11",  
        "Signatures": [  
          {  
            "Subject": "CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
            "Issuer": "CN=Microsoft Code Signing PCA 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
            "NotBefore": "15.12.2020 22:24:20",  
            "NotAfter": "02.12.2021 22:24:20",  
            "DigestAlgorithmName": "SHA256",  
            "Thumbprint": "E8C15B4C98AD91E051EE5AF5F524A8729050B2A2",  
            "TimestampSignatures": [  
              {  
                "Subject": "CN=Microsoft Time-Stamp Service, OU=Thales TSS ESN:3BBD-E338-E9A1, OU=Microsoft America Operations, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
                "Issuer": "CN=Microsoft Time-Stamp PCA 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
                "NotBefore": "12.11.2020 19:26:02",  
                "NotAfter": "11.02.2022 19:26:02",  
                "DigestAlgorithmName": "SHA256",  
                "Thumbprint": "E8220CE2AAD2073A9C8CD78752775E29782AABE8",  
                "Timestamp": "15.06.2021 00:39:50 +02:00"  
              }  
            ]  
          },  
          {  
            "Subject": "CN=Microsoft Corporation, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
            "Issuer": "CN=Microsoft Code Signing PCA 2011, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
            "NotBefore": "15.12.2020 22:31:47",  
            "NotAfter": "02.12.2021 22:31:47",  
            "DigestAlgorithmName": "SHA256",  
            "Thumbprint": "C774204049D25D30AF9AC2F116B3C1FB88EE00A4",  
            "TimestampSignatures": [  
              {  
                "Subject": "CN=Microsoft Time-Stamp Service, OU=Thales TSS ESN:F87A-E374-D7B9, OU=Microsoft Operations Puerto Rico, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
                "Issuer": "CN=Microsoft Time-Stamp PCA 2010, O=Microsoft Corporation, L=Redmond, S=Washington, C=US",  
                "NotBefore": "14.01.2021 20:02:23",  
                "NotAfter": "11.04.2022 21:02:23",  
                "DigestAlgorithmName": "SHA256",  
                "Thumbprint": "ED2C601EDD49DD2A934D2AB32DCACC19940161EF",  
                "Timestamp": "15.06.2021 00:39:53 +02:00"  
              }  
            ]  
          }  
        ],  
        "ParentProcess": null  
      }  
    }

## 项目地址

 **Qlog：** 【点击阅读原文】

## 参考资料

https://threathunters.io  

![](https://gitee.com/fuli009/images/raw/master/public/20211019143502.png)  

  

精彩推荐

  
  
  
  
  
 **
**![](https://gitee.com/fuli009/images/raw/master/public/20211019143508.png)****  

[![](https://gitee.com/fuli009/images/raw/master/public/20211019143509.png)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486625&idx=1&sn=b68fcc53d322bb9a2e43d00a112ca40d&chksm=ce1cf63ef96b7f287356248af6a7c190268d07993c30b87488a27f8b1a6cb71f68be08ea4b78&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20211019143510.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486586&idx=1&sn=8fb235328402751e06c0578e05f3c905&scene=21#wechat_redirect)

[![](https://gitee.com/fuli009/images/raw/master/public/20211019143512.png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486647&idx=1&sn=13ae89f5104291b30864af54ff28ceda&scene=21#wechat_redirect)
** ** ** ** ** ** **![]()**************

预览时标签不可点

收录于话题 #

个 __

上一篇 下一篇

阅读原文

阅读

分享 收藏

赞 在看

____已同步到看一看[写下你的想法](javascript:;)

前往“发现”-“看一看”浏览“朋友在看”

![示意图](//res.wx.qq.com/mmbizwap/zh_CN/htmledition/images/pic/appmsg/pic_like_comment55871f.png)

前往看一看

**看一看入口已关闭**

在“设置”-“通用”-“发现页管理”打开“看一看”入口

[我知道了](javascript:;)

__

已发送

取消 __

####  发送到看一看

发送

Qlog：一款功能强大的Windows安全日志工具

最多200字，当前共字

__

发送中

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。 视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

