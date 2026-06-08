---
layout: post
title:  "Rev Challenges"
date:   2026-06-08 14:36:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /GPN-CTF-2026-rev/
---

**Title**: Auto Cooker

**Category**: Reversing

**Author**: MisterPine

**Description**: I always feel like cooking is such a chore... You have to chop up all your ingredients, cook them for hours and then make the plating look half-decent.

But not with this new machine I got! You just have to put in your recipe (weirdly, the interface calls it a flag...) and it will get cooked for you. It's so easy, even someone with no experience in ~~cooking~~ reverse engineering can do it.

I get the challenge file:
```shell
file autocooker  
autocooker: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, not stripped
```

Putting the file into ghidra, I find the main function:
```shell
undefined8 main(int param_1,long param_2)

{
  int iVar1;
  undefined4 local_c;
  
  local_c = 0;
  if (1 < param_1) {
    iVar1 = strcmp(*(char **)(param_2 + 8),"cooking_class");
    if (iVar1 == 0) {
      local_c = 1;
    }
  }
  printf("%s",HEADER);
  printf("%s\n\n",WELCOME);
  puts("Enter your recipe (flag) you want to cook and confirm with [ENTER]:");
  fgets((char *)&RECIPE,0x40,stdin);
  check_recipe_length();
  FOOD = RECIPE;
  DAT_00404128 = DAT_004040e8;
  DAT_00404130 = DAT_004040f0;
  DAT_00404138 = DAT_004040f8;
  DAT_00404140 = DAT_00404100;
  DAT_00404148 = DAT_00404108;
  DAT_00404150 = DAT_00404110;
  DAT_00404158 = DAT_00404118;
  explain_current_food(local_c);
  salt();
  explain_current_food(local_c);
  fry();
  explain_current_food(local_c);
  trim();
  explain_current_food(local_c);
  mix();
  explain_current_food(local_c);
  taste();
  puts("Congratulations, you \"cooked\" a delicious plate of food!");
  return 0;
}
```

`salt`:
```shell
void salt(void)

{
  uint local_c;
  
  puts("Salting...");
  for (local_c = 0; local_c < 0x40; local_c = local_c + 1) {
    *(byte *)((long)&FOOD + (long)(int)local_c) =
         *(byte *)((long)&FOOD + (long)(int)local_c) ^ GRAIN_OF_SALT;
  }
  return;
}
```

`fry`:
```shell
void fry(void)

{
  uint local_c;
  
  puts("Frying...");
  for (local_c = 0; local_c < 0x40; local_c = local_c + 1) {
    *(byte *)((long)&FOOD + (long)(int)local_c) =
         *(char *)((long)&FOOD + (long)(int)local_c) << 4 |
         *(byte *)((long)&FOOD + (long)(int)local_c) >> 4;
  }
  return;
}
```

`trim`:
```shell
void trim(void)

{
  uint local_c;
  
  puts("Oops, it burned :(\nCutting off the burnt bits...");
  for (local_c = TARGET_LENGTH; local_c < 0x40; local_c = local_c + 1) {
    *(byte *)((long)&FOOD + (long)(int)local_c) = *(byte *)((long)&FOOD + (long)(int)local_c) & 0xf;
  }
  return;
}
```

`mix`:
```shell
void mix(void)

{
  undefined local_58 [56];
  undefined8 local_20;
  uint local_c;
  
  puts("Mixing...");
  local_20 = DAT_00404158;
  for (local_c = 0; local_c < 0x40; local_c = local_c + 1) {
    *(undefined *)((long)&FOOD + (long)(int)local_c) =
         *(undefined *)((long)&local_20 + (7 - (long)(int)local_c));
  }
  return;
}
```

`taste`:
```shell
void taste(void)

{
  bool bVar1;
  uint local_10;
  
  puts("Taste testing...");
  bVar1 = false;
  for (local_10 = 0; local_10 < 0x40; local_10 = local_10 + 1) {
    bVar1 = (bool)(bVar1 | *(char *)((long)&FOOD + (long)(int)local_10) != DELICIOUS[(int)local_10])
    ;
  }
  if (bVar1) {
    puts("YUCK!\nOur taste tester thinks your recipe produces bad food, we cannot serve this...");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  return;
}
```

The program transforms the input flag through four reversible operations:
- salt: XOR with `GRAIN_OF_SALT` (0xAA)
- fry: swap nibbles `(x << 4) | (x >> 4)`
- trim: zero out the high nibble for indices greater than `TARGET_LENGTH`
- mix: XOR with a 64 byte constant extracted from `.data`

`taste()` then compares the result against a known `DELICIOUS` array.

Using `objdump -t autocooker` I get the addresses I need. Then I run this python:
```shell
#!/usr/bin/env python3
from pwn import *

elf = ELF('./autocooker')
target_len = u32(elf.read(0x404060, 4))
grain = u8(elf.read(0x404064, 1))
delicious = elf.read(0x404080, 64)
mix_const = elf.read(0x4040e8, 64)

after_mix = delicious
before_mix = xor(after_mix, mix_const)

after_fry = bytearray(before_mix)
for i in range(target_len, 64):
    after_fry[i] &= 0x0f

after_salt = bytes(((b << 4) | (b >> 4)) & 0xff for b in after_fry)
original = xor(after_salt, grain)

print(original[:target_len].decode())
```

output:
```shell
python3 solve.py  
[*] '/home/user/Desktop/ctf/GPNCTF2026/autocooker/autocooker'  
	Arch:       amd64-64-little  
	RELRO:      Partial RELRO  
	Stack:      No canary found  
	NX:         NX enabled  
	PIE:        No PIE (0x400000)  
	Stripped:   No  
  
  
  
  
}w0n_53hs1D_Ts3dr4h_Ruo_r0f_yD43R_Er4_UOy_ekil_13Ef_1{FTC
```

Reverse the string and add `GPN` to get the flag: [cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=Reverse('Character')&input=fXcwbl81M2hzMURfVHMzZHI0aF9SdW9fcjBmX3lENDNSX0VyNF9VT3lfZWtpbF8xM0VmXzF7RlRD)

`GPNCTF{1_fE31_like_yOU_4rE_R34Dy_f0r_ouR_h4rd3sT_D1sh35_n0w}`