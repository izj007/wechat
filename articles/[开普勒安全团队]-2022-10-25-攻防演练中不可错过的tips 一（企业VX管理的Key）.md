#  攻防演练中不可错过的tips 一（企业VX管理的Key）

原创 开普勒安全团队  [ 开普勒安全团队 ](javascript:void\(0\);)

**开普勒安全团队** ![]()

微信号 kaipuleanquan

功能介绍 开普勒安全团队秉着四项基本原则：互相尊重团队，互不攻击诽谤、合作共赢、共同提升安全技术。并自成立之初宣布永不商业化，纯民间团队。

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20221025181419.png)  

**  声明**

  

 由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，开普勒安全团队及文章作者不为此承担任何责任。

    开普勒安全团队拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经开普勒安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **简介**

  

在某些集成类软件中，存在一些调用接口的功能，比如某微云桥、在线交互办公平台等，在里面包含了一些不可错过的敏感信息，今天就来说一下其中的企业VX管理员的Key。

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

官方帮助 一

  

获取通讯录管理secret的方法如下：  
1、进入企业微信管理后台，在“管理工具” -- “通讯录同步”开启“API接口同步”

![](https://gitee.com/fuli009/images/raw/master/public/20221025181431.png)  

  

  

  

  

  

  

  

  

  

  

  

2、开启后，可设置通讯录API的权限：读取或者编辑通讯录  

  

  

  

  

  

  

  

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20221025181432.png)  

  

  

  

  

  

  

3、配置企业可信IP，仅所配IP可调用通讯录API。（2022年6月20日起，新开启的通讯录同步必须配置企业可信IP。为保证企业数据安全，请配置本企业服务器的IP地址，不允许配置第三方服务商的IP）

4、使用通讯录同步的secret进行开发。点击查看 示例代码

> 只有管理员才可以看到“管理工具” - “通讯录同步”这个工具，其他分级管理员无操作权限。当企业开启API接口编辑通讯录后，可接收成员变更个人信息的事件。

  

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **官方帮助 二**

  

创建成员：

请求方式：POST（HTTPS）

请求地址：

https://qyapi.weixin.qq.com/cgi-bin/user/create?access_token=ACCESS_TOKEN

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {  "userid": "zhangsan",  "name": "张三",  "alias": "jackzhang",  "mobile": "+86 13800000000",  "department": [1, 2],  "order":[10,40],  "position": "产品经理",  "gender": "1",  "email": "zhangsan@gzdev.com",  "biz_mail":"zhangsan@qyycs2.wecom.work",  "is_leader_in_dept": [1, 0],  "direct_leader":["lisi","wangwu"],  "enable":1,  "avatar_mediaid": "2-G6nrLmr5EC3MNb_-zL1dDdzkd0p7cNliYu9V5w7o8K0",  "telephone": "020-123456",  "address": "广州市海珠区新港中路",  "main_department": 1,  "extattr": {    "attrs": [      {        "type": 0,        "name": "文本名称",        "text": {          "value": "文本"        }      },      {        "type": 1,        "name": "网页名称",        "web": {          "url": "http://www.test.com",          "title": "标题"        }      }    ]  },  "to_invite": true,  "external_position": "高级产品经理",  "external_profile": {    "external_corp_name": "企业简称",    "wechat_channels": {      "nickname": "视频号名称"    },    "external_attr": [      {        "type": 0,        "name": "文本名称",        "text": {          "value": "文本"        }      },      {        "type": 1,        "name": "网页名称",        "web": {          "url": "http://www.test.com",          "title": "标题"        }      },      {        "type": 2,        "name": "测试app",        "miniprogram": {          "appid": "wx8bd8012614784fake",          "pagepath": "/index",          "title": "my miniprogram"        }      }    ]  }}  
    

![]()

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **官方帮助 三**

  

读取成员

请求方式：GET（HTTPS）

请求地址：

