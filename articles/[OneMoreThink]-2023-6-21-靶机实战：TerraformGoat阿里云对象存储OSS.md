#  靶机实战：TerraformGoat阿里云对象存储OSS

原创 罗锦海 [ OneMoreThink ](javascript:void\(0\);)

**OneMoreThink** ![]()

微信号 OneMoreThinkkk

功能介绍 网络安全运营顾问。昨日所思绝非今日立场。

____

___发表于_

收录于合集 #安全技术 4个

  * 1、Bucket资产
      * 1.1、Bucket资产
  * 2、Bucket权限
      * 2.1、ListObjects权限
      * 2.2、GetBucketPolicy权限
      * 2.3、PutBucketPolicy权限
      * 2.4、GetBucketAcl权限
      * 2.5、PutBucketAcl权限
  * 3、Object权限
      * 3.1、GetObject权限
      * 3.2、PutObject权限
      * 3.3、GetObjectAcl权限
      * 3.4、PutObjectAcl权限
  * 4、其它权限
      * 4.1、日志存储
      * 4.2、访问方式
      * 4.3、SSE-OSS服务端加密方式
      * 4.4、SSE-KMS服务端加密方式

# 靶机实战：TerraformGoat阿里云对象存储OSS

1、目录架构（Bucket资产->5个Bucket权限->4个Object权限）可作为攻击方的渗透测试SOP。需要注意的是，真实环境（区别于靶机环境）测试
PutBucketPolicy、PutBuchetACL、PutObjectAcl 权限需先请运维团队备份策略，否则测试完后可能无法恢复原样。

2、每个靶机都会使用完整的渗透测试SOP进行渗透，直到拿到flag为止，因此本文整体上可能有些啰嗦。

3、防守方自查可先建立策略基线，再通过AKSK调用 GetBucketPolicy、GetBucketAcl、GetObjectAcl
权限来检查是否符合基线。

4、靶机搭建工具 TerraformGoat[1]

5、搭建靶机 TerraformGoat/aliyun/oss[2]

## 1、Bucket资产

### 1.1、Bucket资产

1、搭建靶机，获得Bucket域名。（bucket_public_access靶机）

![](https://gitee.com/fuli009/images/raw/master/public/20230621223729.png)

2、通过 **信息收集** 方式发现Bucket资产，就是对收集到的Bucket域名进行访问确认。

2.1、如果Bucket存在：

2.1.1、返回结果是 **AccessDenied** ，说明匿名用户没有 `ListObjects` 权限。

> 参考文档：The bucket you access does not belong to you[3]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223730.png)

2.1.2、登录OSS管理控制台，授予所有用户 `ListObjects` 权限，返回结果就是 **ListBucketResult** 。返回结果中没有
`Contents` 字段，说明Bucket中没有Object。

> 参考文档：ListObjectsV2（GetBucketV2）[4]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223731.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223732.png)

2.2、如果Bucket不存在：

2.2.1、注销靶机，返回结果就是 **NoSuchBucket** ，现实场景中一般是不用了销毁了。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223733.png)

此时可以接管Bucket。登录自己的OSS管理控制台，创建完全相同的Bucket域名（BucketName和Region都相同）就能接管Bucket了。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223734.png)

3、通过 **暴力破解** 方式发现Bucket资产，就是构造Bucket域名进行访问确认，格式需满足
`<BucketName>.<Region>.aliyuncs.com`。

> 参考文档：OSS域名构成规则[5]

3.1、如果格式错误，返回结果是  **InvalidRequest** ，不要犯这种低级错误。

> 参考文档：InvalidBucketName[6]
>
> BucketName命名规范如下：
>  
>  
>     1、只包含小写字母、数字和短划线（-）  
>     > 2、以小写字母或者数字开头和结尾  
>     > 3、长度必须在3~63字符之间

![](https://gitee.com/fuli009/images/raw/master/public/20230621223735.png)

3.2、如果格式正确，返回结果包括AccessDenied、ListBucketResult、NoSuchBucket，注意以下几种情况：

3.2.1、`BucketName` 错误，返回结果是 **NoSuchBucket** ，此时爆破其它BucketName即可。

> 参考文档：The specified bucket does not exist[7]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223736.png)

