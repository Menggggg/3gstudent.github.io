---
layout: post
title: 故障诊断平台在渗透测试中的应用技巧
---

## shift后门(替换系统文件)

通过修改`setch.exe`来达到持久化后门

在低版本的windows中，我们可以直接把`C:\Windows\System32\`下`setch.exe`

替换成我们的后门程序

![0x01](https://tva1.sinaimg.cn/large/00831rSTly1gcxuu947nsj31500dmabt.jpg)

