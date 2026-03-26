---
layout: post
title:  "Web Challenges"
date:   2026-03-25 20:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /tamuCTF-2026-web/
---
* TOC
{:toc}

## bad-apple

**Category**: web

**Author**: `moveslow`

**Description**: funny touhou reference [https://bad-apple.tamuctf.com](https://bad-apple.tamuctf.com) 

I get the challenge files:
```shell
cat httpd-append.conf
<VirtualHost *:80>
    WSGIScriptAlias / /srv/app/wsgi_app.py

    <Directory /srv/app>
        Require all granted
    </Directory>

    Alias /browse /srv/http/uploads
    <Directory /srv/http/uploads>
        Options +Indexes
        DirectoryIndex disabled
        IndexOptions FancyIndexing FoldersFirst NameWidth=* DescriptionWidth=* ShowForbidden
        AllowOverride None
        Require all granted

        <FilesMatch "\.gif$">
            AuthType Basic
            AuthName "Admin Area"
            AuthUserFile /srv/http/.htpasswd
            Require valid-user
        </FilesMatch>
    </Directory>
</VirtualHost>
```

`wsgi_app.py`:
```python
from flask import Flask, render_template, request, redirect, url_for, send_from_directory, make_response, jsonify
import os
import subprocess
import uuid
from werkzeug.utils import secure_filename

BASE_DIR = os.path.dirname(os.path.abspath(__file__))

app = Flask(__name__, template_folder=os.path.join(BASE_DIR, 'templates'),  static_folder='/srv/http/static')

UPLOAD_FOLDER = '/srv/http/uploads'
FRAMES_BASE = '/srv/http/static/frames'

os.makedirs(UPLOAD_FOLDER, exist_ok=True)
os.makedirs(FRAMES_BASE, exist_ok=True)

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024

@app.errorhandler(413)
def request_entity_too_large(error):
    return jsonify({'success': False, 'error': 'File too large. Maximum size is 16MB.'}), 413

def get_user_id():
    user_id = request.cookies.get('user_id')
    if not user_id:
        user_id = str(uuid.uuid4())
    return user_id

def extract_frames(input_path, output_dir, gif_name):
    os.makedirs(output_dir, exist_ok=True)
    
    width = 120
    
    cmd = [
        'ffmpeg', '-i', input_path,
        '-vf', f'fps=10,scale={width}:-1:flags=lanczos,palettegen',
        '-y', f'{output_dir}/palette.png'
    ]
    subprocess.run(cmd, capture_output=True)
    
    cmd = [
        'ffmpeg', '-i', input_path,
        '-i', f'{output_dir}/palette.png',
        '-lavfi', f'fps=10,scale={width}:-1:flags=lanczos[x];[x][1:v]paletteuse',
        '-y', f'{output_dir}/output.gif'
    ]
    subprocess.run(cmd, capture_output=True)
    
    frame_count = 0
    cmd = [
        'ffmpeg', '-i', f'{output_dir}/output.gif',
        f'{output_dir}/frame_%04d.png'
    ]
    subprocess.run(cmd, capture_output=True)
    
    for f in os.listdir(output_dir):
        if f.startswith('frame_') and f.endswith('.png'):
            frame_count += 1
    
    return frame_count

@app.route('/')
def index():
    user_id = get_user_id()
    
    view_gif = request.args.get('view')
    view_user_id = request.args.get('view_user_id', user_id)
    view_frames = []
    
    if view_gif:
        view_frames_dir = os.path.join(FRAMES_BASE, view_user_id, view_gif)
        if os.path.exists(view_frames_dir):
            view_frames = sorted([f for f in os.listdir(view_frames_dir) if f.startswith('frame_') and f.endswith('.png')])
    
    default_frames_dir = os.path.join(FRAMES_BASE, 'shared', 'bad-apple')
    default_frames = []
    if os.path.exists(default_frames_dir):
        default_frames = sorted([f for f in os.listdir(default_frames_dir) if f.startswith('frame_') and f.endswith('.png')])
    
    user_frames_dir = os.path.join(FRAMES_BASE, user_id)
    gifs = []
    if os.path.exists(user_frames_dir):
        for gif_name in os.listdir(user_frames_dir):
            gif_path = os.path.join(user_frames_dir, gif_name)
            if os.path.isdir(gif_path):
                frames = [f for f in os.listdir(gif_path) if f.startswith('frame_') and f.endswith('.png')]
                gifs.append({'name': gif_name, 'frame_count': len(frames)})
    
    response = make_response(render_template('index.html', 
        user_id=user_id, 
        default_gif='bad-apple',
        default_frames=default_frames,
        gifs=gifs,
        view_gif=view_gif,
        view_user_id=view_user_id,
        view_frames=view_frames))
    response.set_cookie('user_id', user_id)
    return response

@app.route('/upload', methods=['POST'])
def upload():
    if 'file' not in request.files:
        return jsonify({'success': False, 'error': 'No file uploaded'}), 400
    
    file = request.files['file']
    if file.filename == '':
        return jsonify({'success': False, 'error': 'No file selected'}), 400
    
    if file:
        filename = secure_filename(file.filename)
        user_id = request.cookies.get('user_id')
        if not user_id:
            user_id = str(uuid.uuid4())
        
        user_dir = os.path.join(app.config['UPLOAD_FOLDER'], user_id)
        os.makedirs(user_dir, exist_ok=True)
        
        filepath = os.path.join(user_dir, filename)
        file.save(filepath)
        
        safe_name = os.path.splitext(os.path.basename(filename))[0]
        output_dir = os.path.join(FRAMES_BASE, user_id, safe_name)
        
        if os.path.exists(output_dir) and os.listdir(output_dir):
            return jsonify({'success': False, 'error': 'GIF with this name already exists'}), 400
        
        os.makedirs(output_dir, exist_ok=True)
        
        try:
            extract_frames(filepath, output_dir, safe_name)
            response = make_response(jsonify({'success': True, 'user_id': user_id, 'gif_name': safe_name}))
            response.set_cookie('user_id', user_id)
            return response
        except Exception as e:
            return jsonify({'success': False, 'error': str(e)}), 500

@app.route('/convert')
def convert():
    user_id = request.args.get('user_id', 'anonymous')
    filename = request.args.get('filename', '')

    input_path = os.path.join(app.config['UPLOAD_FOLDER'], secure_filename(user_id), filename)
    if not os.path.exists(input_path):
        return "File not found", 404

    safe_name = os.path.splitext(os.path.basename(filename))[0]
    output_dir = os.path.join(FRAMES_BASE, user_id, safe_name)
    os.makedirs(output_dir, exist_ok=True)

    try:
        frame_count = extract_frames(input_path, output_dir, safe_name)
        return redirect(url_for('index', view=safe_name, user_id=user_id))
    except Exception as e:
        return f"Error processing file: {str(e)}", 500

@app.route('/get_frames')
def get_frames():
    user_id = request.args.get('user_id', 'anonymous')
    gif_name = request.args.get('gif_name', '')
    
    frames_dir = os.path.join(FRAMES_BASE, user_id, gif_name)
    
    if not os.path.exists(frames_dir):
        return jsonify({'error': 'GIF not found'}), 404
    
    frames = sorted([f for f in os.listdir(frames_dir) if f.startswith('frame_') and f.endswith('.png')])
    
    return jsonify(frames)

application = app
```
The app takes a gif and splits it into frames to be generated as a video in ASCII format. 

The actual frames for each gif are served from the flask static directory:
```python
app = Flask(__name__, template_folder=os.path.join(BASE_DIR, 'templates'),  static_folder='/srv/http/static')

UPLOAD_FOLDER = '/srv/http/uploads'
FRAMES_BASE = '/srv/http/static/frames'
```

From `https://bad-apple.tamuctf.com/browse/admin/` I know the flag is in `https://bad-apple.tamuctf.com/browse/admin/35f3e5e97d0a8f45ad8c55775b8558f9-flag.gif`.

I can access each frame at: 
```
https://bad-apple.tamuctf.com/static/frames/admin/35f3e5e97d0a8f45ad8c55775b8558f9-flag/frame_0001.png
``` 
and so on. 

The admin flag has 155 frames (found at `https://bad-apple.tamuctf.com/get_frames?user_id=admin&gif_name=35f3e5e97d0a8f45ad8c55775b8558f9-flag`). I run this to download each frame:
```shell
for i in $(seq -w 1 155); do  
wget https://bad-apple.tamuctf.com/static/frames/admin/35f3e5e97d0a8f45ad8c55775b8558f9-flag/frame_0$i.png  
done
```

Then I can make it a video with `ffmpeg -framerate 10 -i frame_%04d.png out.mp4`. From the video I read the flag.

`gigem{3z_t0h0u_fl4g_r1t3}`

___

## Broken Website

**Category**: web

**Author**: `cobra`

**Description**: My fancy new website is broken. Can you figure out what is wrong? [https://broken-website.tamuctf.cybr.club](https://broken-website.tamuctf.cybr.club)

The site does not load in any browser. `HTTP/1.1` and `HTTP/2` TLS connections are rejected:
```shell
curl -v https://broken-website.tamuctf.cybr.club  
* Host broken-website.tamuctf.cybr.club:443 was resolved.  
* IPv6: (none)  
* IPv4: 54.91.191.64  
*   Trying 54.91.191.64:443...  
* ALPN: curl offers h2,http/1.1  
* TLSv1.3 (OUT), TLS handshake, Client hello (1):  
*  CAfile: /etc/ssl/certs/ca-certificates.crt  
*  CApath: /etc/ssl/certs  
* Recv failure: Connection reset by peer  
* TLS connect error: error:00000000:lib(0)::reason(0)  
* OpenSSL SSL_connect: Connection reset by peer in connection to broken-website.tamuctf.cybr.club:443  
* closing connection #0  
curl: (35) Recv failure: Connection reset by peer
```

The only attempt that got a real reply from the server was over `QUIC/UDP` `HTTP/3`:
```shell
curl --http3 https://broken-website.tamuctf.cybr.club -v  
* Host broken-website.tamuctf.cybr.club:443 was resolved.  
* IPv6: (none)  
* IPv4: 54.91.191.64  
*   Trying 54.91.191.64:443...  
*  CAfile: /etc/ssl/certs/ca-certificates.crt  
*  CApath: /etc/ssl/certs  
*   Trying 54.91.191.64:443...  
* SSL certificate problem: unable to get local issuer certificate  
* QUIC connect to 54.91.191.64 port 443 failed: SSL peer certificate or SSH remote key was not OK
... etc
```

Skip cert verification with `-k`:
```shell
curl --http3 -k https://broken-website.tamuctf.cybr.club  
<!DOCTYPE html>  
<html lang="en">  
<head>  
	<meta charset="UTF-8">  
	<meta name="viewport" content="width=device-width, initial-scale=1.0">  
	<title>Fancy Website</title>  
	<link rel="stylesheet" type="text/css" href="style.css">  
	<link rel="preconnect" href="https://fonts.googleapis.com">  
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>  
<link href="https://fonts.googleapis.com/css2?family=Ubuntu:ital,wght@0,300;0,400;0,500;0,700;1,300;1,400;1,500;1,700&display=swap" rel="stylesheet">  
</head>  
<body>  
	<h1>Welcome to my website!</h1>  
	<h2>Here's the flag:</h2>  
	<h2>gigem{7h3_fu7u23_15_qu1c_64d1f5}</h2>  
</body>  
</html>
```

`gigem{7h3_fu7u23_15_qu1c_64d1f5}`
