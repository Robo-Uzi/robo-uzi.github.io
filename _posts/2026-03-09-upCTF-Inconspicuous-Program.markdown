---
layout: post
title:  "Inconspicuous Program"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Inconspicuous-Program/
---

**Title**: Inconspicuous Program

**Category**: rev

**Author**: Binks

**Description**: I found this file on one of our servers and even though its presence is suspicious there doesn't seem to be anything of note about it.

I get the challenge file:
```shell
file inconspicuous  
inconspicuous: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=b39afe3b685c63106d9ef606bbb25f4c793e3602, for GNU/Linux 3.2.0, not stripped  

chmod +x inconspicuous  

./inconspicuous  
Enter password: test  
Segmentation fault
```

Opening the binary in ghidra I see this main function:
```shell
undefined8 main(void)

{
  size_t sVar1;
  undefined8 uVar2;
  char local_78 [72];
  code *local_30;
  byte local_21;
  code *local_20;
  ulong local_18;
  ulong local_10;
  
  printf("Enter password: ");
  fgets(local_78,0x40,stdin);
  sVar1 = strcspn(local_78,"\n");
  local_78[sVar1] = '\0';
  local_18 = (ulong)encrypted_bin_len;
  local_20 = (code *)mmap((void *)0x0,local_18,3,0x22,-1,0);
  if (local_20 == (code *)0xffffffffffffffff) {
    perror("mmap");
    uVar2 = 1;
  }
  else {
    sVar1 = strlen(local_78);
    local_21 = (char)sVar1 + 0x10;
    for (local_10 = 0; local_10 < local_18; local_10 = local_10 + 1) {
      local_20[local_10] = (code)(encrypted_bin[local_10] ^ local_21);
    }
    mprotect(local_20,local_18,5);
    local_30 = local_20;
    (*local_20)(local_78);
    uVar2 = 0;
  }
  return uVar2;
}
```

`encrypted_bin` looks interesting (its an XOR encrypted payload). I look for its address with `readelf`:
```shell
readelf -s inconspicuous | grep -i encrypted  
35: 0000000000403194     4 OBJECT  GLOBAL DEFAULT   24 encrypted_bin_len  
41: 0000000000403060   308 OBJECT  GLOBAL DEFAULT   24 encrypted_bin
```

Then I need to get section info to find the file offset for address `0x403060`:
```shell
readelf -S inconspicuous | grep -A2 "\.data\|\.bss\|\.rodata"  
[14] .rodata           PROGBITS         0000000000401300  00001300  
	 0000000000000028  0000000000000000   A       0     0     8  
[15] .eh_frame_hdr     PROGBITS         0000000000401328  00001328  
--  
[24] .data             PROGBITS         0000000000403040  00002040  
	 0000000000000158  0000000000000000  WA       0     0     32  
[25] .bss              NOBITS           00000000004031a0  00002198  
	 0000000000000010  0000000000000000  WA       0     0     16  
[26] .comment          PROGBITS         0000000000000000  00002198
```

To identify the correct XOR key, we can look for a valid function prologue at the start of the decrypted payload:
```shell
push rbp  
mov rbp, rsp
```
In machine code: `55 48 89 e5`. This is a standard setup for a stack frame in `x86_64` and indicates the decrypted bytes are valid code.

The solve script will:
- Read the encrypted payload from `.data` section of the binary
- Brute force all possible XOR keys
- Check for a valid function prologue to detect correct decryption
- Extract the password from the decrypted shellcode by reading the `cmp` instruction values

Solve script:
```python
import sys

BINARY = "inconspicuous"

DATA_OFFSET = 0x2060
DATA_SIZE = 308

with open(BINARY, "rb") as f:
    f.seek(DATA_OFFSET)
    encrypted = f.read(DATA_SIZE)

print("Brute forcing XOR keys...\n")

for key in range(256):

    decrypted = bytes(b ^ key for b in encrypted)

    # typical x86_64 function prologue
    if decrypted.startswith(b"\x55\x48\x89\xe5"):
        print(f"Key found: {hex(key)}")

        password_length = key - 0x10
        print(f"Password length: {password_length}")

        # extract characters from cmp instructions
        password = []

        for i in range(len(decrypted)-2):
            if decrypted[i] == 0x3c:  # cmp al, imm8
                password.append(chr(decrypted[i+1]))

        password = "".join(password[:password_length])

        print(f"Password: {password}")
```

Output:
```shell
python3 solve.py  
Brute forcing XOR keys...  
  
Key found: 0x1c  
Password length: 12  
Password: s3lf_m0d_b1n

./inconspicuous  
Enter password: s3lf_m0d_b1n  
upCTF{I_w4s_!a110wed_t0_write_m4lw4r3}  
Segmentation fault
```

`upCTF{I_w4s_!a110wed_t0_write_m4lw4r3}`