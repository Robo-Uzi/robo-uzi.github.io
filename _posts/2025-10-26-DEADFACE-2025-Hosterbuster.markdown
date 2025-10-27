---
layout: post
title:  "Hostbuster"
date:   2025-10-26 21:39:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /hostbuster/
---
* TOC
{:toc}

## Let Me In

**Category:** forensics

**Author:** @syyntax

**Description:** Due to the efforts of the Turbo Tactical team, we’ve managed to acquire credentials belonging to `gh0st404` that can be used on a remote DEADFACE machine. Access this machine and find the flag in the user’s home directory. Submit the flag as `deadface{flag_text}`  

IP Address: `hostbusters.deadface.io`  
Username: `gh0st404`  
Password: `ReadySetG0!`

ssh with `ssh gh0st404@hostbusters.deadface.io` and password `ReadySetG0!`:
```shell
~ $ ls -la  
total 28  
drwxr-sr-x    1 gh0st404 gh0st404      4096 Oct 25 14:27 .  
drwxr-xr-x    1 root     root          4096 Sep 17 02:04 ..  
-rw-------    1 gh0st404 gh0st404         7 Oct 25 14:27 .ash_history  
-rw-r--r--    1 gh0st404 gh0st404       382 Sep 17 02:04 .dont_forget  
-rw-r--r--    1 gh0st404 gh0st404      1850 Sep 17 02:04 notes.txt  
drwxr-sr-x    1 gh0st404 gh0st404      4096 Sep 17 02:04 tools  
~ $ cat notes.txt  
Day 1 — 10:56 PM  
Started pokin’ around De Monne’s public face. Clean enough on the outside, but you know how it goes — pretty lobby, rotten basement. Grabbed some domains: demonnefinancial.com, plus a couple cute little subdomains hangin’ off it. They really love their marketing crap.  
  
Day 2 — 01:12 AM  
Email patterns? Easy. first.last@demonnefinancial.com. Classic rookie move. Ran a quick scrape off LinkedIn — already got a list of 50+ employees. Shoutout to the interns who think posting their whole résumé online is a flex.  
  
Day 3 — 02:20 AM  
Spotted their VPN login page. Looks shiny, probably thinks it’s bulletproof. No MFA prompt though. That’s a red flag. If it’s really missing, well… let’s just say some middle manager is definitely rockin’ “Password123!” somewhere in there.  
  
Day 4 — 11:37 PM  
Their external web app? Yikes. Old Apache headers, outdated PHP version just sittin’ there. Might as well hang a “hack me” sign on the door. Gonna run some deeper scans later. Bet I’ll find something dusty.  
  
Day 5 — 03:03 AM  
Poked at their DNS records tonight. Found a staging server hanging out in the wild. Who does that in 2025? Might just be a breadcrumb, but if it’s real… it’s prob got hardcoded creds all over it. Note to self: come back when I’m less tired.  
  
Day 6 — 12:18 AM  
Weird thing: their main site pushes traffic through Cloudflare, but half their subdomains don’t. Split-brain security. Classic. Whoever set this up either cut corners or didn’t know what the hell they were doing. Both work in my favor.  
  
Day 7 — 01:45 AM  
No break-ins yet — just circling the building, peeking in the windows. Feels like De Monne wants me to think they’re solid. But I can see the cracks. Just need the right pressure to make ‘em split.  
  
deadface{hostbusters1_cf6a12ddf781cfbc}~ $
```

`deadface{hostbusters1_cf6a12ddf781cfbc}`

___

## Secret Stash

**Category:** forensics

**Author:** @syyntax

**Description:** Look around for files that `gh0st404` might have hidden - he’s a novice DEADFACE member so we suspect he probably hides files in the simplest way. Submit the flag as `deadface{flag_text}`. Use the connection information from **Hostbusters: Let Me In**

```shell
~ $ cat .dont_forget  
random junk, don’t lose this file (again)  
  
[personal]  
Gh0st!v3r$e_404  
0nly_Sh4d0w_Kn0ws  
n0scopez420!  
  
[tools]  
- burp:         waf_slayer$$  
- metasploit:   c0d3BReaKer!  
- vpn (alt):    4sh3s2dust_  
  
[misc]  
- music:      s1lent_echoes  
- github:       b4ckd00rRabb1t  
- deephax pen: Fr4gm3ntedSkull!!  
  
# note to self: change the pen one (again)  
  
deadface{hostbusters2_4685d0c801939781}~ $
```