3.2.2、`Region` 错误，返回结果是 **AccessDenied**
，说明BucketName正确但是Region错误，此时爆破其它Region即可。

> 参考文档：The bucket you are attempting to access must be addressed using the
> specified endpoint. Please send all future requests to this endpoint[8]
>
> 参考文档：访问域名和数据中心[9]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223737.png)

4、安全建议

4.1、暴力破解Bucket域名的行为是无法监控并阻断的，只能假设Bucket域名一定会泄露，并从评估Bucket权限是否合理的角度进行防御。

4.2、在云管平台删除Bucket时，应同时在域名托管平台删除指向该Bucket的域名CNAME记录，从而避免Bucket可被第三方接管。

## 2、Bucket权限

### 2.1、ListObjects权限

1、搭建靶机，获得Bucket域名。（bucket_object_traversal靶机）

![]()

2、访问Bucket域名，返回结果是 **ListBucketResult** ，说明Bucket存在，而且匿名用户有 `ListObjects` 权限。有
`Contents` 字段，说明Bucket中有Object，并能拿到所有Object的 `Key`。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223738.png)

3、访问Key即可获得对象内容。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223739.png)

4、安全建议

4.1、如非业务必须，禁止授予匿名用户 `ListObjects` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223740.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223741.png)

### 2.2、GetBucketPolicy权限

1、搭建靶机，获得Bucket域名。（bucket_policy_readable靶机）

![](https://gitee.com/fuli009/images/raw/master/public/20230621223742.png)

1.1、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![]()

1.2、访问 `?policy`，返回结果是 Bucket Policy，说明匿名用户有 `GetBucketPolicy` 权限。

> 参考文档：GetBucketPolicy[10]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223743.png)

1.3、安全建议

1.3.1、如非业务必须，禁止授予匿名用户 `GetBucketPolicy` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223744.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223745.png)

2、搭建靶机，获得Bucket域名。（special_bucket_policy靶机）

需要说明的是，该靶机不是考核通过匿名用户查看Bucket Policy的能力，而是考核通过AKSK查看Bucket Policy的能力。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223746.png)

2.1、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![]()

2.2、访问 `?policy`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketPolicy` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223747.png)

2.3、假设已经获得AKSK，先下载并安装ossutil命令：

> 参考文档：下载并安装ossutil[11]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223748.png)

再通过 `ossutil config` 为ossutil命令配置AKSK：

> 参考文档：配置访问凭证[12]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223749.png)

最后通过 `./ossutil64 bucket-policy --method get oss://bucketname local_json_file`
获取Bucket Policy配置，发现有 `Condition` 要求UserAgent是test或HxSecurityLab。

> 参考文档：获取Bucket Policy配置[13]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223750.png)

2.4、通过BurpSuite修改UserAgent后，访问Bucket域名，返回结果是 **ListBucketResult** ，说明获得了
`ListObjects` 权限。返回结果中没有 `Contents` 字段，说明Bucket中没有Object。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223751.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223752.png)

2.5、安全建议

2.5.1、避免代码明文使用AccessKey或本地加密存储AccessKey[14]，从而避免AKSK泄露。

### 2.3、PutBucketPolicy权限

1、没有靶机就自己制作靶机。登录OSS管理控制台创建一个Bucket，并为匿名用户添加 `GetBucketPolicy` 和
`PutBucketPolicy` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223754.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223755.png)

2、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223756.png)

3、访问 `?policy`，返回结果是 Bucket Policy，说明匿名用户有 `GetBucketPolicy` 权限。

![]()

4、修改 Bucket Policy，在原有基础上为匿名用户添加 `*` 权限，返回结果是 **200 OK** ，说明匿名用户有
`PutBucketPolicy` 权限。

> 参考文档：PutBucketPolicy[15]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223757.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223758.png)

5、安全建议

5.1、如非业务必须，禁止授予匿名用户 `PutBucketPolicy` 权限。

### 2.4、GetBucketAcl权限

