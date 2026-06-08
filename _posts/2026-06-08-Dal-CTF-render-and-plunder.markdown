---
layout: post
title:  "Render and Plunder"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-render-and-plunder/
---

**Title**: Render and Plunder

**Author**: ASmilyBun

**Category**: Defense

**Description**: I wrote a user-profile service with a nice renderer for myself. Surely it's secure, right? Take a look and patch any vulnerabilities while keeping the app fully functional. `https://defense.dalctf2026.com/`

For this one I get `server.py`:
```python
import os
import json
import datetime

from flask import Flask, request, jsonify
from jinja2 import Template
import jwt

app = Flask(__name__)

JWT_SECRET = os.environ["JWT_SECRET"]

_raw = os.environ["USERS_JSON"]
USERS = json.loads(_raw)
ID_TO_USER = {v["id"]: k for k, v in USERS.items()}


@app.route("/health")
def health():
    return jsonify({"status": "ok"}), 200


@app.route("/login", methods=["POST"])
def login():
    data = request.get_json(silent=True) or {}
    username = data.get("username", "")
    password = data.get("password", "")

    user = USERS.get(username)
    if user and user["password"] == password:
        token = jwt.encode(
            {
                "username": username,
                "user_id": user["id"],
                "exp": datetime.datetime.utcnow() + datetime.timedelta(hours=1),
            },
            JWT_SECRET,
            algorithm="HS256",
        )
        return jsonify({"token": token, "user_id": user["id"]})
    return jsonify({"error": "Invalid credentials"}), 401


def _decode_token(token: str) -> dict:
    return jwt.decode(token, JWT_SECRET, algorithms=["HS256"])


def _require_auth():
    token = request.headers.get("Authorization", "").replace("Bearer ", "")
    return _decode_token(token)


@app.route("/api/users/<user_id>/data", methods=["GET"])
def get_user_data(user_id: str):
    try:
        payload = _require_auth()
    except Exception:
        return jsonify({"error": "Unauthorized"}), 401

    username = ID_TO_USER.get(user_id)
    if not username:
        return jsonify({"error": "User not found"}), 404

    return jsonify({"profile": USERS[username]["profile"]}), 200


@app.route("/api/users/<user_id>/data", methods=["PUT"])
def update_user_data(user_id: str):
    try:
        payload = _require_auth()
    except Exception:
        return jsonify({"error": "Unauthorized"}), 401

    username = ID_TO_USER.get(user_id)
    if not username:
        return jsonify({"error": "User not found"}), 404

    data = request.get_json(silent=True) or {}
    bio = data.get("bio", "")
    USERS[username]["profile"]["bio"] = bio
    return jsonify({"message": "Profile updated", "bio": bio}), 200


@app.route("/render", methods=["POST"])
def render_report():
    try:
        payload = _require_auth()
    except Exception:
        return jsonify({"error": "Unauthorized"}), 401

    data = request.get_json(silent=True) or {}
    name = data.get("name", "User")

    template_source = (
        f"Hello {name}! Here is your report for user {payload.get('username', '?')}."
    )
    result = Template(template_source).render()

    return jsonify({"report": result}), 200


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

There is SSTI in `template_source`:
```python
template_source = (
        f"Hello {name}! Here is your report for user {payload.get('username', '?')}."
    )
    result = Template(template_source).render()
```

I changed it to:
```python
template_source = Template("Hello {{ name }}! Here is your report for user {{ username }}.").render(name=name, username=payload.get('username', '?'))
```

Then I change:
```python
return jsonify({"report": result}), 200
```

I changed it to:
```python
return jsonify({"report": template_source}), 200
```
This is just so `result` is no longer undefined. 

`dalctf{W1ll_Y0u_ID0R_Moi_SSTI?}`