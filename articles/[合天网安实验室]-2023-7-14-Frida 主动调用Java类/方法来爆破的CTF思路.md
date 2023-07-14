#  Frida 主动调用Java类/方法来爆破的CTF思路

原创 mi1itray-axe  [ 合天网安实验室 ](javascript:void\(0\);)

**合天网安实验室** ![]()

微信号 hee_tian

功能介绍 为广大信息安全爱好者提供有价值的文章推送服务！

____

___发表于_

收录于合集

  

利用Frida去调用java代码中的类，然后爆破。算是一种主动调用的方法。主动调用可以用于爆破，模拟程序部分执行，需要注意的知识点是在java代码中的static类型数据在爆破过程中需要每次都对这种类型值重新设置。因为static类型在所有实例中都是统一，修改一个实例就会修改所有实例，需要用`变量.属性.value
= ...`的写法重新设置值。

    
    
    var bvar = b.$new(IntClass.$new(2));  
    for (...) {  
        bvar._static_val.value = ...;  
    }

# 背景知识

## java类中静态值在爆破中需要修改

在java类中，一个属性如果是`static`的，不是说这个值不能改，而是说这个属性在程序中是唯一的，无论几个实例，只要改了其中一个实例中static的值，其他实例对应的值也会被改变。

在爆破过程中，如果需要爆破过程中不停new一个新的类实例，记得看看其中有没有static类型的变量。比如下面的这个例子

    
    
    public class b {  
        public static ArrayList<Integer> a = new ArrayList<>();  
        static String b = "abcdefghijklmnopqrstuvwxyz";  
        static Integer d = 0;  
        Integer[] c = {8, 25, 17, 23, 7, 22, 1, 16, 6, 9, 21, 0, 15, 5, 10, 18, 2, 24, 4, 11, 3, 14, 19, 12, 20, 13};  
      
      
        public b(Integer num) {  
            for (int intValue = num.intValue(); intValue < this.c.length; intValue++) {  
                a.add(this.c[intValue]);  
            }  
            for (int i = 0; i < num.intValue(); i++) {  
                a.add(this.c[i]);  
            }  
        }  
    ...

每次new一个b类，比如`b bVar = new
b(2)`.如果要不停调用这个类，并且使用其中的方法，要注意其中的`static`变量会不会变。如果会变，那么在爆破过程中，需要new完实例后，修改static变量的值。

## frida调用java中静态方法与动态方法

如果调用静态方法，可以直接调用，比如java代码如下

    
    
    public class Verifier {  
        private Verifier() {  
        }  
      
        public static boolean verifyPassword(Context context, String input) {  
            ...  
        }

那么如果调用verifyPassword可以直接在frida中调用

    
    
     var verify = Java.use("org.teamsik.ahe17.qualification.Verifier");  
    verify.verifyPassword(a, b);

如果是动态方法，有两种方法可以调用动态方法

第一种是，使用内存中已存在实例的方法，需要用到`java.choose(...)`，这个是在内存中寻找对象

    
    
    //从内存中（堆）直接搜索已存在的对象  
    Java.choose('xxx.xxx.xxx', //这里写类名   
    { //onMatch 匹配到对象执行的回调函数  
        onMatch: function (instance) {  
        },  
        //堆中搜索完成后执行的回调函数  
        onComplete: function () {  
        }  
    });

第二种是，我们new一个新的实例，然后调用实例中的方法

    
    
     //获取类的引用  
    var cls = Java.use('这里写类名');  
      
    //调用构造函数 创建新对象  这里注意参数  
    var obj = cls.$new();

# Easy-QAHE17

首先是看吾爱破解的一道题目，核心代码如下。

    
    
    public void verifyPasswordClick(View view) {  
            String password = this.txPassword.getText().toString();  
            if (!Verifier.verifyPassword(this, password)) {  
                Toast.makeText(this, (int) org.teamsik.ahe17.qualification.easy.R.string.dialog_failure, 1).show();  
            } else {  
                showSuccessDialog();  
            }  
        }  
      
    public class Verifier {  
        private Verifier() {  
        }  
      
        public static boolean verifyPassword(Context context, String input) {  
            if (input.length() != 4) {  
                return false;  
            }  
            byte[] v = encodePassword(input);  
            byte[] p = "09042ec2c2c08c4cbece042681caf1d13984f24a".getBytes();  
            if (v.length == p.length) {  
                for (int i = 0; i < v.length; i++) {  
                    if (v[i] != p[i]) {  
                        return false;  
                    }  
                }  
                return true;  
            }  
            return false;  
        }  
    ...  
    ...  
    ...

输入长度为4，通过分析后面知道输入是数字。所以范围是1000-9999.所以是可以爆破的，但是爆破是要用到`encodePassword`方法，自己写一个当然也可以，但是很麻烦。这里就可以直接frida调用`encodePassword`函数.

> 注意这里encodePassword是静态方法，所以可以直接调用
    
    
    function main() {  
        Java.perform(function x() {  
            console.log("In Java perform")  
            var verify = Java.use("org.teamsik.ahe17.qualification.Verifier")  
            var stringClass = Java.use("java.lang.String")  
            var p = stringClass.$new("09042ec2c2c08c4cbece042681caf1d13984f24a")  
              
            for (var i = 999; i < 10000; i++){  
                var v = stringClass.$new(String(i))  
                var vSign = verify.encodePassword(v)  
                if (parseInt(p) == parseInt(stringClass.$new(vSign))) {  
                    console.log("yes: " + v)  
                    break  
                }  
                console.log("not :" + v)  
            }  
        })  
    }  
    setImmediate(main)

结果

    
    
    not :9078  
    not :9079  
    not :9080  
    not :9081  
    not :9082  
    yes: 9083

需要注意的是，要调用`parseInt`解析内存中的内存再对比，因为string类型是java的string类型，对js代码来说是一段内存。

# EasyJava

这题是纯java题，逻辑很清晰，对输入的每一个字符单个检查，加密并对比。所以可以很简单的想到爆破的思路

    
    
    public static Boolean b(String str) {  
        if (str.startsWith("flag{") && str.endsWith("}")) {  
            String substring = str.substring(5, str.length() - 1);  
            b bVar = new b(2);  
            a aVar = new a(3);  
            StringBuilder sb = new StringBuilder();  
            int i = 0;  
            for (int i2 = 0; i2 < substring.length(); i2++) {  
                sb.append(a(substring.charAt(i2) + "", bVar, aVar));  
                Integer valueOf = Integer.valueOf(bVar.b().intValue() / 25);  
                if (valueOf.intValue() > i && valueOf.intValue() >= 1) {  
                    i++;  
                }  
            }  
            return Boolean.valueOf(sb.toString().equals("wigwrkaugala"));  
        }  
        return false;  
    }

所以可以单个字符爆破，但是要注意到，`com.a.easyjava.b`和`com.a.easyjava.a`两个类中都存在static属性的变量，下面是b类的

    
    
    public class b {  
        public static ArrayList<Integer> a = new ArrayList<>();  
        static String b = "abcdefghijklmnopqrstuvwxyz";  
        static Integer d = 0;  
        Integer[] c = {8, 25, 17, 23, 7, 22, 1, 16, 6, 9, 21, 0, 15, 5, 10, 18, 2, 24, 4, 11, 3, 14, 19, 12, 20, 13};  
      
        public b(Integer num) {  
            for (int intValue = num.intValue(); intValue < this.c.length; intValue++) {  
                a.add(this.c[intValue]);  
            }  
            for (int i = 0; i < num.intValue(); i++) {  
                a.add(this.c[i]);  
            }  
        }  
      
        public static void a() {  
            int intValue = a.get(0).intValue();  
            a.remove(0);  
            a.add(Integer.valueOf(intValue));  
            b += "" + b.charAt(0);  
            b = b.substring(1, 27);  
            Integer num = d;  
            d = Integer.valueOf(d.intValue() + 1);  
        }  
      
        public Integer a(String str) {  
            int i = 0;  
            if (b.contains(str.toLowerCase())) {  
                Integer valueOf = Integer.valueOf(b.indexOf(str));  
                for (int i2 = 0; i2 < a.size() - 1; i2++) {  
                    if (a.get(i2) == valueOf) {  
                        i = Integer.valueOf(i2);  
                    }  
                }  
            } else {  
                i = str.contains(" ") ? -10 : -1;  
            }  
            a();  
            return i;  
        }  
      
        public Integer b() {  
            return d;  
        }  
    }

b类中的a，b，d变量都是static类型的同时，这三个变量都会被下面的方法所改变。所以如果要爆破，需要重新修改实例中的属性值。如果不重新修改属性的值，我们通过观察b类中的b变量可以看到会有什么问题。

>
> 这个脚本是爆破第一个字符在加密后所有的可能性。爆破范围通过分析b类可以缩小到`a-z`，然后模仿加密过程，加密一个字符看看结果。中间每循环一次会重新申请一个`b`和`a`类的实例，想通过申请新的实例来避免类中变量的修改.
    
    
    function main() {  
        Java.perform(function x() {  
            console.log('[+] script load');  
              
            var b = Java.use("com.a.easyjava.b");  
            var a = Java.use("com.a.easyjava.a");  
            var StringClass = Java.use("java.lang.String");  
            var IntClass = Java.use("java.lang.Integer");  
            var MainActivity = Java.use("com.a.easyjava.MainActivity");  
      
            try { // try catch 用来查看报错的，可以去掉  
                for (var i = 97; i < 123; i++) {  
                    var bvar = b.$new(IntClass.$new(2));  
                    var avar = a.$new(IntClass.$new(3));  
      
                    var s = String.fromCharCode(i);  
                    var c = MainActivity.a(s, bvar, avar);  
                    console.log(`enc(${s}) => ${c}, b.a => ${b._b.value}`);  
                }  
            } catch (e) {  
                console.log(e);  
            }  
            console.log('[+] script end');  
        })  
    }  
    setImmediate(main)

结果如下

    
    
    # python3 loader.py  
    [+] script load  
    enc(a) => a, b.b => bcdefghijklmnopqrstuvwxyza  
    enc(b) => a, b.b => cdefghijklmnopqrstuvwxyzab  
    enc(c) => a, b.b => defghijklmnopqrstuvwxyzabc  
    enc(d) => a, b.b => efghijklmnopqrstuvwxyzabcd  
    enc(e) => a, b.b => fghijklmnopqrstuvwxyzabcde  
    enc(f) => a, b.b => ghijklmnopqrstuvwxyzabcdef  
    enc(g) => a, b.b => hijklmnopqrstuvwxyzabcdefg  
    enc(h) => a, b.b => ijklmnopqrstuvwxyzabcdefgh  
    enc(i) => a, b.b => jklmnopqrstuvwxyzabcdefghi  
    enc(j) => a, b.b => klmnopqrstuvwxyzabcdefghij  
    enc(k) => a, b.b => lmnopqrstuvwxyzabcdefghijk  
    enc(l) => a, b.b => mnopqrstuvwxyzabcdefghijkl  
    enc(m) => a, b.b => nopqrstuvwxyzabcdefghijklm  
    enc(n) => a, b.b => opqrstuvwxyzabcdefghijklmn  
    enc(o) => a, b.b => pqrstuvwxyzabcdefghijklmno  
    enc(p) => a, b.b => qrstuvwxyzabcdefghijklmnop  
    enc(q) => a, b.b => rstuvwxyzabcdefghijklmnopq  
    enc(r) => a, b.b => stuvwxyzabcdefghijklmnopqr  
    enc(s) => a, b.b => tuvwxyzabcdefghijklmnopqrs  
    enc(t) => a, b.b => uvwxyzabcdefghijklmnopqrst  
    enc(u) => a, b.b => vwxyzabcdefghijklmnopqrstu  
    enc(v) => a, b.b => wxyzabcdefghijklmnopqrstuv  
    enc(w) => a, b.b => xyzabcdefghijklmnopqrstuvw  
    enc(x) => a, b.b => yzabcdefghijklmnopqrstuvwx  
    enc(y) => a, b.b => zabcdefghijklmnopqrstuvwxy  
    enc(z) => a, b.b => abcdefghijklmnopqrstuvwxyz  
    [+] script end

可以看到实际上，虽然每次new了一个新的实例，但是实例中的static变量是变了的，这导致了之前的爆破会影响到下一次爆破，同时也可以看到加密结果全部都是`a`。所以如果要爆破，就得想办法让每次爆破，新的实例中的值不变。

需要使用`bvar._b.value =
StringClass.$new("abcdefghijklmnopqrstuvwxyz");`这样的语法对static类型的变量重新设值。

> 需要注意的是有些变量在jadx/jeb中看到的名字可能会被重载，需要加一个下划线比如b ->
> _b。可以通过console打印看看是不是unknow，也可以直接用jadx右键复制frida片段，查看此变量frida需不需要加一个下划线

解题脚本的思路就很简单，单个字符来爆破，每次重新生成类的实例，并将类中的值置为初始状态（通过调用类`$init`方法）。

exp

    
    
    function main() {  
        Java.perform(function x() {  
            console.log('[+] script load');  
      
            var b = Java.use("com.a.easyjava.b");  
            var a = Java.use("com.a.easyjava.a");  
            var IntClass = Java.use("java.lang.Integer");  
            var StringClass = Java.use("java.lang.String");  
            var ArrayList = Java.use("java.util.ArrayList");  
            var MainActivity = Java.use("com.a.easyjava.MainActivity");  
      
            var flag = new Array();  
            var cipher = "wigwrkaugala";  
      
            var bvar = b.$new(IntClass.$new(2));  
            var avar = a.$new(IntClass.$new(3));  
      
            for (var _ = 0; _ < cipher.length; _++) {  
                for (var i = 97; i < 123; i++) { // 97 - 123是字母a-z  
                    // reset static value  
                    bvar._b.value = StringClass.$new("abcdefghijklmnopqrstuvwxyz");  
                    bvar.d.value = IntClass.$new(0);  
                    bvar._a.value = ArrayList.$new();  
                    bvar["$init"](IntClass.$new(2));  
      
                    avar.b.value = StringClass.$new("abcdefghijklmnopqrstuvwxyz");  
                    avar.d.value = IntClass.$new(0);  
                    avar._a.value = ArrayList.$new();  
                    avar["$init"](IntClass.$new(3));  
      
                    var s = String.fromCharCode(i);  
                    flag.push(s);  
      
                    for (var e = 0; e < flag.length; e++) {  
                        var c = MainActivity.a(flag[e].toString(), bvar, avar);  
                        if (c != cipher[e]) {  
                            break;  
                        }  
                    }  
                    if (c == cipher[flag.length - 1]) {  
                        console.log(flag);  
                        break  
                    }  
                    flag.length -= 1;  
                }  
            }  
            console.log('flag{' + flag.join('') + '}');  
            console.log('[+] script end');  
        })  
    }  
    setImmediate(main);

结果

    
    
    root@kali ~/frida-script-dev# python3 loader.py  
    [+] script load  
    v  
    v,e  
    v,e,n  
    v,e,n,i  
    v,e,n,i,v  
    v,e,n,i,v,i  
    v,e,n,i,v,i,a  
    v,e,n,i,v,i,a,i  
    v,e,n,i,v,i,a,i,v  
    v,e,n,i,v,i,a,i,v,i  
    v,e,n,i,v,i,a,i,v,i,c  
    v,e,n,i,v,i,a,i,v,i,c,i  
    flag{veniviaivici}  
    [+] script end

# 参考

Frida Android hook | Sakuraのblog (eternalsakura13.com)

52pojie2020春节红包-第三题（升级版）暴力破解

frida主动调用app方法 - 码农教程 (manongjc.com)

java static 区别_Java 静态与动态的区别_

攻防世界新手练习题_MOBILE(移动) - 菜鸟-传奇 - 博客园 (cnblogs.com)

 **原创稿件征集**

征集原创技术文章中，欢迎投递

投稿邮箱：edu@antvsion.com

文章类型：黑客极客技术、信息安全热点安全研究分析等安全相关

通过审核并发布能收获200-800元不等的稿酬。

  

[更多详情，点我查看！](http://mp.weixin.qq.com/s?__biz=MjM5MTYxNjQxOA==&mid=2652885477&idx=1&sn=39e97a60d7b68d19569284654e74ffa1&chksm=bd59ad288a2e243e4d89b7c456fbd44a93d241c881075b342af22431d93dca56e52076ed75ce&scene=21#wechat_redirect)

![](https://gitee.com/fuli009/images/raw/master/public/20230714180401.png)靶场实操，戳“阅读原文
**“**

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

