- 1、 创建进程时候隐藏窗口

```
//可以在startinfo里面指定
si.wShowWindow=SW_HIDE;
si.dwFlags=STARTF_USESHOWWINDOW
```

//也可以在`createflags`里面指定

![title](https://leanote.com/api/file/getImage?fileId=5f8d00f4ab6441260200015f)


- 2、 [MSSQL使用CLR程序集来执行命令](https://xz.aliyun.com/t/6682)
这里面有命令执行的CLR程序集。
更多功能的程序集，如文件下载可以参考此github项目：[MSSQL-Fileless-Rootkit-WarSQLKit](https://github.com/mindspoof/MSSQL-Fileless-Rootkit-WarSQLKit)

    - [MSSQL安全审计文件执行Rootkit-WarSQLKit](https://www.cnblogs.com/a00ium/p/13800496.html)
    - [MSSQL Fileless Rootkit - WarSQLKit](http://eyupcelik.com.tr/guvenlik/493-mssql-fileless-rootkit-warsqlkit)


- 3、[MSSQL_SQL_BYPASS_WIKI](https://github.com/Hacker-One/MSSQL_SQL_BYPASS_WIKI)
[mssql_getshell的一些总结](http://120.24.64.98/2020/10/12/mssql%E8%8E%B7%E5%8F%96%E6%9D%83%E9%99%90%E6%80%BB%E7%BB%93/#more)