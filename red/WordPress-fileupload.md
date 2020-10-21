# WordPress评论插件wpDiscuz 任意文件上传漏洞

> 2020-08

## #漏洞描述

wpDiscuz插件会使用`mime_content_type`函数来获取MIME类型，但是该函数在获取MIME类型是通过文件的十六进制起始字节来判断，所以只要文件头符合图片类型即可

## #影响范围

wpDiscuz `v7.0.0-v7.0.4`

## #环境搭建
https://cn.wordpress.org/latest-zh_CN.zip

https://downloads.wordpress.org/plugin/wpdiscuz.7.0.3.zip

管理员登录，在插件模块安装wpdiscuz插件，即可看到评论界面变成这个样子，点击图片的小标可上传图片

![1](https://tva1.sinaimg.cn/large/007S8ZIlly1gjw8xds9zlj31c20q8wi1.jpg)

![2](https://tva1.sinaimg.cn/large/007S8ZIlly1gjw93cci50j31qx0u0auv.jpg)



## #POC

```python
import requests
import re
import sys

class wpdiscuz():
    def __init__(self):
        self.s = requests.session()
        self.s.headrs = {
            "User-Agent":
            "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36 Edg/80.0.361.66"
        }
        self.nonce = ""
        self.state = False

    def check(self, url):
        res = self.s.get(url=url)

        pat1 = "wpdiscuz/themes/default/style\.css\?ver=(.*?)'"
        reSearch1 = re.search(pat1, res.text)
        if reSearch1 == None:
            print("%s 评论插件不存在任意文件漏洞" % url) 
            return
        mess = reSearch1.group(0)
        version = reSearch1.group(1)
        # 判断版本
        vers = version.split(".")
        if (len(vers) == 3):
            if int(vers[0]) == 7:
                if int(vers[2]) <= 4:
                    print(url + " 存在任意文件上传漏洞 wpdiscuz版本为 %s" % version)
                    self.state = True

        if self.state == True:
            # nonce
            pat2 = '"wmuSecurity":"(.*?)"'
            reSearch2 = re.search(pat2, res.text)
            nonce = reSearch2.group(1)
            self.nonce = nonce
        else:
            print("%s 评论插件不存在任意文件漏洞" % url)    

    def exp(self, url, project, filepath):
        pass

if __name__ == "__main__":
    wpdiscuz = wpdiscuz()
    url = sys.argv[1]
    print("检测漏洞结果:")
    wpdiscuz.check(url)
```