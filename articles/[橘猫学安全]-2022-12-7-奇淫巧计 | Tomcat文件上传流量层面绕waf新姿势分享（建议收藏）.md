#  奇淫巧计 | Tomcat文件上传流量层面绕waf新姿势分享（建议收藏）

[ 橘猫学安全 ](javascript:void\(0\);)

**橘猫学安全** ![]()

微信号 gh_af700ee13397

功能介绍 每日一干货🙂

____

___发表于_

收录于合集

写在前面

无意中看到ch1ng师傅的文章觉得很有趣，不得不感叹师傅太厉害了，但我一看那长篇的函数总觉得会有更骚的东西，所幸还真的有，借此机会就发出来一探究竟，同时也不得不感慨下RFC文档的妙处，当然本文针对的技术也仅仅只是在流量层面上waf的绕过。  

## Pre

很神奇对吧，当然这不是终点,接下来我们就来一探究竟。![](https://gitee.com/fuli009/images/raw/master/public/20221207172258.png)

##  

## 前置

这里简单说一下师傅的思路部署与处理上传war的servlet是
`org.apache.catalina.manager.HTMLManagerServlet`在文件上传时最终会通过处理
`org.apache.catalina.manager.HTMLManagerServlet#upload`![](https://gitee.com/fuli009/images/raw/master/public/20221207172302.png)  
调用的是其子类实现类`org.apache.catalina.core.ApplicationPart#getSubmittedFileName`这里获取filename的时候的处理很有趣![](https://gitee.com/fuli009/images/raw/master/public/20221207172304.png)  
看到这段注释，发现在RFC 6266文档当中也提出这点  

    
    
    1   Avoid including the "\" character in the quoted-string form of the filename parameter, as escaping is not implemented by some user agents, and "\" can be considered an illegal path character.

  
那么我们的tomcat是如何处理的嘞？这里它通过函数`HttpParser.unquote`去进行处理。

    
    
      
    

|

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public static String unquote(String input) {        if (input == null || input.length() < 2) {            return input;        }  
            int start;        int end;  
            // Skip surrounding quotes if there are any        if (input.charAt(0) == '"') {            start = 1;            end = input.length() - 1;        } else {            start = 0;            end = input.length();        }  
            StringBuilder result = new StringBuilder();        for (int i = start ; i < end; i++) {            char c = input.charAt(i);            if (input.charAt(i) == '\\') {                i++;                result.append(input.charAt(i));            } else {                result.append(c);            }        }        return result.toString();    }  
  
---|---  
简单做个总结。如果首位是`"`(前提条件是里面有`\`字符)，那么就会去掉跳过从第二个字符开始，并且末尾也会往前移动一位，同时会忽略字符`\`，师傅只提到了类似`test.\war`这样的例子。但其实根据这个我们还可以进一步构造一些看着比较恶心的比如
`filename=""y\4.\w\arK"
。`![](https://gitee.com/fuli009/images/raw/master/public/20221207172306.png)

##  

## 深入

还是在
`org.apache.catalina.core.ApplicationPart#getSubmittedFileName`当中，一看到这个将字符串转换成map的操作总觉得里面会有更骚的东西(这里先是解析传入的参数再获取，如果解析过程有利用点那么也会影响到后面参数获取)，不扯远继续回到正题![](https://gitee.com/fuli009/images/raw/master/public/20221207172307.png)  
首先它会获取header参数`Content-Disposition`当中的值，如果以`form-
data`或者`attachment`开头就会进行我们的解析操作，跟进去一看果不其然，看到`RFC2231Utility`瞬间不困了![](https://gitee.com/fuli009/images/raw/master/public/20221207172310.png)  
后面这一坨就不必多说了，相信大家已经很熟悉啦支持QP编码，忘了的可以考古看看我之前写的文章Java文件上传大杀器-绕waf(针对commons-
fileupload组件)，这里就不再重复这个啦，我们重点看三元运算符前面的这段既然如此，我们先来看看这个hasEncodedValue判断标准是什么，字符串末尾是否带`*`

  *   *   *   *   *   * 

    
    
    1   public static boolean hasEncodedValue(final String paramName) {2     if (paramName != null) {3       return paramName.lastIndexOf('*') == (paramName.length() - 1);4     }5     return false;6   }

在看解密函数之前我们可以先看看RFC 2231文档当中对此的描述，英文倒是很简单不懂的可以在线翻一下，这里就不贴中文了  

  *   *   *   * 

    
    
    Asterisks ("*") are reused to provide the indicator that language and character set 2   information is present and encoding is being used. A single quote ("'") is used to delimit the character set and language information at the beginning of the parameter value. Percent signs ("%") are used as the encoding flag, which agrees with RFC 2047.Specifically, an asterisk at the end of a parameter name acts as an indicator that character set and language information may appear at  the beginning of the parameter value. A single quote is used to separate the character set, language, and actual value information in the parameter value string, and an percent sign is used to flag octets encoded in hexadecimal.  For example:Content-Type: application/x-stuff;         title*=us-ascii'en-us'This%20is%20%2A%2A%2Afun%2A%2A%2A

接下来回到正题，我们继续看看这个解码做了些什么  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public static String decodeText(final String encodedText) throws UnsupportedEncodingException {  final int langDelimitStart = encodedText.indexOf('\'');  if (langDelimitStart == -1) {    // missing charset    return encodedText;  }  final String mimeCharset = encodedText.substring(0, langDelimitStart);  final int langDelimitEnd = encodedText.indexOf('\'', langDelimitStart + 1);  if (langDelimitEnd == -1) {    // missing language    return encodedText;  }  final byte[] bytes = fromHex(encodedText.substring(langDelimitEnd + 1));  return new String(bytes, getJavaCharset(mimeCharset));}

结合注释可以看到标准格式`@param encodedText - Text to be decoded has a format of {@code
<charset>'<language>'<encoded_value>}`,分别是编码，语言和待解码的字符串，同时这里还适配了对url编码的解码，也就是`fromHex`函数,具体代码如下，其实就是url解码

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private static byte[] fromHex(final String text) {  final int shift = 4;  final ByteArrayOutputStream out = new ByteArrayOutputStream(text.length());  for (int i = 0; i < text.length();) {    final char c = text.charAt(i++);    if (c == '%') {      if (i > text.length() - 2) {        break; // unterminated sequence      }      final byte b1 = HEX_DECODE[text.charAt(i++) & MASK];      final byte b2 = HEX_DECODE[text.charAt(i++) & MASK];      out.write((b1 << shift) | b2);    } else {      out.write((byte) c);    }  }  return out.toByteArray();}

因此我们将值当中值得注意的点梳理一下

  1. 支持编码的解码
  2. 值当中可以进行url编码
  3. @code<charset>'<language>'<encoded_value> 中间这位language可以随便写，代码里没有用到这个的处理

  
既然如此那么我们首先就可以排出掉utf-8，毕竟这个解码后就直接是明文，从Java标准库当中的charsets.jar可以看出，支持的编码有很多![](https://gitee.com/fuli009/images/raw/master/public/20221207172313.png)  
同时通过简单的代码也可以输出

    
    
      
    

|

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    Locale locale = Locale.getDefault();Map<String, Charset> maps = Charset.availableCharsets();StringBuilder sb = new StringBuilder();sb.append("{");for (Map.Entry<String, Charset> entry : maps.entrySet()) {  String key = entry.getKey();  Charset value = entry.getValue();  sb.append("\"" + key + "\",");}sb.deleteCharAt(sb.length() - 1);sb.append("}");System.out.println(sb.toString());  
  
---|---  
运行输出

  *   * 

    
    
    //res{"Big5","Big5-HKSCS","CESU-8","EUC-JP","EUC-KR","GB18030","GB2312","GBK","IBM-Thai","IBM00858","IBM01140","IBM01141","IBM01142","IBM01143","IBM01144","IBM01145","IBM01146","IBM01147","IBM01148","IBM01149","IBM037","IBM1026","IBM1047","IBM273","IBM277","IBM278","IBM280","IBM284","IBM285","IBM290","IBM297","IBM420","IBM424","IBM437","IBM500","IBM775","IBM850","IBM852","IBM855","IBM857","IBM860","IBM861","IBM862","IBM863","IBM864","IBM865","IBM866","IBM868","IBM869","IBM870","IBM871","IBM918","ISO-2022-CN","ISO-2022-JP","ISO-2022-JP-2","ISO-2022-KR","ISO-8859-1","ISO-8859-13","ISO-8859-15","ISO-8859-2","ISO-8859-3","ISO-8859-4","ISO-8859-5","ISO-8859-6","ISO-8859-7","ISO-8859-8","ISO-8859-9","JIS_X0201","JIS_X0212-1990","KOI8-R","KOI8-U","Shift_JIS","TIS-620","US-ASCII","UTF-16","UTF-16BE","UTF-16LE","UTF-32","UTF-32BE","UTF-32LE","UTF-8","windows-1250","windows-1251","windows-1252","windows-1253","windows-1254","windows-1255","windows-1256","windows-1257","windows-1258","windows-31j","x-Big5-HKSCS-2001","x-Big5-Solaris","x-COMPOUND_TEXT","x-euc-jp-linux","x-EUC-TW","x-eucJP-Open","x-IBM1006","x-IBM1025","x-IBM1046","x-IBM1097","x-IBM1098","x-IBM1112","x-IBM1122","x-IBM1123","x-IBM1124","x-IBM1166","x-IBM1364","x-IBM1381","x-IBM1383","x-IBM300","x-IBM33722","x-IBM737","x-IBM833","x-IBM834","x-IBM856","x-IBM874","x-IBM875","x-IBM921","x-IBM922","x-IBM930","x-IBM933","x-IBM935","x-IBM937","x-IBM939","x-IBM942","x-IBM942C","x-IBM943","x-IBM943C","x-IBM948","x-IBM949","x-IBM949C","x-IBM950","x-IBM964","x-IBM970","x-ISCII91","x-ISO-2022-CN-CNS","x-ISO-2022-CN-GB","x-iso-8859-11","x-JIS0208","x-JISAutoDetect","x-Johab","x-MacArabic","x-MacCentralEurope","x-MacCroatian","x-MacCyrillic","x-MacDingbat","x-MacGreek","x-MacHebrew","x-MacIceland","x-MacRoman","x-MacRomania","x-MacSymbol","x-MacThai","x-MacTurkish","x-MacUkraine","x-MS932_0213","x-MS950-HKSCS","x-MS950-HKSCS-XP","x-mswin-936","x-PCK","x-SJIS_0213","x-UTF-16LE-BOM","X-UTF-32BE-BOM","X-UTF-32LE-BOM","x-windows-50220","x-windows-50221","x-windows-874","x-windows-949","x-windows-950","x-windows-iso2022jp"}

这里作为掩饰我就随便选一个了`UTF-16BE`![](https://gitee.com/fuli009/images/raw/master/public/20221207172315.png)  
同样的我们也可以进行套娃结合上面的`filename=""y\4.\w\arK"`改成`filename="UTF-16BE'Y4tacker'%00%22%00y%00%5C%004%00.%00%5C%00w%00%5C%00a%00r%00K"`接下来处理点小加强，可以看到在这里分隔符无限加，而且加了🌟号的字符之后也会去除一个🌟号![](https://gitee.com/fuli009/images/raw/master/public/20221207172317.png)  
因此我们最终可以得到如下payload，此时仅仅基于正则的waf规则就很有可能会失效

  *   *   *   *   *   *   * 

    
    
    ------WebKitFormBoundaryQKTY1MomsixvN8vXContent-Disposition: form-data*;;;;;;;;;;name*="UTF-16BE'Y4tacker'%00d%00e%00p%00l%00o%00y%00W%00a%00r";;;;;;;;filename*="UTF-16BE'Y4tacker'%00%22%00y%00%5C%004%00.%00%5C%00w%00%5C%00a%00r%00K"Content-Type: application/octet-stream  
    123  
    ------WebKitFormBoundaryQKTY1MomsixvN8vX--

可以看见成功上传![](https://gitee.com/fuli009/images/raw/master/public/20221207172319.png)

## 变形 更新2022-06-20

这里测试版本是Tomcat8.5.72，这里也不想再测其他版本差异了只是提供一种思路在此基础上我发现还可以做一些新的东西，其实就是对`org.apache.tomcat.util.http.fileupload.ParameterParser#parse(char[],
int, int, char)`函数进行深入分析在获取值的时候`paramValue = parseQuotedToken(new char[]
{separator
});`，其实是按照分隔符`;`分割，因此我们不难想到前面的东西其实可以不用`"`进行包裹，在parseQuotedToken最后返回调用的是`return
getToken(true);`，这个函数也很简单就不必多解释

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private String getToken(final boolean quoted) {        // Trim leading white spaces        while ((i1 < i2) && (Character.isWhitespace(chars[i1]))) {            i1++;        }        // Trim trailing white spaces        while ((i2 > i1) && (Character.isWhitespace(chars[i2 - 1]))) {            i2--;        }        // Strip away quotation marks if necessary        if (quoted            && ((i2 - i1) >= 2)            && (chars[i1] == '"')            && (chars[i2 - 1] == '"')) {            i1++;            i2--;        }        String result = null;        if (i2 > i1) {            result = new String(chars, i1, i2 - i1);        }        return result;    }

可以看到这里也是成功识别的![](https://gitee.com/fuli009/images/raw/master/public/20221207172327.png)既然调用`parse`解析参数时可以不被包裹，结合getToken函数我们可以知道在最后一个参数其实就不必要加`;`了，并且解析完通过`params.get("filename")`获取到参数后还会调用到`org.apache.tomcat.util.http.parser.HttpParser#unquote`那也可以基于此再次变形为了直观这里就直接明文了，是不是也很神奇![](https://gitee.com/fuli009/images/raw/master/public/20221207172329.png)

## 扩大利用面

现在只是war包的场景，多多少少影响性被降低，但我们这串代码其实抽象出来就一个关键  

  *   * 

    
    
    Part warPart = request.getPart("deployWar");String filename = warPart.getSubmittedFileName();

通过查询官方文档，可以发现从Servlet3.1开始，tomcat新增了对此的支持，也就意味着简单通过`javax.servlet.http.HttpServletRequest#getParts`即可，简化了我们文件上传的代码负担(如果我是开发人员，我肯定首选也会使用，谁不想当懒狗呢)

    
    
      
    

|

  *   *   *   *   *   *   * 

    
    
    getSubmittedFileNameString getSubmittedFileName()Gets the file name specified by the clientReturns:the submitted file nameSince:Servlet 3.1  
  
---|---  
  
## 更新Spring 2022-06-20

早上起床想着昨晚和陈师的碰撞，起床后又看了下陈师的星球，看到这个不妨再试试Spring是否也按照了RFC的实现呢（毕竟Spring内置了Tomcat，可能会有类似的呢）![](https://gitee.com/fuli009/images/raw/master/public/20221207172330.png)  
Spring为我们提供了处理文件上传MultipartFile的接口

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    public interface MultipartFile extends InputStreamSource {    String getName(); //获取参数名    @Nullable    String getOriginalFilename();//原始的文件名    @Nullable    String getContentType();//内容类型    boolean isEmpty();    long getSize(); //大小    byte[] getBytes() throws IOException;// 获取字节数组    InputStream getInputStream() throws IOException;//以流方式进行读取    default Resource getResource() {        return new MultipartFileResource(this);    }    // 将上传的文件写入文件系统    void transferTo(File var1) throws IOException, IllegalStateException;  // 写入指定path    default void transferTo(Path dest) throws IOException, IllegalStateException {        FileCopyUtils.copy(this.getInputStream(), Files.newOutputStream(dest));    }}

而spring处理文件上传逻辑的具体关键逻辑在`org.springframework.web.multipart.support.StandardMultipartHttpServletRequest#parseRequest`，抄个文件上传demo来进行测试分析

### Spring4

这里我测试了`springboot1.5.20.RELEASE`内置`Spring4.3.23`，具体小版本之间是否有差异这里就不再探究其中关于`org.springframework.web.multipart.support.StandardMultipartHttpServletRequest#parseRequest`的调用也有些不同

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private void parseRequest(HttpServletRequest request) {    try {        Collection<Part> parts = request.getParts();        this.multipartParameterNames = new LinkedHashSet(parts.size());        MultiValueMap<String, MultipartFile> files = new LinkedMultiValueMap(parts.size());        Iterator var4 = parts.iterator();  
            while(var4.hasNext()) {            Part part = (Part)var4.next();            String disposition = part.getHeader("content-disposition");            String filename = this.extractFilename(disposition);            if (filename == null) {                filename = this.extractFilenameWithCharset(disposition);            }  
                if (filename != null) {                files.add(part.getName(), new StandardMultipartHttpServletRequest.StandardMultipartFile(part, filename));            } else {                this.multipartParameterNames.add(part.getName());            }        }  
            this.setMultipartFiles(files);    } catch (Throwable var8) {        throw new MultipartException("Could not parse multipart servlet request", var8);    }}

简单看了下和tomcat之前的分析很像，这里Spring4当中同时也是支持`filename*`格式的![](https://gitee.com/fuli009/images/raw/master/public/20221207172334.png)  
看看具体逻辑

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private String extractFilename(String contentDisposition, String key) {        if (contentDisposition == null) {            return null;        } else {            int startIndex = contentDisposition.indexOf(key);            if (startIndex == -1) {                return null;            } else {                //截取filename=后面的内容                String filename = contentDisposition.substring(startIndex + key.length());                int endIndex;                //如果后面开头是“则截取”“之间的内容                if (filename.startsWith("\"")) {                    endIndex = filename.indexOf("\"", 1);                    if (endIndex != -1) {                        return filename.substring(1, endIndex);                    }                } else {                  //可以看到如果没有“”包裹其实也可以，这和当时陈师分享的其中一个trick是符合的                    endIndex = filename.indexOf(";");                    if (endIndex != -1) {                        return filename.substring(0, endIndex);                    }                }  
                    return filename;            }        }    }

简单测试一波，与心中结果一致![](https://gitee.com/fuli009/images/raw/master/public/20221207172335.png)![](https://gitee.com/fuli009/images/raw/master/public/20221207172337.png)同时由于indexof默认取第一位，因此我们还可以加一些干扰字符尝试突破waf逻辑  
![](https://gitee.com/fuli009/images/raw/master/public/20221207172339.png)如果filename*开头但是spring4当中没有关于url解码的部分![](https://gitee.com/fuli009/images/raw/master/public/20221207172341.png)  
没有这部分会出现什么呢？我们只能自己发包前解码，这样的话如果出现00字节就会报错，报错后![](https://gitee.com/fuli009/images/raw/master/public/20221207172345.png)看起来是spring框架解析header的原因，但是这里报错信息也很有趣将项目地址的绝对路径抛出了，感觉不失为信息收集的一种方式

### Spring5

也是随便来个新的springboot2.6.4的，来看看spring5的，小版本间差异不测了，经过测试发现spring5和spring4之间也是有版本差异处理也有些不同，同样是在`parseRequest`

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private void parseRequest(HttpServletRequest request) {        try {            Collection<Part> parts = request.getParts();            this.multipartParameterNames = new LinkedHashSet(parts.size());            MultiValueMap<String, MultipartFile> files = new LinkedMultiValueMap(parts.size());            Iterator var4 = parts.iterator();  
                while(var4.hasNext()) {                Part part = (Part)var4.next();                String headerValue = part.getHeader("Content-Disposition");                ContentDisposition disposition = ContentDisposition.parse(headerValue);                String filename = disposition.getFilename();                if (filename != null) {                    if (filename.startsWith("=?") && filename.endsWith("?=")) {                        filename = StandardMultipartHttpServletRequest.MimeDelegate.decode(filename);                    }  
                        files.add(part.getName(), new StandardMultipartHttpServletRequest.StandardMultipartFile(part, filename));                } else {                    this.multipartParameterNames.add(part.getName());                }            }  
                this.setMultipartFiles(files);        } catch (Throwable var9) {            this.handleParseFailure(var9);        }  
        }

很明显可以看到这一行`filename.startsWith("=?") &&
filename.endsWith("?=")`，可以看出Spring对文件名也是支持QP编码在上面能看到还调用了一个解析的方法`org.springframework.http.ContentDisposition#parse`，多半就是这里了,那么继续深入下可以看到一方面是QP编码，另一方面也是支持`filename*`,同样获取值是截取`"`之间的或者没找到就直接截取`=`后面的部分![](https://gitee.com/fuli009/images/raw/master/public/20221207172347.png)  
如果是`filename*`后面的处理逻辑就是else分之，可以看出和我们上面分析spring4还是有点区别就是这里只支持`UTF-8/ISO-8859-1/US_ASCII`，编码受限制

  *   *   *   *   *   *   *   *   * 

    
    
    int idx1 = value.indexOf(39);int idx2 = value.indexOf(39, idx1 + 1);if (idx1 != -1 && idx2 != -1) {  charset = Charset.forName(value.substring(0, idx1).trim());  Assert.isTrue(StandardCharsets.UTF_8.equals(charset) || StandardCharsets.ISO_8859_1.equals(charset), "Charset should be UTF-8 or ISO-8859-1");  filename = decodeFilename(value.substring(idx2 + 1), charset);} else {  filename = decodeFilename(value, StandardCharsets.US_ASCII);}

但其实仔细想这个结果是符合RFC文档要求的![](https://gitee.com/fuli009/images/raw/master/public/20221207172350.png)  
接着我们继续后面会继续执行`decodeFilename`![](https://gitee.com/fuli009/images/raw/master/public/20221207172352.png)  
代码逻辑很清晰字符串的解码,如果字符串是否在`RFC 5987`文档规定的Header字符就直接调用baos.write写入

    
    
      
    

|

  *   *   *   * 

    
    
    attr-char     = ALPHA / DIGIT                  / "!" / "#" / "$" / "&" / "+" / "-" / "."                  / "^" / "_" / "`" / "|" / "~"                  ; token except ( "*" / "'" / "%" )  
  
---|---  
如果不在要求这一位必须是`%`然后16进制解码后两位，其实就是url解码，简单测试即可![](https://gitee.com/fuli009/images/raw/master/public/20221207172355.png)

## 参考文章

https://www.ch1ng.com/blog/264.htmlhttps://datatracker.ietf.org/doc/html/rfc6266#section-4.3https://datatracker.ietf.org/doc/html/rfc2231https://datatracker.ietf.org/doc/html/rfc5987#section-3.2.1https://y4tacker.github.io/2022/02/25/year/2022/2/Java%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E5%A4%A7%E6%9D%80%E5%99%A8-%E7%BB%95waf(%E9%92%88%E5%AF%B9commons-
fileupload%E7%BB%84%E4%BB%B6)/https://docs.oracle.com/javaee/7/api/javax/servlet/http/Part.html#getSubmittedFileName
--
http://t.zoukankan.com/summerday152-p-13969452.html#%E4%BA%8C%E3%80%81%E5%A4%84%E7%90%86%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6multipartfile%E6%8E%A5%E5%8F%A3文章来源：乌雲安全

  *   * 

    
    
    文章作者: Y4tacker，欢迎访问作者博客文章链接: https://y4tacker.github.io

如有侵权，请联系删除

 **推荐阅读**

[实战|记一次奇妙的文件上传getshell](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495718&idx=1&sn=e25bcb693e5a50988f4a7ccd4552c2e2&chksm=c04d7718f73afe0e282c778af8587446ff48cd88422701126b0b21fa7f5027c3cde89e0c3d6d&scene=21#wechat_redirect)  
[「 超详细 | 分享
」手把手教你如何进行内网渗透](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495694&idx=1&sn=502c812024302566881bad63e01e98cb&chksm=c04d7730f73afe267fd4ef57fb3c74416b20db0ba8e6b03f0c1fd7785348860ccafc15404f24&scene=21#wechat_redirect)  
[神兵利器 | siusiu-
渗透工具管理套件](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495385&idx=1&sn=4d2d8456c27e058a30b147cb7ed51ab1&chksm=c04d69e7f73ae0f11b382cddddb4a07828524a53c0c2987d572967371470a48ad82ae96e7eb1&scene=21#wechat_redirect)  
[一款功能全面的XSS扫描器](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495361&idx=1&sn=26077792908952c6279deeb2a19ebe37&chksm=c04d69fff73ae0e9f2e03dd8e347f35d660a7fd3d51b0f5e45c8c64afc90c0ee34c4251f9c80&scene=21#wechat_redirect)  
[实战 |
一次利用哥斯拉马绕过宝塔waf](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495331&idx=1&sn=94b63a0ec82de62191f0911a39b63b7a&chksm=c04d699df73ae08b946e4cf53ceea1bc7591dad0ce18a7ccffed33aa52adccb18b4b1aa78f4c&scene=21#wechat_redirect)  
[BurpCrypto:
万能网站密码爆破测试工具](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495253&idx=1&sn=d4c46484a44892ef7235342d2763e6be&chksm=c04d696bf73ae07d0c16cff3317f6eb847df2251a9f2332bbe7de56cb92da53b206cd4100210&scene=21#wechat_redirect)  
[快速筛选真实IP并整理为C段 --
棱眼](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495199&idx=1&sn=74c00ba76f4f6726107e2820daf7817a&chksm=c04d6921f73ae037efe92e051ac3978068d29e76b09cf5b0b501452693984f96baa9436457e4&scene=21#wechat_redirect)  
[自动探测端口顺便爆破工具t14m4t](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495141&idx=1&sn=084e8231c0495e91d1bd841e3f43b61c&chksm=c04d6adbf73ae3cdbb0a4cc754f78228772d6899b94d0ea6bb735b4b5ca03c51e7715b43d0af&scene=21#wechat_redirect)  
[渗透工具｜无状态子域名爆破工具（1秒扫160万个子域）](http://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247495099&idx=1&sn=385764328aff5ec49acddab380721af0&chksm=c04d6a85f73ae393ffab22021839f5baec3802d495c34fb364cbdd9b7cb0cf642851e9527ba7&scene=21#wechat_redirect)  
 **查看更多精彩内容，还请关注** **橘猫学安全：** **每日坚持学习与分享，觉得文章对你有帮助可在底部给点个“** **再看** **”**

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

