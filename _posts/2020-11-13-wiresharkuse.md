---
layout: post
title: Wireshark过滤语法小记
---

> 因最近要准备 “山东省信息安全管理与评估大赛“ ，比赛中我们会进行流量监控，我们通过wireshark查看流量包对我们哪里进行了攻击并且可以对其溯源，然后防御并且编写脚本得分。

Wireshark过滤支持比较运算符、逻辑运算符，内容过滤时还能使用位运算，但是不支持`glob`语法显得就很鸡肋，希望官方以后会加入此功能。

## Comparing operators（比较运算符）

| 英文写法 | C语言写法 |   含义   |
| :------: | :-------: | :------: |
|    eq    |    ==     |   等于   |
|    ne    |    !=     |  不等于  |
|    gt    |     >     |   大于   |
|    lt    |     <     |   小于   |
|    ge    |    >=     | 大于等于 |
|    le    |    <=     | 小于等于 |

## Logical expressions（逻辑运算符）

| 英文写法 | C语言写法 |   含义   |
| :------: | :-------: | :------: |
|   and    |    &&     |  逻辑与  |
|    or    |   \|\|    |  逻辑或  |
|   xor    |    ^^     | 逻辑异或 |
|   not    |    ！     |  逻辑非  |

## 过滤语法

过滤IP地址

```
1) ip.addr eq 192.168.1.1 //只显示源/目的IP为192.168.1.1的数据包
2) not ip.src eq 192.168.1.1 //不显示源IP为1.1.1.1的数据包
3) ip.src eq 1.1.1.1 or ip.dst eq 1.1.1.2 //只显示源IP为1.1.1.1或目的IP为1.1.1.2的数据包
```

过滤端口

```
1) tcp.port eq 80 //显示TCP 80端口
4) tcp.port eq 80 or udp.port eq 80 //显示TCP 80端口或UDP 80端口
5) tcp.dstport eq 80 //只显示TCP协议的目标端口80
6) tcp.srcport eq 80 //只显示TCP协议的来源端口80
8) tcp.port >= 1 and tcp.port <= 80 //过滤端口范围
```

过滤Mac地址

```
1) eth.dst eq MAC地址 //显示目标MAC
2) eth.src eq MAC地址 //显示来源MAC
3) eth.addr eq MAC地址 //显示来源MAC和目标MAC都等于MAC地址
```

过滤http协议

```
1) http.request.method eq "GET" //显示http get请求
2) http.request.method eq "POST" //显示http post请求
3) http.host eq "www.baidu.com"
4) http.request.uri.query eq "id=1"//显示URL参数中含有id=1
5) http and ip.addr eq 10.16.27.203 //显示源地址为10.16.27.203的http协议
6）http.request.uri eq "/www/c.php" //显示访问“/www/c.php”的请求
```