I also found I can become `deephax` with `su deephax` and password `Fr4gm3ntedSkull!!`:
```shell
~ $ su deephax  
Password:    
YO welcome to deephax’s red team console  type 'help' for commands or whatever. let’s HAX!   
(deephax)> help  
  
Documented commands (type help <topic>):  
========================================  
base64  decrypt  encrypt  forensics  hash  help  quit  recon  
  
(deephax)>
```

`deadface{hostbusters2_4685d0c801939781}`

___

## Et Cetera

**Category:** forensics

**Author:** @syyntax

**Description:** `gh0st404` is complaining in Ghost Town about his script not working. See if you can locate the script on the remote machine and read the flag. Submit the flag as `deadface{flag_text}`. Use the connection information from **Hostbusters: Let Me In**

On Ghost Town (`https://ghosttown.deadface.io/t/bruh-why-is-my-script-still-broken/93`) I find them talking about how the script won't work because it was moved to `/etc`. 

To view the file I just change the permissions with `chmod`:
```shell
~ $ chmod 644 /etc/logclean.sh  
~ $ cat /etc/logclean.sh  
#!/bin/bash  
# logclean.sh  
# gh0st404’s “oops, wasn’t me” button  
  
echo "[*] wiping bash history..."  
cat /dev/null > ~/.bash_history 2>/dev/null  
history -c 2>/dev/null  
  
echo "[*] nuking known_hosts..."  
rm -f ~/.ssh/known_hosts 2>/dev/null  
  
echo "[*] clearing tmp files..."  
rm -rf /tmp/* 2>/dev/null  
rm -rf /var/tmp/* 2>/dev/null  
  
echo "[*] zeroing logs (local only)..."  
for log in /var/log/*.log; do  
   [ -f "$log" ] && cat /dev/null > "$log"  
done  
  
echo "[*] cleaning done. ghost mode re-enabled."  
  
echo "deadface{hostbusters3_0547796725934bbd}"~ $
```

`deadface{hostbusters3_0547796725934bbd}`

___

## Lay of the Land

**Category:** forensics

**Author:** @syyntax

**Description:** Scope out and continue characterizing the host. What are some of the characters of this environment? Find the flag associated with `hostbusters4`. Submit the flag as `deadface{flag_text}`. 

Looking at `/proc/1/environ` shows the flag:
```shell
/ $ cat /proc/1/environ  
USER=gh0st404SHLVL=1HOME=/home/gh0st404PAGER=lessflag=deadface{hostbusters4_c6e54afa62741d34}LOGNAME=gh0st404TERM=xtermLC_COLLATE=CPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binLANG=C.UTF-8SHELL=/bin/shPWD=/home/gh0st404CHARSET=UTF-8/ $
```
A much simpler way of finding this is running the command `env` or `set`...

`deadface{hostbusters4_c6e54afa62741d34}`

___

## Worldwide

**Category:** forensics

**Author:** @syyntax

**Description:** The system seems to have multiple users, one of which is `deephax`. There’s a GhostTown thread between `gh0st404` and `deephax` talking about a `pen-console.py` being `deephax`’s default shell. He also mentions giving `gh0st404` his password. Look around the machine and see if you escalate laterally to `deephax`’s account. There may be information we can use (i.e., flag) in `deephax`’s shell environment. Submit the flag as `deadface{flag_text}`. Use the connection information from **Hostbusters: Let Me In**. 