1、没有靶机就自己制作靶机。登录OSS管理控制台创建一个Bucket，并为匿名用户添加 `GetBucketAcl` 和 `PutBucketAcl`
权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223759.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223800.png)

2、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223801.png)

3、访问 `?policy`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketPolicy` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223802.png)

4、修改 Bucket Policy，为匿名用户添加 `*` 权限，返回结果是 **403 Forbidden** ，说明匿名用户没有
`PutBucketPolicy` 权限。

5、访问 `?acl`，返回结果是 **AccessControlPolicy** ，说明匿名用户有 `GetBucketAcl` 权限。

> 参考文档：GetBucketAcl[16]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223803.png)

6、安全建议

6.1、如非业务必须，禁止授予匿名用户 `GetBucketAcl` 权限。

### 2.5、PutBucketAcl权限

1、没有靶机就自己制作靶机。沿用刚才的靶机，修改 Bucket ACL，改为 `public-read`，奇怪的是返回结果是
**AccessDenied** ，表示匿名用户没有 `PutBucketAcl` 权限。

> 参考文档：PutBucketAcl[17]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223804.png)

2、假设已经获得AKSK，使用ossutil命令修改 Bucket ACL，改为 `public-
read`，修改成功，但用的是AKSK的权限，不是匿名用户的 `PutBucketAcl` 权限。

> 参考文档：设置或修改Bucket ACL[18]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223805.png)

![]()

3、安全建议

3.1、如非业务必须，禁止授予匿名用户 `PutBucketAcl` 权限。

## 3、Object权限

### 3.1、GetObject权限

1、搭建靶机，获得Bucket域名。（object_public_access靶机）

![](https://gitee.com/fuli009/images/raw/master/public/20230621223806.png)

2、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223807.png)

3、访问 `?policy`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketPolicy` 权限。

![]()

4、修改 Bucket Policy，为匿名用户添加 `*` 权限，返回结果是 **403 Forbidden** ，说明匿名用户没有
`PutBucketPolicy` 权限。

5、访问 `?acl`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketAcl` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223808.png)

6、修改 Bucket ACL，改为 `public-read-write`，返回结果是 **AccessDenied** ，说明匿名用户没有
`PutBucketAcl` 权限。

7、爆破 Object Key，发现存在Key是 flag 的对象，说明匿名用户有 `GetObject` 权限。

> 参考文档：GetObject[19]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223809.png)

8、访问Key即可获得对象内容。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223810.png)

9、安全建议

9.1、如非业务必须，禁止授予匿名用户 `GetObject` 权限（Bucket Policy层面），以及包含GetObject权限的 `public-
read` 和 `public-read-write` 权限（Bucket ACL层面）。

> 参考文档：读写权限类型[20]

![](https://gitee.com/fuli009/images/raw/master/public/20230621223811.png)

此时Key仍能访问，检查策略发现 Object ACL 是 `public-read`，需改为 `private` 才能正确禁用匿名用户的
`GetObject` 权限。（Object ACL层面）

> 参考文档：匿名请求鉴权流程[21]

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20230621223812.png)

### 3.2、PutObject权限

1、搭建靶机，获得Bucket域名。（unrestricted_file_upload靶机）

![](https://gitee.com/fuli009/images/raw/master/public/20230621223813.png)

2、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223814.png)

3、访问 `?policy`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketPolicy` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223815.png)

4、修改 Bucket Policy，为匿名用户添加 `*` 权限，返回结果是 **403 Forbidden** ，说明匿名用户没有
`PutBucketPolicy` 权限。

5、访问 `?acl`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketAcl` 权限。

![]()

6、修改 Bucket ACL，改为 `public-read-write`，返回结果是 **AccessDenied** ，说明匿名用户没有
`PutBucketAcl` 权限。

7、爆破 Object Key，没有爆出对象，不好判断匿名用户是否有 `GetObject` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223816.png)

8、上传 Object，返回结果是 **200 OK** ，说明匿名用户有 `PutObject` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223817.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223818.png)

上传图片对象。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223819.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223820.png)

9、安全建议

9.1、如非业务必须，禁止授予匿名用户 `PutObject` 权限（Bucket Policy层面），以及包含PutObject权限的 `public-
read-write` 权限（Bucket ACL层面）。

`PutObject`
权限可用于文件篡改，后果非常严重。例如政府官网上的五星红旗图片被替换成了美国的星条旗，例如前端html文件（需支持解析）被替换后进行XSS钓鱼、挂黑页暗链等攻击，例如后端php等脚本文件（需支持解析）被替换后进行网站功能修改、Webshell上传等攻击。

### 3.3、GetObjectAcl权限

1、搭建靶机，获得Bucket域名。（object_acl_readable靶机）![]()

2、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223821.png)

3、访问 `?policy`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketPolicy` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223822.png)

