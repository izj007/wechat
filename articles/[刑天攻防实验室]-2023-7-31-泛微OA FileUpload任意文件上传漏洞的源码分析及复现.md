#  泛微OA FileUpload任意文件上传漏洞的源码分析及复现

[ 刑天攻防实验室 ](javascript:void\(0\);)

**刑天攻防实验室** ![]()

微信号 XT-Lab

功能介绍 聚丁科技旗下刑天攻防实验室

____

___发表于_

收录于合集 #代码审计 6个

![]()

### 环境配置

l 系统：Winserver2019

l 数据库：MySQL5.7.26

l JDK：java 1.8.0_371

l 复现版本：ecology9_V10.42

![]()

### 源码分析

第一个数据包对应的接口位置：

workrelate/plan/util/uploaderOperate.jsp ,  

如下图所示：

![]()

在第16行代码处：new了FileUpload对象，追入该方法中，如下图所示：

![]()

获取了请求服务器的ip地址，并且调用isMultipartData方法来判断var1的Content-Type是不是以“multipart/form-
data”开头，然后把var1和var2传进“this.getAttachment”方法，跟进去看看，如下图所示：

![]()

new了DefaultFileRenamePolicy()和SystemComInfo()对象，

且对var4.getFilesystem使用了getCreateDir方法。然后获取var4的isaesencrypt，后续用来判断是否使用了ase加密。生成13位的随机字符串当作asecode。接着if语句判断var4是否需要压缩。

最后返回了MultipartRequest()对象。下面我们来分析这两个地方。

①第504行代码：对var4.getFilesystem使用了getCreateDir方法，该方法的具体功能如下图所示：

![]()

根据我们传进来的参数，可进入第二个if语句。把var0中的“\\\”和“/”替换成“#$^123”，然后再把它替换成文件分隔符，且在var0尾部加上一个文件分隔符。后面通过var0、年份、月份、一个文件分隔符、26个随机数和一个文件分隔符拼接成新的var0并返回。

② MultipartRequest对象的构造方法调用了他的重载方法，如下图所示：

![]()

重载方法如下图所示：

![]()

这里传进来的参数分别对应的是var1、var5、var1.getContentLength()、（String）null、DefaultFileRenamePolicy()
var3、this.needzip、this.needzipencrypt、var2、this.isaesencrypt、this.aescode。因为var1不是空的，所以进入else分支。对一些属性进行了初始化，然后new了MultipartParser()对象，接着if语句判断get请求参数是否为空。我们使用的是post请求，可跳过该方法。

追踪MultipartParser()对象的构造方法，如下图所示：

![]()

初始化encoding，然后通过两种方式获取"Content-Type"，并比较出长度较长的，且判断它是否是以"multipart/form-
data"开头的。

回到MultipartRequest对象的构造方法，继续往下分析，如下图所示：

![]()

进入while循环。对var11使用了readNextPart()方法并赋值给var19，接下来if判断语句分别进入var19.isParam和var19.isFile()分支。

readNextPart()方法的具体实现如下图所示：

![]()

把this.readLine()赋值给line后，如果line不为空则进入else分支，后续的while循环语句是为了去除掉请求包中的空行。

继续往下看，如下图所示：

![]()

第197行代码：从headers向量获取元素的枚举，然后进行遍历，且如果是以"content-
disposition:"开头的话，把它分割成数组，并赋值给name，filename，origname。

接着往下看，如下图所示：

![]()

如果filename为null，new一个ParamPart()对象，且返回part。否则进入else分支，new一个FilePart()对象，返回FilePart。

ParamPart()对象的构造方法如下图所示：

![]()

FilePart()对象的构造方法如下图所示：

![]()

回到最开始的uploaderOperate.jsp 下，如下图所示：

![]()

new了FileUpload()对象后，赋值给fu。然后进行一些参数的初始化。

第37行代码处：调用deu.uploadDocToImg()方法。追踪进去看如下图所示：

![]()

在第111行代码处调用了var1.uploadFiles(var3)，

跟进去，如下图所示：

![]()

把传进来的参数转换成字符串数组，然后调用uploadFiles重载方法，如下图所示：

![]()

可以看到第389行代码处调用了this.saveFile()，继续跟进，如下图所示：

![]()

获取了一些参数值，然后判断是否需要压缩。

接着往下看，如下图所示：

![]()

第789行代码处：通过把上面获取的参数值与文件分隔符进行拼接，且赋值给var16。然后使用this.rs.executeProc()把var16写到数据库里。第793行代码：修改数据库imageFile表中imagefileid
= var25的一些参数值。第794行代码：通过uploadFile()把文件上传到AlioSSobjectManager上。

分析第二个数据包，对应的servlet路径如下classbean/DBstep/OfficeServer.class，如下图所示：

![]()

判断我们的请求是否是POST方法，然后根据获取的“OPTION”值进入不同分支。我们传入的“OPTION”值应该是“INSERTIMAGE”，所以进入第二个else
if分支，如下图所示：

![]()

第113行代码：判断“isInsertImageNew”是否是1，我们是插入图片操作，所以进入该分支。

第115行代码：从数据库“imagefile”表中查询“imagefileid”为“imagefileid4pic”值的“imagefilename”。往下是判断我们的“imagefilename”和“imagefileid4pic”是否为空，且调用OdocFileUtil.getFileExt()方法获取文件名的后缀，如下图所示：

![]()

判断去除空格后的“imagefilename”是否为空，然后查找“.”最后一次出现的索引，如果没有的话返回空格，否则返回该索引后面的字符。

回到service方法，如下图所示：

![]()

第121行代码：如果文件名后缀为空的话，默认把文件后缀设置为“.jpg”，否则是它原本的后缀名。

第128行代码：

调用OdocFileUtil.getFileFromByte()方法，

① 先看里面的

ImageFileManager.getInputStreamById()方法，如下图所示：

![]()

通过“imagefileid4pic”从数据库获取信息给到变量。往下，如下图所示：

![]()

判断是否需要压缩，

随后会通过BufferedInputStream()获取文件流。

② 跟进OdocFileUtil.inputStream2Byte，如下图所示：

![]()

该方法的具体功能是把输入流转换成字节数组，然后返回。

③ 最后看getFileFromByte方法，如下图所示：

![]()

判断var1是否存在且是文件夹，然后拼接var1、文件分隔符和var2，

再调用BufferedOutputStream()将字节数组写进文件。分析结束。

![]()

### 漏洞验证

接口workrelate/plan/util/uploaderOperate.jsp对应的数据包如下图所示：

![]()

把一句话木马放到内容里，发送后可看到fileid的值。这里是把文件上传到了数据库中。

OfficeServer接口对应的数据包如下图所示：

![]()

把imagefileid4pic的值设置为上一个数据包的fileid，发送，既从数据库中把刚才上传的文件下载到系统资源目录里。如下图所示：

![]()

哥斯拉连接成功如下图所示：

![]()

分享文章，私信获取相关代码学习资料

![]()  
![]()

###### 公众号：

###### 刑天攻防实验室

扫码关注 了解更多内容

![]()

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