https://qyapi.weixin.qq.com/cgi-
bin/user/get?access_token=ACCESS_TOKEN&userid=USERID

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {  "errcode": 0,  "errmsg": "ok",  "userid": "zhangsan",  "name": "张三",  "department": [1, 2],  "order": [1, 2],  "position": "后台工程师",  "mobile": "13800000000",  "gender": "1",  "email": "zhangsan@gzdev.com",  "biz_mail":"zhangsan@qyycs2.wecom.work",  "is_leader_in_dept": [1, 0],  "direct_leader":["lisi","wangwu"],  "avatar": "http://wx.qlogo.cn/mmopen/ajNVdqHZLLA3WJ6DSZUfiakYe37PKnQhBIeOQBO4czqrnZDS79FH5Wm5m4X69TBicnHFlhiafvDwklOpZeXYQQ2icg/0",  "thumb_avatar": "http://wx.qlogo.cn/mmopen/ajNVdqHZLLA3WJ6DSZUfiakYe37PKnQhBIeOQBO4czqrnZDS79FH5Wm5m4X69TBicnHFlhiafvDwklOpZeXYQQ2icg/100",  "telephone": "020-123456",  "alias": "jackzhang",  "address": "广州市海珠区新港中路",  "open_userid": "xxxxxx",  "main_department": 1,  "extattr": {    "attrs": [      {        "type": 0,        "name": "文本名称",        "text": {          "value": "文本"        }      },      {        "type": 1,        "name": "网页名称",        "web": {          "url": "http://www.test.com",          "title": "标题"        }      }    ]  },  "status": 1,  "qr_code": "https://open.work.weixin.qq.com/wwopen/userQRCode?vcode=xxx",  "external_position": "产品经理",  "external_profile": {    "external_corp_name": "企业简称",    "wechat_channels": {      "nickname": "视频号名称",      "status": 1    },    "external_attr": [{        "type": 0,        "name": "文本名称",        "text": {          "value": "文本"        }      },      {        "type": 1,        "name": "网页名称",        "web": {          "url": "http://www.test.com",          "title": "标题"        }      },      {        "type": 2,        "name": "测试app",        "miniprogram": {          "appid": "wx8bd80126147dFAKE",          "pagepath": "/index",          "title": "my miniprogram"        }      }    ]  }}  
      
    

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **名词解释**

  

access_token:是企业后台去企业微信的后台获取信息时的重要票据，由corpid和secret产生。所有接口在通信时都需要携带此信息用于验证接口的访问权限。

corpsecret:是企业应用里面用于保障数据安全的“钥匙”，每一个应用都有一个独立的访问密钥，为了保证数据的安全，secret务必不能泄漏.

corpid:每个企业都拥有唯一的corpid.

userid:每个成员都有唯一的userid.

tagid:每个标签都有唯一的标签id

external_userid:企业外部联系人的id，可能是微信用户，也可能是企业微信用户.

agentid:每个应用都有唯一的agentid.

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **检查权限**

  

access_token 权限，通讯录范围 - 部门，应用权限

https://open.work.weixin.qq.com/devtool/query

![](https://gitee.com/fuli009/images/raw/master/public/20221025181437.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **使用方法**

  

当我们通过某微云桥、在线交互办公平台等方式获取到了企业的Key：

![](https://gitee.com/fuli009/images/raw/master/public/20221025181439.png)

![]()  

我们就可以利用以上的这些官方帮助，实现添加企业员工的操作，比如说创建成员，获取成员列表。

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **举例**

  

###  获取部门列表

https://qyapi.weixin.qq.com/cgi-bin/department/list?access_token=access_token

![](https://gitee.com/fuli009/images/raw/master/public/20221025181441.png)  

### 获取部门成员

https://qyapi.weixin.qq.com/cgi-
bin/user/simplelist?access_token=access_token&department_id=1&&fetch_child=1

![](https://gitee.com/fuli009/images/raw/master/public/20221025181443.png)  

### 获取部门成员详情

https://qyapi.weixin.qq.com/cgi-
bin/user/list?access_token=access_token&department_id=1&fetch_child=1

![](https://gitee.com/fuli009/images/raw/master/public/20221025181444.png)  

### 创建成员

https://qyapi.weixin.qq.com/cgi-bin/user/create?access_token=ACCESS_TOKEN

![]()

  

更多官方帮助可以查看：

  

https://developer.work.weixin.qq.com/document/path/90195

![](https://gitee.com/fuli009/images/raw/master/public/20221025181429.png)

 **案例场景**

  

通过某集成平台，获取到企业VX的管理Key，然后利用此管理员的Key值，创建企业VX员工用户，伪造身份，进行内部钓鱼。

![](https://gitee.com/fuli009/images/raw/master/public/20221025181446.png)

企业VX的工作台中包含VPN，直接使用VPN功能连接目标内网。

关注本公众号

  
  
  

![](https://gitee.com/fuli009/images/raw/master/public/20221025181447.png)

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

