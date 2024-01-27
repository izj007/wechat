#  将IDA与VS Code完美结合，插件大放异彩！

[ 二进制磨剑 ](javascript:void\(0\);)

**二进制磨剑** ![]()

微信号 pyable

功能介绍 安全技术分享

____

___发表于_

# 🚀 将IDA与VS Code完美结合，插件大放异彩！🚀

你是否曾经在IDA中编写Python脚本时感到不便？是否梦想着一个能够让你在Visual Studio Code中编写、执行、调试IDA
Python脚本的神器？现在，这个梦想成真啦！让我们一起来探索这个名为IDACode的神秘宝藏吧！

## 项目介绍

IDACode是一个让你能够在IDA环境中轻松执行和调试Python脚本的扩展工具，而且你不需要离开Visual Studio
Code！只需一键，你便可以在这两个强大工具之间无缝切换。要获取VS Code扩展，可以前往市场。

但是，请注意！IDACode还处于早期开发阶段，虫子在所难免。如果你遇到了什么问题，不要犹豫，快去开个新Issue吧！

## 特点和优势

  *  **速度** ：快速创建和执行脚本。
  *  **调试** ：随时都可附加Python调试器。
  *  **兼容性** ：IDACode不需要你特别修改脚本，所有脚本都可以不用改动直接在IDA中执行。
  *  **模块化** ：IDACode并不依赖于繁复的线程同步安全包装器，这让你可以随时从任何路径导入任何模块。它通过与IDA主线程同步脚本执行线程来避免性能和意外问题。
  *  **同步** ：IDACode利用`debugpy`进行通信，自然地与VS Code的输出面板同步。

IDACode同时支持Python 2和Python 3！

## 应用场景

IDACode适用于需要在IDA中使用Python脚本进行逆向工程分析的场合，尤其是当你希望利用Visual Studio
Code强大的编辑和调试功能时。它对安全研究人员、恶意软件分析师、逆向工程师等领域的专家来说将是一款不可或缺的工具。

## 安装和使用方法

  * 确保使用正确的Python版本。
  * 安装依赖：`python -m pip install --user debugpy tornado`。
  * 克隆此仓库或从这里下载发布包。将所有文件复制到IDA的插件目录。
  * 配置你的环境设置，编辑`idacode_utils/settings.py`。
  * 通过点击插件菜单中的`IDACode`来启动插件。

VS Code扩展在市场上可用。请参考扩展的README来配置扩展。

## 使用例子

### IDA

点击插件菜单中的`IDACode`，你会看到下面的文本：

    
    
    IDACode listening on 127.0.0.1:7065  
    

### VS Code

在VS Code中，版本0.2.0起，IDACode支持"保存时执行"功能，默认启用。VS
Code将在你保存当前文档时自动在IDA中执行你的脚本，例如使用CTRL+S。你可以在设置中禁用此行为。

你可以使用以下4个命令：![]()一旦你打开了一个文件夹，并在VS Code提示时指定了文件夹，你就可以连接到IDA。你可以通过执行`Connect to
IDA`或者`Connect and attach a debugger to
IDA`来实现。请记住，一旦开始调试会话，直到重启IDA前都是永久的。一旦调试器启动，你不能更改工作区文件夹。确保工作区文件夹是你主脚本所在的文件夹。连接后，你就可以选择`Execute
script in IDA`。

## 总结

IDACode是一个强大的插件，它将IDA的逆向工程能力与VS
Code的编辑和调试功能完美整合。它的快速、兼容和模块化特性使得工作流程变得更加流畅。无论你是逆向工程新手还是老手，IDACode都是你的得力助手。

#标签 #IDA #Python #VisualStudioCode #调试 #逆向工程 #插件 #自动化

项目地址：点击阅读原文

  

预览时标签不可点

微信扫一扫  
关注该公众号

继续滑动看下一个

# 将IDA与VS Code完美结合，插件大放异彩！

[ 二进制磨剑 ](javascript:void\(0\);)

轻触阅读原文

![]()

二进制磨剑

赞 分享 在看

向上滑动看下一个

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看 分享 留言

