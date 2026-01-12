---
layout: post
title:  "Commentary"
date:   2026-01-12 01:04:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-commentary/
---

**Title**: Commentary

**Category**: Web

**Author**: mel

**Description**: You're currently speaking to my favorite **host** right now (`ctf.rusec.club`), but who's to say you even had to speak with one?

Sometimes, the treasure to be found is just bloat that people forgot to remove.

Grab the IP of the server:
```shell
dig +short ctf.rusec.club  
5.161.91.13

# OR

ping -c 1 ctf.rusec.club  
PING ctf.rusec.club (5.161.91.13) 56(84) bytes of data.  
64 bytes from static.13.91.161.5.clients.your-server.de (5.161.91.13): icmp_seq=1 ttl=128 time=179 ms  
  
--- ctf.rusec.club ping statistics ---  
1 packets transmitted, 1 received, 0% packet loss, time 0ms  
rtt min/avg/max/mdev = 178.835/178.835/178.835/0.000 ms
```

Curl it with http to get the default nginx page with an added flag:
```shell
curl http://5.161.91.13  
<!DOCTYPE html>  
<html>  
<head>  
<title>Welcome to nginx!</title>  
<style>  
html { color-scheme: light dark; }  
body { width: 35em; margin: 0 auto;  
font-family: Tahoma, Verdana, Arial, sans-serif; }  
</style>  
</head>  
<body>  
<h1>Welcome to nginx!</h1>  
<p>If you see this page, the nginx web server is successfully installed and  
working. Further configuration is required.</p>  
  
<p>For online documentation and support please refer to  
<a href="http://nginx.org/">nginx.org</a>.<br/>  
Commercial support is available at  
<a href="http://nginx.com/">nginx.com</a>.</p>  
  
<!-- you found me :3 --!>  
<!-- RUSEC{truly_the_hardest_ctf_challenge} --!>  
  
<p><em>Thank you for using nginx.</em></p>  
</body>  
</html>
```

`RUSEC{truly_the_hardest_ctf_challenge}`