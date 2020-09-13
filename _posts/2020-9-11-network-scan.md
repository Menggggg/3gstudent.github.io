---
layout: post
title: 后渗透-内网信息/资产收集
---

[TOC]

>”渗透测试的本质是信息收集“
>
>此篇文章用作介绍拿到Webshell之后的一些针对内网的操作，本系列文章将会持续更新包括但不限于 “权限提升、权限维持、内网潜伏、嗅探钓鱼”等



## 内网信息描述

当从Web端拿到webshell之后，对服务器主机的了解几乎是0，应从内往外一步步的收集服务器的相关信息，首先要对主机当前所处的网络环境进行判断，通常来说分为3种：

- 我是谁

  对机器角色的判断，是指判断已经控制的机器是 **普通Web服务器、开发测试服务器、公共服务器、文件服务器、代理服务器、DNS服务器还是存储服务器**等。具体的判断是通过对机器内的 **主机名、文件、网络连接**等多种情况综合进行的

- 这是哪

  对目前机器所处网络环境的拓扑结构进行分析和判断，是指需要对所处内网进行全面的数据收集及分析整理，绘制出大概的内网整体拓扑结构图，以便后期进行进一步的内网渗透和准确定位内网具体目标，从而完成渗透测试

- 我在哪

  对目前机器所处位置区域的判断，是指判断机器处于网络拓扑中的哪个区域，是在 **DMZ区、办公网，还是核心区核心DB **等位置。当然，这里的区域并不是绝对的，只是一个大概的环境，不同位置的网络环境不一样，区域的界限也不一定明显

  

## 存活主机检测

Cmd:

```cmd
for /l %i in (1,1,255) do @ ping 172.16.80.%i -w 1 -n 1|find /i "ttl="
```

![image-20200913151147025](https://tva1.sinaimg.cn/large/007S8ZIlgy1gip1kdg744j312m04omyf.jpg)

Powershell：

```powershell
$ip = "172.20.10."
for ($i = 1; $i -lt 255; $i ++){
# get each ip
$cur = $ip + $i
# ping once 
ping -n 1 $cur | Out-Null
if ($LASTEXITCODE -eq 0) {
	Write-Host "$cur online"
} else {
	Write-Host "$cur dead"
	}
}
```

![image-20200913151326903](https://tva1.sinaimg.cn/large/007S8ZIlgy1gip1m2n2qfj312s08ewgj.jpg)



## 开放端口检测

1.PowerShell端口扫描器：针对单个IP的多个端口的扫描

```powershell
1..1024 | % {echo ((new-object Net.Sockets.TcpClient).Connect("172.20.10.1",$_)) "Port $_ is open!"} 2>$null
```

2.Test-Netconnection 针对某IP段中单个端口的扫描

```powershell
foreach ($ip in 1..20) {Test-NetConnection -Port 80 -InformationLevel "Detailed" 172.20.10.$ip}
```

`Test-NetConnection`命令是在 **4.0 **版本的PowerShell中才引入的。

3.针对某IP段 & 多个端口的扫描器

```powershell
1..20 | % { $a = $_; 1..1024 | % {echo ((new-object Net.Sockets.TcpClient).Connect("172.20.10.$a",$_)) "Port $_ is open!"} 2>$null}
```

4.针对某IP段 & 指定多个端口的扫描器

```powershell
1..20 | % { $a = $_; write-host "------"; write-host "172.20.10.$a"; 22,53,80,445 | % {echo ((new-object Net.Sockets.TcpClient).Connect("10.0.0.$a",$_)) "Port $_ is open!"} 2>$null}
```

---

命令分解

1）`1..1024` - 创建值为从1到1024的一系列变量

2）`|` - 管道运算符，将上述对象传递给循环体

3）`%` - 在PowerShell中，%是foreach对象的别名，用来开始一个循环。循环体为接下来使用大括号{}括起来的内容

4）`echo` - 将输出打印至屏幕

5）`new-object Net.Sockets.TcpClient` - 新建一个.Net TcpClient类的实例，它允许我们和TCP端口之间建立socket连接

6）`.Connect("10.0.0.100",$_))` - 调用TcpClient类的Connect函数，参数为10.0.0.100和端口$_。其中$_这个变量表示当前对象，即本轮循环中的数字（1..1024）

7）`"Port $_ is open!")` - 当程序发现一个开放的端口时，屏幕打印『Port # is open!』

8）`2>$null` - 告诉PowerShell遇到任何错误都不显示

上述示例中扫描的端口是`1-1024`，但是可以很容易改成如（22..53）、(8000..9000)等端口范围。

---

**常见的端口及服务**

|    21、69     |      FTP\TFTP      |       弱口令\嗅探        |
| :-----------: | :----------------: | :----------------------: |
|      22       |        SSH         |          弱口令          |
|      23       |       telnet       |    弱口令、嗅探、探测    |
|      25       |        SMTP        |           邮件           |
|      53       |        DNS         | 区域传送、dns欺骗、域控  |
|    67、68     |        DHCP        |        劫持、欺骗        |
| 80、443、8080 |      WEB应用       |     弱口令、WEB攻击      |
|  7001、7002   |      weblogic      |     反序列化、弱口令     |
|  8080、8089   |   jboss、jenkins   |     反序列化、弱口令     |
|     9090      |     websphere      |     反序列化、弱口令     |
|      110      |        POP3        |        爆破、嗅探        |
|   139、445    |       samba        | 未授权访问、远程代码执行 |
|      143      |        IMAP        |           爆破           |
|      161      |        SNMP        |      爆破、信息泄露      |
|      389      |        LDAP        |     弱口令、匿名访问     |
|     3389      |        RDP         |    爆破、远程代码执行    |
|     5900      |        VNC         |          弱口令          |
|     5632      |     PcAnywhere     |      嗅探、代码执行      |
|     3306      |       mysql        |          弱口令          |
|     1433      | msssql、sql server |          弱口令          |
|     1521      |       oracle       |          弱口令          |
|     5432      |       pgsql        |          弱口令          |
| 27017、27018  |      mongodb       |        未授权访问        |
|     6379      |       redis        |        未授权访问        |
|     5000      |    sysbase/DB2     |          弱口令          |
|     11211     |     memcached      |        未授权访问        |
|  9200、9300   |   elasticsearch    |       远程代码执行       |
|     2181      |     zookeeper      |        未授权访问        |
|     8069      |       zabbix       |         远程执行         |
|     3690      |        SVN         |         SVN泄露          |
|      873      |       rsync        |         匿名访问         |
|   888、8888   |        宝塔        |     宝塔后门、弱口令     |

## 系统信息

`systeminfo`查看KB号

`wmic qfe get Caption,Description,HotFixID,InstalledOn`查看安装的补丁

**提权辅助网页**（根据补丁查找exp）：

http://bugs.hacking8.com/tiquan/

查看 DNS服务器 



## 查看进程

可以查看进程是否有 mysql，nginx，Apache，redis等敏感服务

`tasklist /svc` or `wmic process list brief`

进程杀软对比工具-avlist https://github.com/gh0stkey/avList/



## 服务信息

`wmic service list brief` 

查看是否开启DHCP或其他第三方服务



## 已安装程序

`wmic product get name,version`

`powershell "Get-WmiObject -class Win32_Product |Select-Object -Property name,version"`

查看已安装程序，判断机器作用及价值，如安装了VMware vSphere Client或者xshell,ftp等，就可以去提取账号口令了



## 查看启动项

`wmic startup get command,caption`

