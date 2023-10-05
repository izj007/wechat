#  通过emobile打e-cology前台RCE

原创 摸鱼sec  [ 摸鱼Sec ](javascript:void\(0\);)

**摸鱼Sec** ![]()

微信号 gh_e3d95d1a5b73

功能介绍 摸鱼sec由一群具有共同兴趣的网络安全爱好者组成，致力于发现和利用网络安全领域的漏洞和弱点，以保护用户隐私和数据安全。

____

___发表于_

收录于合集

![]()

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private String checkvalue(Map<String, String> var1) {WeaResultMsg var2 = new WeaResultMsg(false);String var3 = (String)var1.get("em_auth_token"); //请求头 可控String var4 = (String)var1.get("em_auth_ecid"); //请求头 可控String var5 = (String)var1.get("em_auth_stamp"); //请求头 可控if (var1.containsKey("emauthtoken")) {var3 = (String)var1.get("emauthtoken");}  
    if (var1.containsKey("emauthecid")) {var4 = (String)var1.get("emauthecid");}  
    if (var1.containsKey("emauthstamp")) {var5 = (String)var1.get("emauthstamp");}  
    if (StringUtils.isBlank(var3)) {return var2.fail("em_auth_token is empty.").toString();} else if (StringUtils.isBlank(var4)) {return var2.fail("em_auth_ecid is empty.").toString();} else if (StringUtils.isBlank(var5)) {return var2.fail("em_auth_stamp is empty.").toString();} else {try {EMManager var6 = new EMManager();String var7 = (String)EMManager.getEMData().get("accesstoken"); //如果没有集成EMP系统，这里获取出来为nulllogger.debug("em_auth_stamp:" + var1.toString());if (!Util_public.checkStamp(var5)) { //请求头 可控return var2.fail("Token expires or server time is inconsistent.", 1).toString();}  
    String var8 = SHA1.gen(new String[]{var7 + var4 + var5}); //请求头 可控，导致SHA1结果可控，导致绕过登录logger.debug("accesstoken:" + var7 + "##" + var8);if (var3.equals(var8)) {logger.info("登录成功!");return "";}  
    var2.fail("Token verification is inconsistent", 1);logger.error("登录失败:" + var2.getJSONString());} catch (Exception var9) {var9.printStackTrace();var2.fail(var9.getMessage());} return var2.toString();}}

![]()

![]()

此处获取参数用了fastjson进行解析，可以通过\u方式绕过泛微waf 调用到

  * 

    
    
    WEB-INF/classes/com/engine/hrm/cmd/emmanager/GetResourceInfoCmd.class cmd=getResourceListByField

