# 0x01 前言

XSS 和越权依然是出镜率比较高的漏洞。

以前我问过一个朋友，什么是越权？他说，就是逻辑漏洞呗。回想起来，我感觉从他的回答中，说明这个人并不了解越权的本质。

个人认为：越权的本质，不是你能看不该看的、你能干不该干的，那都只是现象。透过接口看**越权的本质，是未经权限认证的情况下的数据库交互**。




# 0x02 实例篇




## 越权点1：敏感接口未加权限认证


这是水平越权的常见形式。

**越权原理：**

1、 与数据库交互的时候，其查询参数为用户可控

- 数据库操作，无外乎增删查改。所以越权高发区在一些增删查改的用户操作处。
- 查询参数需为用户可控，控制点有：`http`数据包（请求包、**响应包**）中的 GET\POST 接口、body 数据、json 数据等；敏感接口 url 中的参数；cookie&session 中的一些参数
- 提交信息也是一种「改」；取消操作也是一种「删」

2、 未验证用户权限，没有校验 session 是否匹配。


删：
[华住酒店某处越权删除修改用户信息](https://shuimugan.com/bug/view?bug_no=212974)


查：
[人保财险某系统越权查看大量保单信息](https://shuimugan.com/bug/view?bug_no=208260)
[Mtime时光网越权查看订单](https://shuimugan.com/bug/view?bug_no=206705)

看上去是在修改参数，实际上是把参数带进数据库进行一些这样的操作：


增：

``` sql
insert into tablename values(一些字段) where userid/username=12345/用户名 
```

删： 
```
delete from tablename where id=123 
```


查：

``` sql
select * from tablename where id=12345 
```

改：
``` sql
update 一些字段 tablename set 一些字段 where userid/username=12345/用户名 
```

-------------------


**越权漏洞挖洞流程：**

1、 寻找敏感接口 -- 查看 url 中是否带参数，对这些参数进行测试

2、 双账号测试法 -- 结合存在【增删查改】的功能点进行测试

- [优酷某站越权取消订单](https://shuimugan.com/bug/view?bug_no=169416)

3、 如何寻找 id？

在尝试越权的时候，很多时候我们需要把某 id 换为其他账号的 id,这个时候我们可能会遇到一些「如何寻找此 id 号的问题」。

- 【方法一】：从 HTML 的标签中寻找 id：从浏览器 `console` → `Elements` 里面寻找各种参数 id
    - id 在 js 中：[海底捞一处越权可为任意用户发布说说（类似朋友圈）](https://shuimugan.com/bug/view?bug_no=163031)
    - id 在 a 标签中：[麦德龙网上商城可越权删除其他用户信息](https://shuimugan.com/bug/view?bug_no=152762)
- 【方法二】：在 http 的请求响应包中寻找 id:
    - 修改 json 数据中的 id：[麦德龙网上商城可越权删除其他用户信息](https://shuimugan.com/bug/view?bug_no=152381)
    - 修改 GET 请求接口中的 id：[小牛电动设计缺陷&越权操作](https://shuimugan.com/bug/view?bug_no=167489)
    - 通过修改返回包中的 id：[好利网APP越权登录任意用户+越权获取全站几十万用户信息（密码md5\手机号\身份证\姓名\账户余额\银行卡）](https://shuimugan.com/bug/view?bug_no=156125)
 

4、 有时候，越权需要替换的参数也不一定是明文数字或密文数字，可能直接替换用户名就越权了：[CSDN一个轻微的越权](https://shuimugan.com/bug/view?bug_no=156766)

5、 当替换的参数不是自然数，可以轻松猜到时候，要获取替换的参数，可能就需要掌握参数的生成规律。

- 【越权变形】：通常，越权的难点之一在于「难以寻找存在越权点的敏感接口」，越权的另一个难点在于有时候「越权的平行参数是需要精心构造的」。
    - [飞客某处订单详情越权查看](https://shuimugan.com/bug/view?bug_no=156326) - 参数为时间戳加随机数，掌握了参数的构造方法即可批量获取
    - [金山词霸安卓app越权问题（signature算法太弱轻松破解）](https://shuimugan.com/bug/view?bug_no=190106) -  破解了参数生成算法即可批量生成参数
    - [大华DSS平台低权限账户越权直接修改system密码](https://shuimugan.com/bug/view?bug_no=194741) - 通过逆向 JS 等手段构造越权数据包


6、越权漏洞最大化利用：

- 1、结合其他功能
[易企秀越权修改信息致任意用户登入](https://shuimugan.com/bug/view?bug_no=191904)
这个例子的亮点在于**结合其他功能**。易企秀某接口越权可修改任意用户信息 ，导致可登入任意账号。然后结合密码找回流程可以控制账号。
<br>[合力贷某处越权+登录撞库](https://shuimugan.com/bug/view?bug_no=154106)
在这个例子中，用用户地址的接口（存在平行越权），获得了一些用户账号密码。再拿这些账号密码去测试此站的 login 接口，进行验证。
注：但是其实这并不算「撞库」，而是算信息泄露，因为站点没变。撞库操作指拿着已经拿到的账号密码去别的网站测试登陆。比如，百度账号，用的163邮箱，然后用这百度账号凭证去登陆邮箱。
<br>

- 2、批量
[哇哈哈某系统口令修改处越权漏洞造成大量员工资料泄露](https://shuimugan.com/bug/view?bug_no=171431)
通过修改参数遍历即可达到批量的目的。
<br>

- 3、 多处越权结合使用
[我是如何让别人别人给我买darry ring钻戒的（几处越权）](https://shuimugan.com/bug/view?bug_no=153647)
查看收货地址+修改收货地址两处越权配合，可以达到漏洞利用的目的。


-----------------


**挖越权洞 Tips 总结：**

1. 注意遍历的编号 Range
2. 越权漏洞高发于「地址处」的操作，如：[暴风某站平行越权(用户敏感信息泄露)](https://shuimugan.com/bug/view?bug_no=207583)；另一个「地址ID的越权」：[phpok企业建站系统(越权修改任意用户收货地址)](https://shuimugan.com/bug/view?bug_no=153131)
3. 一切有编号、涉及某种 ID 的地方都可以去 Fuzz
4. 难点在于找接口，需要遍历网站存在数据库交互的功能点，或者存在 id 的接口，需要耐心。
5. 多 fuzz 几个参数点，比如这个包（`USERID=4103251989*****&APPID=310158******`）里面我可能只会去试 USERID，但是实际越权漏洞点在 APPID。[Home Bugs 建行信用卡越权可批量遍历用户信息](https://shuimugan.com/bug/view?bug_no=177744)
6. id 可能写在 url 中，也可能在 GET/POST 包的 body 中：[中燃慧生活APP大量越权操作](https://shuimugan.com/bug/view?bug_no=176144)
7. 改哪些 id 的问题。越权可能修改一处 id 即可达成越权，但也可能需要同时修改多个 id 才可以完成越权。但是经验表明，需要同时修改的参数数量一般`≤2`，这是因为在写 SQL 查询语句的时候，一般也就用 `AND` 并列两个查询条件，很少同时写3个及以上的查询条件。
8. 查看或修改参数时候可以用 `burp` - `proxy` - `params` 看的更清楚：[飞客旅行网泄露常用旅客信息（可越权删除旅客信息）](https://shuimugan.com/bug/view?bug_no=155809)


--------------------


**一些可模仿性、可操作性强的实例记录：**

- [酷派某处越权可泄露所有用户敏感信息（姓名、电话、住址）](https://shuimugan.com/bug/view?bug_no=168230)
- [佑一良品某站多处平行越权漏洞](https://shuimugan.com/bug/view?bug_no=196371)
- [焦点科技旗下领动云建站平台某处存在越权](https://shuimugan.com/bug/view?bug_no=165620)
- [磨房网某处越权漏洞](https://shuimugan.com/bug/view?bug_no=165520)
- [小牛电动某站越权可以获取其他管理员明文密码](https://shuimugan.com/bug/view?bug_no=154280)

------------------------


## 越权点2：绕过前端验证越权


- 【例1】[四川师范大学SQL注射漏洞/XSS/越权访问漏洞打包可getshell
](https://shuimugan.com/bug/view?bug_no=168574)

此例中，直接通过把响应包里面的一段用于重定向的 `script` 代码删了，就绕过了验证越权成功。这种绕过前端验证是否能成功主要取决于后续有没有权限校验。


这种修改响应包需要使用 Burp，经验证如下配置即可修改响应包：

![title](https://leanote.com/api/file/getImage?fileId=5dda98f3ab64411017006f9c)

参考：
[Burp Suite使用中的一些技巧](https://www.pa55w0rd.online/burp/) - 0x06 拦截响应包
[我来分享一个小技巧，Burp修改response欺骗](http://www.hackdig.com/?05/hack-10471.htm)



- 【例2】

用没通过邮箱检验的账号登录，把返回的 `false` 改成了 `true`，绕过了邮箱验证。


-----------------------------


## 越权点3：JWT 垂直越权

JWT 背景知识：[攻击JWT的一些方法](https://xz.aliyun.com/t/6776)

JWT 垂直越权实例：[全程带阻：记一次授权网络攻防演练（上）](https://www.freebuf.com/vuls/211842.html)

---------------------

## 越权点4：直接进后台型越权

注：关于这一类到底算不算越权，存疑。但是早期 wooyun 上一些前辈普遍觉得这类算越权漏洞，就是未经授权查看了后台数据，执行了管理员才应有的的权限。但是如果按照水平越权垂直越权那么想，越权是给你A的权限但是可以执行B的操作，或者给你低权限但是可以执行高权限的操作，那么可能这一类就不算了。现在提交的漏洞中，这一类漏洞大多数被称为「未授权访问」。究竟是否算越权或许不那么重要，读者自行判断。

这类越权单纯是因为一些敏感页面访问权限控制没做好。

- 【例1】[珍味时乐比萨到后台越权访问漏洞](https://shuimugan.com/bug/view?bug_no=163262)

此例中，可以未授权访问后台的一些敏感页面。挖掘方式：

``` 
intitle:订单查询|订单导出
```

另外我自己也遇到过某网站 7777 端口开了http 服务，直接是一个网页， title 就是后台。无需权限进去可直接访问浏览后台数据。挖掘方法：
1. 对单个网站扫 http 服务
2. `site:xxx.com intitle:后台`

- 【例2】[盛大某游戏GM后台越权](https://shuimugan.com/bug/view?bug_no=161400)

此实例中，对一个段扫 banner，发现其中一些可以直接进后台的页面。对一个段扫 banner，也是批量化的常用思路。



- 【例3】phpMyAdmin 越权

[某商场服务配置不当越权访问可泄漏大量订单](https://shuimugan.com/bug/view?bug_no=169992)

phpMyAdmin 即为一种特定的后台。如果能被直接访问，即为越权。

注意 phpMyAdmin 弱口令不算越权，越权就是**没有经过权限认证**而能够进行认证后的各项操作，才能称之为越权。

--------------------------------

# 0x03 修复篇


回归漏洞本质，越权最简单的修复办法是用户对某一数据进行增删改查时需要去校验下所操作的数据是否属于该用户。

修复方案：

1. 检查存在与数据库进行交互的接口，进行严格权限验证；
2. 加入 token 等机制，进行身份校验；参数与 `session`，`token` 等参数做绑定；
3. 对数据进行加密。

总结一下，当你越权时，实际上绕过了什么呢？

绕过了挖不出洞来没东西交差的尴尬。^_^

------

参考文档：

1. [乌云关于越权的漏洞记录中前6页共120条漏洞记录](https://shuimugan.com/bug/index?BugSearch%5Bbug_no%5D=&BugSearch%5Btitle%5D=%E8%B6%8A%E6%9D%83&BugSearch%5Bvendor%5D=&BugSearch%5Bauthor%5D=&BugSearch%5Bbug_type%5D=&BugSearch%5Bbug_level_by_whitehat%5D=&BugSearch%5Bbug_level_by_vendor%5D=&BugSearch%5Brank_by_whitehat%5D=&BugSearch%5Bbug_date%5D=&page=6)，wooyun
2. [全程带阻：记一次授权网络攻防演练（上）](https://www.freebuf.com/vuls/211842.html)，FREEBUF，yangyangwithgnu，2019年8月21日
2. [我的越权之道](https://wooyun.x10sec.org/static/drops/tips-727.html)，wooyun知识库，小川，2013年11月4日 
3. [中粮集团某分公司OA系统SQL注入/越权访问/getshell](https://shuimugan.com/bug/view?bug_no=157540)，wooyun，路人甲，2015年12月2日
4. 私信XSS：[磨房网一处越权加两处未修复XSS](https://shuimugan.com/bug/view?bug_no=165683)，wooyun，天地不仁 以万物为刍狗，2015年12月29日
5. 任意密码重置：[销冠经纪APP任意手机号密码重置/越权](https://shuimugan.com/bug/view?bug_no=165812)，wooyun，路人甲，2015年12月30日
6. [磨房网某处XSS加越权](https://shuimugan.com/bug/view?bug_no=165776)，wooyun，天地不仁 以万物为刍狗，2015年12月29日