4、修改 Bucket Policy，为匿名用户添加 `*` 权限，返回结果是 **403 Forbidden** ，说明匿名用户没有
`PutBucketPolicy` 权限。

5、访问 `?acl`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketAcl` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223823.png)

6、修改 Bucket ACL，改为 `public-read-write`，返回结果是 **AccessDenied** ，说明匿名用户没有
`PutBucketAcl` 权限。

7、爆破 Object Key，发现存在Key是 flag 的对象，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetObject`
权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223824.png)

![]()

8、上传 Object，返回结果是 **403 Forbidden** ，说明匿名用户没有 `PutObject` 权限。

9、访问对象的 `?acl`，返回结果是 **AccessControlPolicy** ，说明匿名用户有 `GetObjectAcl` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223825.png)

10、安全建议

10.1、如非业务必须，禁止授予匿名用户 `GetObjectAcl` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223826.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223827.png)

### 3.4、PutObjectAcl权限

1、搭建靶机，获得Bucket域名。（object_acl_writable靶机）

![]()

2、访问Bucket域名，返回结果是 **AccessDenied** ，说明Bucket存在，只是匿名用户没有 `ListObjects` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223828.png)

3、访问 `?policy`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketPolicy` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223829.png)

4、修改 Bucket Policy，为匿名用户添加 `*` 权限，返回结果是 **403 Forbidden** ，说明匿名用户没有
`PutBucketPolicy` 权限。

5、访问 `?acl`，返回结果是 **AccessDenied** ，说明匿名用户没有 `GetBucketAcl` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223830.png)

6、修改 Bucket ACL，改为 `public-read-write`，返回结果是 **AccessDenied** ，说明匿名用户没有
`PutBucketAcl` 权限。

7、爆破 Object Key，发现存在Key是 flag.txt 的对象，返回结果是 **AccessDenied** ，说明匿名用户没有
`GetObject` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223831.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223832.png)

8、上传 Object，返回结果是 **403 Forbidden** ，说明匿名用户没有 `PutObject` 权限。

9、访问对象的 `?acl`，返回结果是 **AccessControlPolicy** ，说明匿名用户有 `GetObjectAcl` 权限。

![]()

10、修改 Object ACL，改为 `public-read-write`，返回结果是 **200 OK** ，说明匿名用户有
`PutObjectAcl` 权限。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223833.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223834.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223835.png)

11、安全建议

11.1、如非业务必须，禁止授予匿名用户 `PutObjectAcl` 权限。

## 4、其它权限

### 4.1、日志存储

1、搭建靶机，获得Bucket域名。（bucket_logging_disable靶机）

![](https://gitee.com/fuli009/images/raw/master/public/20230621223836.png)

2、登录OSS管理控制台，发现 `日志存储` 是关闭状态，打开即可。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223837.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223838.png)

3、安全建议

3.1、对象存储应开启日志记录，并对日志进行分析研判与应急响应。

> 参考文档：日志转存[22]

### 4.2、访问方式

1、搭建靶机，获得Bucket域名。（bucket_http_enable靶机）

![]()

2、登录OSS管理控制台，发现 `访问方式` 仅支持 HTTP，改为可支持 HTTPS 即可。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223839.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230621223840.png)

3、安全建议

3.1、对象存储应支持 HTTPS 访问方式。

### 4.3、SSE-OSS服务端加密方式

1、搭建靶机，获得Bucket域名。（server_side_encryption_no_kms_set靶机）

![](https://gitee.com/fuli009/images/raw/master/public/20230621223841.png)

2、登录OSS管理控制台，发现 `服务端加密方式` 未使用KMS方式。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223842.png)

3、安全建议

3.1、若有安全合规要求使用自管理、可指定的密钥，可使用KMS托管密钥进行加解密。

> 参考文档：服务器端加密[23]
>
> 参考文档：加密方式[24]

### 4.4、SSE-KMS服务端加密方式

1、搭建靶机，获得Bucket域名。（server_side_encryption_not_using_BYOK靶机）

![](https://gitee.com/fuli009/images/raw/master/public/20230621223843.png)

2、登录OSS管理控制台，发现 `服务端加密方式` 使用KMS方式，但 `加密密钥` 不存在自带密钥BYOK。

![](https://gitee.com/fuli009/images/raw/master/public/20230621223844.png)

3.1、若有安全合规要求使用自管理、可指定的密钥，可使用KMS托管密钥的自带密钥BYOK进行加解密。 __

> 参考文档： 使用KMS托管密钥进行加解密[25]

### 参考资料

[1]

TerraformGoat:
_https://github.com/HXSecurity/TerraformGoat/blob/main/README_CN.md_

[2]

TerraformGoat/aliyun/oss:
_https://github.com/HXSecurity/TerraformGoat/tree/main/aliyun/oss_

[3]

The bucket you access does not belong to you:
_https://help.aliyun.com/document_detail/185803.html#section-ucp-9zj-ucf_

[4]

ListObjectsV2（GetBucketV2）:
_https://help.aliyun.com/document_detail/187544.html_

[5]

OSS域名构成规则:  _https://help.aliyun.com/document_detail/31834.html#section-
xyk-h5v-tdb_

[6]

InvalidBucketName:
_https://help.aliyun.com/document_detail/185228.html#section-blc-nqq-iob_

[7]

The specified bucket does not exist:
_https://help.aliyun.com/document_detail/185804.html#section-7jw-p56-9do_

[8]

The bucket you are attempting to access must be addressed using the specified
endpoint. Please send all future requests to this endpoint:
_https://help.aliyun.com/document_detail/185803.html#p-jga-uwv-34u_

[9]

访问域名和数据中心:  _https://help.aliyun.com/document_detail/31837.html_

[10]

GetBucketPolicy:  _https://help.aliyun.com/document_detail/154887.html_

[11]

下载并安装ossutil:
_https://help.aliyun.com/document_detail/120075.html#d08636f18cr4c_

[12]

配置访问凭证:
_https://help.aliyun.com/document_detail/474474.html#section-4kp-a7c-n4j_

[13]

获取Bucket Policy配置:
_https://help.aliyun.com/document_detail/129733.html#section-js6-agu-vve_

[14]

避免代码明文使用AccessKey或本地加密存储AccessKey:
_https://help.aliyun.com/document_detail/305636.html#section-xjz-y30-2ct_

[15]

PutBucketPolicy:  _https://help.aliyun.com/document_detail/154652.html_

[16]

GetBucketAcl:  _https://help.aliyun.com/document_detail/31966.html_

[17]

PutBucketAcl:  _https://help.aliyun.com/document_detail/31960.html_

[18]

设置或修改Bucket ACL:
_https://help.aliyun.com/document_detail/120055.html#section-ydz-ga6-l6g_

[19]

GetObject:  _https://help.aliyun.com/document_detail/31980.html_

[20]

读写权限类型:  _https://help.aliyun.com/document_detail/31843.html#section-abc-
esd-t1s_

[21]

匿名请求鉴权流程:  _https://help.aliyun.com/document_detail/212436.html#section-yis-
ml9-g8t_

[22]

日志转存:  _https://help.aliyun.com/document_detail/31868.html_

[23]

服务器端加密:
_https://help.aliyun.com/document_detail/31871.html#section-u6y-yr7-zqt_

[24]

加密方式:  _https://help.aliyun.com/document_detail/31871.html#section-gvf-
xld-5db_

[25]

使用KMS托管密钥进行加解密:  _https://help.aliyun.com/document_detail/31871.html#section-
gvf-xld-5db_

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