![]()  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    private Map getResourceListByField(List var1, String var2, String var3) throws Exception {HashMap var4 = new HashMap();new ArrayList();new ArrayList();int var7 = Util.getIntValue(var3, 7);  
    try {Map var8 = null;var8 = LanguageService.getLanguageService().getActiveLanguage();List var5 = (List)var8.get("activeLanguageIds");List var6 = (List)var8.get("languageIdentify");ArrayList var9 = new ArrayList();ArrayList var10 = new ArrayList();ResourceComInfo var11 = new ResourceComInfo();ResourceBelongtoComInfo var12 = new ResourceBelongtoComInfo();LocationComInfo var13 = new LocationComInfo();JobTitlesComInfo var14 = new JobTitlesComInfo();DepartmentComInfo var15 = new DepartmentComInfo();SubCompanyComInfo var16 = new SubCompanyComInfo();new JobCallComInfo();RecordSet var18 = new RecordSet();String var19 = "selecthrmresource.id,lastname,pinyinlastname,messagerurl,hrmresource.subcompanyid1,hrmresource.departmentid,mobile,telephone,email,jobtitle,jobcall,joblevel,jobactivitydesc,managerid,status,loginid,account,dsporder,accounttype,belongto,fax,workroom,textfield1,textfield2,textfield3,textfield4,textfield5,birthday,folk,nativeplace,regresidentplace,certificatenum,policy,bememberdate,bepartydate,degree,height,weight,residentplace,homeaddress,tempresidentnumber,companystartdate,workstartdate,workyear,companyworkyear,startdate,probationenddate,enddate,locationid,workcode,mobilecall,sex,seclevel,t1.*,t2.*,t3.* ";LinkedHashMap var20 = new LinkedHashMap();LinkedHashMap var21 = null;ArrayList var22 = new ArrayList();HrmFieldManager var23 = null;String[] var24 = new String[]{"id as t1_id", "id as t2_id", "id as t3_id"};String var25 = var18.getPropValue("hrmFieldSync", "basicinfo");String var26 = var18.getPropValue("hrmFieldSync", "workinfo");String var27 = var18.getPropValue("hrmFieldSync", "personalinfo");HashMap var28 = new HashMap();if (var25.equals("1")) {var28.put("0", -1);} JAVA  
    if (var27.equals("1")) {var28.put("1", 1);}  
    if (var26.equals("1")) {var28.put("2", 3);}  
    boolean var29 = false;Iterator var30 = var28.entrySet().iterator();  
    label183:while(var30.hasNext()) {Map.Entry var31 = (Map.Entry)var30.next();int var62 = (Integer)var31.getValue();int var32 = Util.getIntValue((String)var31.getKey());var23 = new HrmFieldManager("HrmCustomFieldByInfoType", var62);var23.getCustomFields();  
    while(true) {do {do {do {if (!var23.next()) {continue label183;}} while(!var23.isUse());} while(var23.getHtmlType().equals("6"));} while(var23.getHtmlType().equals("3") && (var23.getType() == 161 || var23.getType() == 162));  
    if (!var23.isBaseField(var23.getFieldname())) {if (var24[var32].length() > 0) {var24[var32] = var24[var32] + ",";}  
    if (var23.getFieldname().indexOf("field") != -1) {var24[var32] = var24[var32] + var23.getFieldname() + " as t" + var32 + "_" + var23.getFieldname();} else {var24[var32] = var24[var32] + var23.getFieldname();}  
    if (var20.get(var23.getFieldname()) == null) {var21 = new LinkedHashMap();var20.put("t" + var32 + "_" + var23.getFieldname(), var23.getLable());var21.put("fieldname", var23.getFieldname());var21.put("fieldlable", var23.getLable());var21.put("type", var23.getType() + "");var21.put("fieldhtmltype", var23.getHtmlType());var21.put("fieldId", var23.getFieldid() + "");var21.put("dmlurl", var23.getDmrUrl());var21.put("tempfieldname", "t" + var32 + "_" + var23.getFieldname());if (var32 == 0) {var21.put("cusdataname", var23.getFieldname());} else {var21.put("cusdataname", var32 + "_" + var23.getFieldname());}  
    var22.add(var21);}}}}  
    var19 = var19 + " from HrmSubcompany,HrmDepartment,HrmResource left join (SELECT " + var24[0] + " FROM cus_fielddata WHEREscope='HrmCustomFieldByInfoType' AND scopeId=-1) t1 on hrmresource.id=t1_id left join (SELECT " + var24[1] + " FROMcus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=1) t2 on hrmresource.id=t2_id left join (SELECT " + var24[2]+ " FROM cus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=3) t3 on hrmresource.id=t3_id where (status = 0 orstatus = 1 or status = 2 or status = 3) and (HrmSubcompany.canceled != '1' or HrmSubcompany.canceled is null) and(Hrmdepartment.canceled != '1' or Hrmdepartment.canceled is null) and (HrmSubcompany.id = hrmresource.subcompanyid1 andHrmdepartment.id = hrmresource.departmentid) ";if (var1 != null && var1.size() > 0) {var30 = null;  
    for(int var65 = 0; var65 < var1.size(); ++var65) {Map var63 = (Map)var1.get(var65);String var67 = Util.null2String(var63.get("field_name"));String var33 = Util.null2String(var63.get("field_value"));if (var67.length() != 0) {var9.add(var33);var19 = var19 + " and " + var67 + "= ? "; // var67从post过来的内容取得，直接拼接导致sql注入}}}  
    boolean var64 = var18.executeQuery(var19, new Object[]{var9});var18.writeLog("查询人员接口[GetResourceListByField]sql=====" + var19 + JSONObject.toJSONString(var9));  
    while(var18.next()) {HashMap var66 = new HashMap();ArrayList var68 = new ArrayList();ArrayList var69 = new ArrayList();ArrayList var34 = new ArrayList();ArrayList var35 = new ArrayList();int var36 = var18.getInt("id");User var37 = new User();var37.setUid(var36);var37.setLanguage(var7);String[] var38 = Util.TokenizerString2(var12.getBelongtoids("" + var36), ",");String[] var39 = var38;int var40 = var38.length;  
    String var42;for(int var41 = 0; var41 < var40; ++var41) {var42 = var39[var41];var34.add(var11.getDepartmentID(var42));var35.add(StringUtil.vString(Util.toDecimalDigits(var11.getDsporder(var42), 2), "0"));}  
    var66.put("ID", var36);var66.put("Name", Util.formatMultiLang(var18.getString("lastname"), var3));var66.put("certificatenum", Util.formatMultiLang(var18.getString("certificatenum"), var3));String var70 = var18.getString("jobtitle");String var71 = var14.getJobTitlesname(var70);var66.put("title", Util.formatMultiLang(var71, var3));Iterator var72 = var5.iterator();  
    while(var72.hasNext()) {var42 = (String)var72.next();  
    for(int var43 = 0; var43 < var6.size(); ++var43) {if (((Map)var6.get(var43)).get(var42) != null) {HashMap var44 = new HashMap();HashMap var45 = new HashMap();var44.put("lang_tag", ((Map)var6.get(var43)).get(var42));var44.put("set_value", Util.formatMultiLang(var18.getString("lastname"), var42));var45.put("lang_tag", ((Map)var6.get(var43)).get(var42));var45.put("set_value", Util.formatMultiLang(var71, var42));var68.add(var44);var69.add(var45);}}}  
    HashMap var73 = new HashMap();String[] var74 = new String[]{"joblevel", "jobcall", "jobactivitydesc", "fax", "workroom", "textfield1", "textfield2","textfield3", "textfield4", "textfield5", "birthday", "folk", "nativeplace", "regresidentplace", "policy", "bememberdate","bepartydate", "degree", "height", "weight", "residentplace", "homeaddress", "tempresidentnumber", "companystartdate",com.alibaba.fastjson."workstartdate", "workyear", "companyworkyear", "startdate", "probationenddate", "enddate"};String[] var75 = new String[]{"jobcall", "jobactivity", "educationlevel", "usekind"};String[] var76 = var74;int var78 = var74.length;  
    int var46;String var47;String var48;for(var46 = 0; var46 < var78; ++var46) {var47 = var76[var46];var48 = Util.null2String(var18.getString(var47));var73.put(var47, var48);}  
    var76 = var75;var78 = var75.length;  
    for(var46 = 0; var46 < var78; ++var46) {var47 = var76[var46];var48 = Util.null2String(var18.getString(var47));var73.put(var47 + "_id", var48);var73.put(var47 + "_name", this.getFieldValue(var47, var48));}  
    var66.put("lang_data", var68);var66.put("title_lang_data", var69);var66.put("PYName", var18.getString("pinyinlastname"));var66.put("HeaderURL", var18.getString("messagerurl"));var66.put("resourceimageid", var18.getString("resourceimageid"));String var77 = var18.getString("subcompanyid1");var66.put("SubCompanyID", var77);var66.put("SubCompanyName", Util.formatMultiLang(var16.getSubCompanyname(var77), var3));String var79 = var18.getString("departmentid");var66.put("DepartmentID", var79);var66.put("DepartmentName", Util.formatMultiLang(var15.getDepartmentname(var79), var3));var66.put("subDeptIds", var34);var66.put("subOrders", var35);var66.put("mobile", Util.null2String(var18.getString("hrmresource", "mobile", true, true)));var66.put("tel", Util.null2String(var18.getString("hrmresource", "telephone", true, true)));var66.put("email", Util.null2String(var18.getString("hrmresource", "email", true, true)));var66.put("workcode", Util.formatMultiLang(var18.getString("workcode"), var3));String var80 = var18.getString("managerid");var66.put("managerID", var80);var66.put("managerName", Util.formatMultiLang(var11.getLastname(var80), var3));var66.put("status", var18.getString("status"));var66.put("statusName", this.getStatusName(var18.getString("status"), var7));var47 = Util.null2String(var18.getString("loginid"));var66.put("loginid", var47);var66.put("seclevel", var18.getString("seclevel"));var66.put("mobileshowtype", var18.getString("mobileshowtype"));var66.put("showorder", Util.toDecimalDigits(var18.getString("dsporder"), 2));var48 = var18.getString("locationid");var66.put("locationID", var48);var66.put("locationName", Util.formatMultiLang(var13.getLocationname(var48), var3));var66.put("mobilecall", var18.getString("hrmresource", "mobilecall", true, true));var66.put("sex", var18.getString("sex"));var66.put("seclevel", var18.getString("seclevel"));var66.put("accounttype", Util.null2String(var18.getString("accounttype")));var66.put("mainID", Util.null2String(var18.getString("belongto")));ArrayList var49 = new ArrayList();HashMap var50 = null;Iterator var51 = var73.entrySet().iterator();  
    while(var51.hasNext()) {Map.Entry var52 = (Map.Entry)var51.next();var50 = new HashMap();var50.put("id", var52.getKey());var50.put("name", var52.getKey());var50.put("value", Util.formatMultiLang((String)var52.getValue(), var3));selecthrmresource.id,lastname,pinyinlastname,messagerurl,hrmresource.subcompanyid1,hrmresource.departmentid,mobile,telephone,email,jobtitle,jobcall,joblevel,jobactivitydesc,managerid,status,loginid,account,dsporder,accounttype,belongto,fax,workroom,textfield1,textfield2,textfield3,textfield4,textfield5,birthday,folk,nativeplace,regresidentplace,certificatenum,policy,bememberdate,bepartydate,degree,height,weight,residentplace,homeaddress,tempresidentnumber,companystartdate,workstartdate,workyear,companyworkyear,startdate,probationenddate,enddate,locationid,workcode,mobilecall,sex,seclevel,t1.,t2.,t3.* from HrmSubcompany,HrmDepartment,HrmResource left join(SELECT id as t1_id FROM cus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=-1) t1 on hrmresource.id=t1_id left join(SELECT id as t2_id FROM cus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=1) t2 on hrmresource.id=t2_id left join(SELECT id as t3_id FROM cus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=3) t3 on hrmresource.id=t3_id where(status = 0 or status = 1 or status = 2 or status = 3) and (HrmSubcompany.canceled != '1' or HrmSubcompany.canceled is null) and(Hrmdepartment.canceled != '1' or Hrmdepartment.canceled is null) and (HrmSubcompany.id = hrmresource.subcompanyid1 andHrmdepartment.id = hrmresource.departmentid) and 控制得注入点='1'最后上面可控点如上，通过mssqlWEB-INF/classes/weaver/conn/RecordSet.class,有检测不允许有;和--,可以使用mssql的一些技巧，绕过进行堆叠执行。var49.add(var50);}  
    var51 = var22.iterator();  
    while(var51.hasNext()) {Map var81 = (Map)var51.next();var50 = new HashMap();int var53 = Util.getIntValue((String)var81.get("type"));int var54 = Util.getIntValue((String)var81.get("fieldhtmltype"));int var55 = Util.getIntValue((String)var81.get("fieldId"));String var56 = (String)var81.get("fieldname");String var57 = (String)var81.get("dmlurl");String var58 = (String)var81.get("tempfieldname");String var59 = (String)var81.get("cusdataname");String var60 = Util.null2String(var18.getString(var58));if (var60.length() > 0) {var60 = var23.getFieldvalue((HttpSession)null, var37, (String)null, (String)null, var55, var54, var53, var60, 0, var56);}  
    var50.put("id", var55);var50.put("name", var59);var50.put("value", Util.formatMultiLang(var60, var3));var49.add(var50);}  
    var66.put("cusData", var49);var10.add(var66);}  
    if (var64) {var4.put("errcode", 0);var4.put("errmsg", "ok");var4.put("data", var10);} else {var18.writeLog("批量获取用户信息出现异常：" + var19);var4.put("errcode", -5);var4.put("errmsg", "" + SystemEnv.getHtmlLabelName(10005242, ThreadVarLanguage.getLang()) + "");}} catch (Exception var61) {this.writeLog(var61);var4.put("errcode", -5);var4.put("errmsg", "" + SystemEnv.getHtmlLabelName(10005243, ThreadVarLanguage.getLang()) + "" + var61.getMessage());}  
    return var4;}

  *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    selecthrmresource.id,lastname,pinyinlastname,messagerurl,hrmresource.subcompanyid1,hrmresource.departmentid,mobile,telephone,email,jobtitle,jobcall,joblevel,jobactivitydesc,managerid,status,loginid,account,dsporder,accounttype,belongto,fax,workroom,textfield1,textfield2,textfield3,textfield4,textfield5,birthday,folk,nativeplace,regresidentplace,certificatenum,policy,bememberdate,bepartydate,degree,height,weight,residentplace,homeaddress,tempresidentnumber,companystartdate,workstartdate,workyear,companyworkyear,startdate,probationenddate,enddate,locationid,workcode,mobilecall,sex,seclevel,t1.,t2.,t3.* from HrmSubcompany,HrmDepartment,HrmResource left join(SELECT id as t1_id FROM cus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=-1) t1 on hrmresource.id=t1_id left join(SELECT id as t2_id FROM cus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=1) t2 on hrmresource.id=t2_id left join(SELECT id as t3_id FROM cus_fielddata WHERE scope='HrmCustomFieldByInfoType' AND scopeId=3) t3 on hrmresource.id=t3_id where(status = 0 or status = 1 or status = 2 or status = 3) and (HrmSubcompany.canceled != '1' or HrmSubcompany.canceled is null) and(Hrmdepartment.canceled != '1' or Hrmdepartment.canceled is null) and (HrmSubcompany.id = hrmresource.subcompanyid1 andHrmdepartment.id = hrmresource.departmentid) and 控制的注入点='1'

最后上面可控点如上，通过mssql

WEB-
INF/classes/weaver/conn/RecordSet.class,有检测不允许有;和--,可以使用mssql的一些技巧，绕过进行堆叠执行。

![]()

![]()

emobile有接口直接http打，你们自己体会，这里就不给poc了![](https://res.wx.qq.com/t/wx_fed/we-
emoji/res/v1.3.10/assets/newemoji/Yellowdog.png)

 **仅供安全研究与学习之用，如用于其他用途，由使用者承担全部法律及连带责任，与本公众号无关。**

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