```shell
/home $ find / -type f -iname "pen-console.py" 2>/dev/null || true  
/usr/local/bin/pen-console.py  
/home $    
/home $ cat /usr/local/bin/pen-console.py  
#!/usr/bin/python3  
  
import cmd  
import os  
import hashlib  
import base64  
import subprocess  
from cryptography.fernet import Fernet  # you need this: pip install cryptography  
  
flag="deadface{hostbusters5_e16a5c8995620a24}"  
  
class DeephaxConsole(cmd.Cmd):  
   intro = "YO welcome to deephax’s red team console  type 'help' for commands or whatever. let’s HAX! "  
   prompt = '(deephax)> '  
      
   def do_hash(self, arg):  
       """hash <string>: gimme a string, i spit hashes back at ya."""  
       if not arg:  
           print("bruh... gimme sumthin to hash!! ")  
           return  
       print(f"MD5: {hashlib.md5(arg.encode()).hexdigest()}")  
       print(f"SHA1: {hashlib.sha1(arg.encode()).hexdigest()}")  
       print(f"SHA256: {hashlib.sha256(arg.encode()).hexdigest()}")  
      
   def do_encrypt(self, arg):  
       """encrypt <message>: encrypts ur message wit Fernet """  
       if not arg:  
           print("what u tryna lock up? type somethin! ")  
           return  
       key = Fernet.generate_key()  
       cipher = Fernet(key)  
       encrypted = cipher.encrypt(arg.encode())  
       print(f"here's ur key (DON'T LOSE IT): {key.decode()}")  
       print(f"and here’s ur encrypted junk: {encrypted.decode()}")  
      
   def do_decrypt(self, arg):  
       """decrypt <key> <ciphertext>: unlock secrets 🕵‍♂"""  
       args = arg.split()  
       if len(args) != 2:  
           print("use it like: decrypt <key> <ciphertext>")  
           return  
       try:  
           cipher = Fernet(args[0].encode())  
           decrypted = cipher.decrypt(args[1].encode()).decode()  
           print(f"BOOM decrypted: {decrypted}")  
       except Exception as e:  
           print(f"that ain’t workin fam  err: {e}")  
      
   def do_base64(self, arg):  
       """base64 <encode|decode> <data>: 4 all ur b64 nonsense"""  
       args = arg.split(maxsplit=1)  
       if len(args) != 2:  
           print("bruh use it like: base64 <encode|decode> <data>")  
           return  
       mode, data = args[0], args[1]  
       if mode == 'encode':  
           print(base64.b64encode(data.encode()).decode())  
       elif mode == 'decode':  
           try:  
               print(base64.b64decode(data.encode()).decode())  
           except Exception as e:  
               print(f"b64 decode borked: {e}")  
       else:  
           print("nah man, u gotta say 'encode' or 'decode'. not both. not neither.")  
      
   def do_recon(self, arg):  
       """recon <ip>: run nmap on an IP """  
       if not arg:  
           print("yo i need an IP ")  
           return  
       try:  
           result = subprocess.run(['nmap', '-sV', arg], capture_output=True, text=True)  
           print(result.stdout)  
       except FileNotFoundError:  
           print("nmap ain't even installed??? WACK.")  
       except Exception as e:  
           print(f"recon died  err: {e}")  
      
   def do_forensics(self, arg):  
       """forensics <file>: do sum CSI junk on a file """  
       if not arg:  
           print("what file u want me 2 peek at? ")  
           return  
       if not os.path.exists(arg):  
           print("dude... that file don’t even EXIST ")  
           return  
       size = os.path.getsize(arg)  
       file_type = subprocess.run(['file', arg], capture_output=True, text=True).stdout.strip()  
       with open(arg, 'rb') as f:  
           content = f.read()  
           sha256 = hashlib.sha256(content).hexdigest()  
       print(f"Size: {size} bytes")  
       print(f"Type: {file_type}")  
       print(f"SHA256: {sha256}")  
      
   def do_quit(self, arg):  
       """yeet outta this console."""  
       print("PEACE OUT - deephax out ✌")  
       return True  
  
if __name__ == '__main__':  
   DeephaxConsole().cmdloop()
```

`deadface{hostbusters5_e16a5c8995620a24}`

___

## Pulse Check

**Category:** forensics linux

**Author:** @syyntax

**Description:** What is this machine doing? Our team has looked in various files and we can’t seem to locate the 7th flag. Maybe we need to characterize this host more. Submit the flag as `deadface{flag_text}`.

There is a suspicious binary running as a service `/usr/bin/dfcheckalive`.

To download the binary I run `base64 /usr/bin/dfcheckalive` and save the base64 to a file on my machine. Then run `base64 -d dfcheckalive.b64 > dfcheckalive` and I get the file:
```shell
file dfcheckalive  
dfcheckalive: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-x86_64.so.1, BuildID[sha1]=a911c0d6d1c575da94906de579ade3eca6dbcb54, with debug_info, not stripped
```

