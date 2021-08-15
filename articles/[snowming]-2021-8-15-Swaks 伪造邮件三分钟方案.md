1、买近似域名绑定 QQ 邮箱的`域名邮箱`。

2、 确定邮件模板，下载 eml。可以通过 foxmail 对 eml 再次编辑或者修改纯文本。

3 、sh 脚本跑 swaks 批量发。

``` shell
#!/bin/bash								
 sleep 10s 
./swaks	--header-X-Mailer	whu-online.cn	--to	zhangsan@whu.com.cn	--from	OA@whu-online.cn	--data	./payload.eml
 sleep 10s 
./swaks --header-X-Mailer whu-online.cn --to lisi@whu.com.cn --from OA@whu-online.cn --data ./payload.eml
```

注1：whu-online.cn 为伪造的域名域，也可以写 foxmail.com，qq.com 等。
注2：我就是通过这种弱智 sh 脚本批量发的，怎么写成这种 n 行的 sh 脚本呢？写一个 python 脚本把批量发的目标依次拼接成 shell 语句。