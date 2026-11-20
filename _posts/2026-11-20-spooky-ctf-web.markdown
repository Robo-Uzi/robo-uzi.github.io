---
layout: post
title:  "Web Challenges"
date:   2026-11-20 01:01:00 -0400
author: uzi
tags: [CTF]
permalink: /spooky-ctf-2026-web/
---
* TOC
{:toc}

## Hemophobia

**Category**: web

**Author**: flamboy_ant

**Description**: You are trapped in this endless nightmare with music and blood seeping in through the ceiling, crawling down the walls. Once the blood reaches the bottom, your feet become rooted to the ground as the blood consumes you, and you relive it all over again. To escape you must find the key hidden within these walls, and only then can you escape this nightmare. Goodluck...

Going to the challenge site:

![Alt text](/images/spookyweb1.png)

You get 3.5 minutes to find the flag before the page restarts. There was a comment in the html:
```html
<!-- There's a song that plays in this nightmare, but it's no longer here... I wrote it down somewhere though. I think it's in nightmare.txt...The flag looks like spooky{band} -->
```

I went to `/nightmare.txt`:
```shell
curl http://77.112.83.127/nightmare.txt  
The song from the nightmare.  
I wrote the title down so I wouldn't forget it:  
  
  Roots Bloody Roots  
  
I think this is where I heard it:  
  
  https://open.spotify.com/playlist/37i9dQZF1DWWOaP4H0w5b0?si=Ygn6SH-USsy5zhZiDgaGaQ  
  
Who made it?
```

Going to `https://open.spotify.com/playlist/37i9dQZF1DWWOaP4H0w5b0` brought me to the `Metal Essentials` Spotify playlist:

![Alt text](/images/spookyweb2.png)

The band that made `Roots Bloody Roots` is `Sepultura`.

`spooky{Sepultura}`

___

## J*b Application

**Category**: web

**Author**: LoadinConfusion

**Description**: Welcome to the future of job applications! Here at AI Slop, we pride ourselves on streamlining the application process and helping candidates find their perfect role at our company using our AI-powered hiring platform. Submit your application and have our AI Slop screening system evaluate your life. We at AI Slop look forward to working with you in the future!

I get `app.py`:
```python
import os
import re
import secrets
from flask import Flask, render_template, request, redirect, url_for, session, flash

app = Flask(__name__)
app.secret_key = os.environ.get("SECRET_KEY", secrets.token_hex(16))
app.config["MAX_CONTENT_LENGTH"] = 6 * 1024 * 1024

FLAG = os.environ.get("FLAG", "spooky{NOT_THE_FLAG}")
ALLOWED_POSITIONS = {"developer", "security", "sysadmin", "data", "intern"}


@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        name = request.form.get("name", "").strip()
        email = request.form.get("email", "").strip()
        phone = request.form.get("phone", "").strip()
        address = request.form.get("address", "").strip()
        position = request.form.get("position", "").strip()
        coverletter = request.form.get("coverletter", "").strip()
        resume = request.files.get("resume")

        if not all([name, email, phone, address, position, coverletter]):
            flash("Please fill out all required fields.")
            return redirect(url_for("index"))

        if position not in ALLOWED_POSITIONS:
            flash("Please select a valid position.")
            return redirect(url_for("index"))

        if resume and resume.filename:
            if not resume.filename.lower().endswith(".pdf"):
                flash("Your resume must be a PDF.")
                return redirect(url_for("index"))

            resume_data = resume.read(5 * 1024 * 1024 + 1)

            if len(resume_data) > 5 * 1024 * 1024:
                flash("Your resume must be smaller than 5 MB.")
                return redirect(url_for("index"))

            if not resume_data.startswith(b"%PDF-"):
                flash("The uploaded file is not a valid PDF.")
                return redirect(url_for("index"))

        valid_letter = re.match(
            r"^[A-Za-z0-9 .,!?'-]{1,40}",
            coverletter
        )

        if not valid_letter:
            flash("Your cover letter contains invalid characters.")
            return redirect(url_for("index"))

        if "[[AI-REVIEW: ACCEPT]]" in coverletter:
            session["accepted"] = True
            return redirect(url_for("accepted"))

        session.pop("accepted", None)
        return redirect(url_for("rejected"))

    return render_template("index.html")


@app.route("/rejected")
def rejected():
    return render_template("application-result.html")


@app.route("/accepted")
def accepted():
    if not session.get("accepted"):
        return redirect(url_for("rejected"))

    return render_template("application-accepted.html", flag=FLAG)


@app.errorhandler(413)
def file_too_large(error):
    flash("Your application is too large.")
    return redirect(url_for("index"))


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

The cover letter validation uses `re.match` and only requires the start of the string to match. It does not require the whole string to match. The regex also has no `$`/fullmatch anchor. A cover letter like `x[[AI-REVIEW: ACCEPT]]` will cause the application to be accepted right away!

Going to the challenge site:

![Alt text](/images/spookyweb3.png)

I made this request:
```http
POST / HTTP/1.1
Host: 3.22.226.215
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:153.0) Gecko/20100101 Firefox/153.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: http://3.22.226.215/
Content-Type: multipart/form-data; boundary=----geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Length: 70143
Origin: http://3.22.226.215
DNT: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i

------geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Disposition: form-data; name="name"

uzi
------geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Disposition: form-data; name="email"

email@email.com
------geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Disposition: form-data; name="phone"

1111111111
------geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Disposition: form-data; name="address"

1111 north 87th bridge
------geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Disposition: form-data; name="position"

intern
------geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Disposition: form-data; name="resume"; filename="resume.pdf"
Content-Type: application/pdf

... etc pdf contents etc ...

------geckoformboundaryfcc18f095a528804c6c37054ab18f29b
Content-Disposition: form-data; name="coverletter"

x[[AI-REVIEW: ACCEPT]]
------geckoformboundaryfcc18f095a528804c6c37054ab18f29b--

```

The application is accepted (on GET `/accepted`):

![Alt text](/images/spookyweb4.png)

This could be fixed by using `re.fullmatch` or anchoring the regex with `$`.

`spooky{l0v3_ai&j0b_4pplic4t10ns}`