I open the binary in ghidra. I see this function:
```d
void decrypt_flag(long param_1,long param_2,int param_3,long param_4,int param_5)

{
  int local_c;
  
  for (local_c = 0; local_c < param_3; local_c = local_c + 1) {
    *(byte *)(param_1 + local_c) =
         *(byte *)(param_2 + local_c) ^ *(byte *)(param_4 + local_c % param_5);
  }
  *(undefined *)(param_1 + param_3) = 0;
  return;
}
```

There is a XOR key and cipher text in `.rodata`:
```shell
objdump -s -j .rodata ./dfcheckalive  
  
./dfcheckalive:     file format elf64-x86-64  
  
Contents of section .rodata:  
2000 68a1389b f2be7eb9 08000000 00000000  h.8...~.........  
2010 00000000 00000000 00000000 00000000  ................  
2020 0cc459ff 94df1ddc 13c957e8 86dc0bca  ..Y.......W.....  
2030 1cc44ae8 c5e11a81 0b9159af 918a1b8f  ..J.......Y.....  
2040 5cc00df9 c5860300 756e6b6e 6f776e00  \.......unknown.  
2050 302e302e 302e3000 67657469 66616464  0.0.0.0.getifadd  
2060 72730064 6f636b65 72006574 6830002f  rs.docker.eth0./  
2070 00737461 74766673 0072002f 70726f63  .statvfs.r./proc  
2080 2f6d656d 696e666f 00666f70 656e202f  /meminfo.fopen /  
2090 70726f63 2f6d656d 696e666f 004d656d  proc/meminfo.Mem  
20a0 546f7461 6c3a2025 6c6c6420 6b420000  Total: %lld kB..  
20b0 7b227374 61747573 223a2261 6c697665  {"status":"alive  
20c0 222c2266 6c616722 3a222573 222c2268  ","flag":"%s","h  
20d0 6f737422 3a7b2268 6f73746e 616d6522  ost":{"hostname"  
20e0 3a222573 222c2269 70223a22 2573222c  :"%s","ip":"%s",  
20f0 22646973 6b5f6762 223a256c 6c642c22  "disk_gb":%lld,"  
2100 6d656d5f 6762223a 256c6c64 7d7d0045  mem_gb":%lld}}.E  
2110 52524f52 20637265 6174696e 6720736f  RROR creating so  
2120 636b6574 00313237 2e302e30 2e310000  cket.127.0.0.1..  
2130 4552524f 5220696e 76616c69 64207365  ERROR invalid se  
2140 72766572 20495020 61646472 65737300  rver IP address.  
2150 53746172 74696e67 20444541 44464143  Starting DEADFAC  
2160 45202741 6c697665 27205265 706f7274  E 'Alive' Report  
2170 65722028 64666368 65636b61 6c697665  er (dfcheckalive  
2180 292e2e2e 00000000 53656e64 696e6720  ).......Sending    
2190 686f7374 20737461 74757320 4a534f4e  host status JSON  
21a0 20746f20 25733a25 64206576 65727920   to %s:%d every    
21b0 25642073 65636f6e 64732e0a 00455252  %d seconds...ERR  
21c0 4f522073 656e6469 6e67206d 65737361  OR sending messa  
21d0 676500                               ge.
```

Run this python to get the flag:
```python
#!/usr/bin/env python3

import sys

BIN = sys.argv[1] if len(sys.argv) > 1 else "dfcheckalive"

RODATA_START = 0x2000
KEY_OFFSET = RODATA_START
KEY_LEN = 8
ENC_OFFSET = RODATA_START + 0x20
ENC_LEN = 0x27

def xor_decrypt(enc: bytes, key: bytes) -> bytes:
    return bytes(enc[i] ^ key[i % len(key)] for i in range(len(enc)))

def main():
    with open(BIN, "rb") as f:
        data = f.read()

    if len(data) < ENC_OFFSET + ENC_LEN:
        print("Verify path and .rodata offsets :)")
        sys.exit(1)

    key = data[KEY_OFFSET:KEY_OFFSET + KEY_LEN]
    enc = data[ENC_OFFSET:ENC_OFFSET + ENC_LEN]
    out = xor_decrypt(enc, key)

    try:
        print(out.decode("utf-8"))
    except UnicodeDecodeError:
        print("Decoded (bytes):", out)
        print("Decoded (hex):", out.hex())

if __name__ == "__main__":
    main()
```

`deadface{hostbusters7_d8c0a4c4e64a5b78}`

___
