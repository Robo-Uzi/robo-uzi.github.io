---
layout: post
title:  "baby_baby_reverse"
date:   2025-10-18 15:43:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-baby-baby-reverse/
---

**Title:** baby_baby_reverse

**Category:** reverse

**Description:** This is my first binary project but I don't know if it is safe can you help me check it?

**Author:** LemonTea

I get a binary called `main`:
```shell
file main  
main: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=219b28e83309d120b6ff26a8bc26baabe9207df7, for GNU/Linux 4.4.0, not stripped
```

I open it in ghidra and see this as the main function:
```d
undefined8 main(void)

{
  char *pcVar1;
  undefined8 uVar2;
  long in_FS_OFFSET;
  byte local_24a;
  size_t local_248;
  ulong local_240;
  ulong local_238;
  ulong local_230;
  undefined8 key_part_1;
  undefined8 key_part_2;
  byte local_218 [520];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  key_part_1 = 0x5f73315f73316854;
  key_part_2 = 0x79336b5f336874;
  local_238 = 0xf;       // key length = 15
  local_230 = 0x29;      // input length = 41
  printf("Enter flag: ");
  pcVar1 = fgets((char *)local_218,0x200,stdin);
  if (pcVar1 == (char *)0x0) {
    puts("Input error");
    uVar2 = 1;
  }
  else {
    local_248 = strlen((char *)local_218);
    if ((local_248 != 0) && (local_218[local_248 - 1] == 10)) {
      local_218[local_248 - 1] = 0;
      local_248 = local_248 - 1;
    }
    if (local_248 == local_230) {
      local_24a = 0;
      for (local_240 = 0; local_240 < local_230; local_240 = local_240 + 1) {
        local_24a = local_24a |
                    encrypted[local_240] ^
                    *(byte *)((long)&key_part_1 + local_240 % local_238) ^ local_218[local_240];
      }
      if (local_24a == 0) {
        puts("Correct!");
      }
      else {
        puts("Wrong!");
      }
      uVar2 = 0;
    }
    else {
      puts("Wrong!");
      uVar2 = 0;
    }
  }
  if (local_10 == *(long *)(in_FS_OFFSET + 0x28)) {
    return uVar2;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```
The binary checks that the input equals `encrypted[i] ^ key[i % 15]` for 41 bytes (0x29). 

To get the flag I need the 15 byte key (the program stores it in two 8 byte constants (it is `Th1s_1s_th3_k3y`)), and the 41 bytes of the `encrypted` global array.

I find this with `readelf`:
```shell
readelf -S main | grep -E '\.data'
[24] .data             PROGBITS         0000000000004040  00003040

readelf -s main | grep ' encrypted'
9: 0000000000004060    41 OBJECT  GLOBAL DEFAULT   24 encrypted
```

Using the readelf output I can compute the file offset of `encrypted`:
```
file_offset = section_file_offset + (sym_value - section_addr)
file_offset = 0x3040 + (0x4060 - 0x4040)
file_offset = 0x3040 + 0x20
file_offset = 0x3060
```

After getting the offset, I run this python script and get the flag:
```python
from pathlib import Path

data = Path("main").read_bytes()
enc = data[0x3060:0x3060+41]
key = b"Th1s_1s_th3_k3y"
flag = bytes(e ^ key[i % len(key)] for i, e in enumerate(enc))
print(flag.decode())
```

`QnQSec{This_1s_4n_3asy_r3v3rs3_ch4ll3ng3}`