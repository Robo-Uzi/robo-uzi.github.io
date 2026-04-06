---
layout: post
title:  "T-reasure Chest"
date:   2026-04-06 18:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-t-reasure-chest/
---

**Category**: rev

**Author:** @somnomly

**Description**: Score! You found a treasure chest! Now if only you could figure out how to unlock it... maybe there's a magic word?

I get the challenge file:
```shell
file treasure  
treasure: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /nix/store/8p33is69mjdw3bi1wmi8v2zpsxir8nwd-glibc-2.40-66/lib/ld-linux-x86-64.so.2, for GNU/Linux 3.10.0, stripped
```

At first it won't run:
```shell
./treasure  
bash: ./treasure: cannot execute: required file not found
```

I put the binary in ghidra. I find these functions:
```shell
void FUN_004011c6(uint *param_1,int *param_2)

{
  uint local_1c;
  uint local_18;
  int local_10;
  int local_c;
  
  local_18 = *param_1;
  local_1c = param_1[1];
  local_c = 0;
  for (local_10 = 0; local_10 < 0x20; local_10 = local_10 + 1) {
    local_c = local_c + -0x61c88647;
    local_18 = local_18 +
               (local_1c * 0x10 + *param_2 ^ local_c + local_1c ^ param_2[1] + (local_1c >> 5));
    local_1c = local_1c +
               (local_18 * 0x10 + param_2[2] ^ local_c + local_18 ^ param_2[3] + (local_18 >> 5));
  }
  *param_1 = local_18;
  param_1[1] = local_1c;
  return;
}
```

```shell
void FUN_004012a9(long param_1,uint param_2,undefined8 param_3)

{
  undefined8 local_14;
  uint local_c;
  
  for (local_c = 0; local_c < param_2 >> 2; local_c = local_c + 1) {
    local_14 = *(undefined8 *)(param_1 + (int)(local_c << 3));
    FUN_004011c6(&local_14,param_3);
    *(undefined8 *)((int)(local_c << 3) + param_1) = local_14;
  }
  return;
}
```

```shell
undefined8 FUN_0040131b(void)

{
  int iVar1;
  size_t sVar2;
  char local_138 [256];
  undefined8 local_38;
  undefined8 local_30;
  void *local_20;
  int local_14;
  int local_10;
  int local_c;
  
  local_38 = 0x636e655f796e6974;
  local_30 = 0x79656b5f74707972;
  puts("Try to open the chest!");
  printf("Maybe try saying the magic word: ");
  fgets(local_138,0x100,stdin);
  sVar2 = strcspn(local_138,"\n");
  local_138[sVar2] = '\0';
  printf("input: %s\n",local_138);
  sVar2 = strlen(local_138);
  local_10 = (int)sVar2;
  local_14 = local_10 % 8;
  local_20 = malloc((long)(local_14 + local_10));
  memcpy(local_20,local_138,(long)local_10);
  memset((void *)((long)local_10 + (long)local_20),0,(long)local_14);
  local_10 = local_10 + local_14;
  FUN_004012a9(local_20,local_10,&local_38);
  printf("result: 0x");
  for (local_c = 0; local_c < local_10; local_c = local_c + 1) {
    printf("%02X",(ulong)*(byte *)((long)local_20 + (long)local_c));
  }
  putchar(10);
  if ((local_10 == 0x22) && (iVar1 = memcmp(local_20,&DAT_00404080,0x22), iVar1 == 0)) {
    puts("Congrats! Here\'s your treasure: ");
    puts("*******************************************************************************");
    puts("          |                   |                  |                     |");
    puts(" _________|________________.=\"\"_;=.______________|_____________________|_______");
    puts("|                   |  ,-\"_,=\"\"     `\"=.|                  |");
    puts("|___________________|__\"=._o`\"-._        `\"=.______________|___________________");
    puts("          |                `\"=._o`\"=._      _`\"=._                     |");
    puts(" _________|_____________________:=._o \"=._.\"_.-=\"\'\"=.__________________|_______");
    puts("|                   |    __.--\" , ; `\"=._o.\" ,-\"\"\"-._ \".   |");
    puts("|___________________|_._\"  ,. .` ` `` ,  `\"-._\"-._   \". \'__|___________________");
    puts("          |           |o`\"=._` , \"` `; .\". ,  \"-._\"-._; ;              |");
    puts(" _________|___________| ;`-.o`\"=._; .\" ` \'`.\"` . \"-._ /________________|_______");
    puts("|                   | |o;    `\"-.o`\"=._``  \'` \" ,__.--o;   |");
    puts("|___________________|_| ;     (#) `-.o `\"=.`_.--\"_o.-; ;___|___________________");
    puts("____/______/______/___|o;._    \"      `\".o|o_.--\"    ;o;____/______/______/____");
    puts("/______/______/______/_\"=._o--._        ; | ;        ; ;/______/______/______/_");
    puts("____/______/______/______/__\"=._o--._   ;o|o;     _._;o;____/______/______/____");
    puts("/______/______/______/______/____\"=._o._; | ;_.--\"o.--\"_/______/______/______/_");
    puts("____/______/______/______/______/_____\"=.o|o_.--\"\"___/______/______/______/____");
    puts("/______/______/______/______/______/______/______/______/______/______/________");
    puts("*******************************************************************************");
    free(local_20);
    return 0;
  }
  puts("Hmmmm.... didn\'t open...");
  free(local_20);
  return 0;
}
```

It wouldnt run because it was built with NixOS:
```shell
interpreter /nix/store/8p33is69mjdw3bi1wmi8v2zpsxir8nwd-glibc-2.40-66/lib/ld-linux-x86-64.so.2
```

```shell
patchelf --set-interpreter /lib64/ld-linux-x86-64.so.2 treasure

./treasure  
Try to open the chest!  
Maybe try saying the magic word:
```

It doesnt need to run though.

The program is using TEA encryption ([Tiny Encryption Algorithm](https://en.wikipedia.org/wiki/Tiny_Encryption_Algorithm)) with 32 rounds, a 64 bit block, and a 128 bit key. In `FUN_004011c6()`, `-0x61c88647` is the TEA delta constant (the twos complement negative of the TEA delta constant `0x9E3779B9`, which is used every round in the TEA cipher).

The encryption key is right at the top of `FUN_0040131b()`:
```shell
local_38 = 0x636e655f796e6974;
local_30 = 0x79656b5f74707972;
```

[Cyberchef](https://gchq.github.io/CyberChef/#recipe=Swap_endianness('Hex',8,true)From_Hex('Space')&input=NjM2ZTY1NWY3OTZlNjk3NDc5NjU2YjVmNzQ3MDc5NzIK) recipe for encryption key.

Encryption key: `tiny_encrypt_key`

The ciphertext is 34 bytes starting at `DAT_00404080`:
```shell
if ((local_10 == 0x22) && (iVar1 = memcmp(local_20,&DAT_00404080,0x22), iVar1 == 0)) {
    puts("Congrats! Here\'s your treasure: ");
```

The 34 bytes (`0x22`) starting at `0x404080`:
```shell
38755bcb44d2be5d969c5643ea9806754a4813e6d4e88e4f72708bffdc99f876c5c9
```

Solve script:
```python
import struct

DELTA = 0x9E3779B9

def tea_decrypt_block(v, k):
    v0, v1 = v
    sum = (DELTA * 32) & 0xffffffff

    for _ in range(32):
        v1 = (v1 - (((v0 << 4) + k[2]) ^ (sum + v0) ^ ((v0 >> 5) + k[3]))) & 0xffffffff
        v0 = (v0 - (((v1 << 4) + k[0]) ^ (sum + v1) ^ ((v1 >> 5) + k[1]))) & 0xffffffff
        sum = (sum - DELTA) & 0xffffffff

    return v0, v1


key = b"tiny_encrypt_key"
k = struct.unpack("<4I", key)

cipher = bytes.fromhex(
"38755bcb44d2be5d969c5643ea9806754a4813e6d4e88e4f72708bffdc99f876c5c9"
)

out = b""

for i in range(0, len(cipher), 8):
    block = cipher[i:i+8]
    if len(block) < 8:
        block = block.ljust(8, b"\x00")

    v = struct.unpack("<2I", block)
    d = tea_decrypt_block(v, k)
    out += struct.pack("<2I", *d)

print(out.rstrip(b"\x00"))
```

Output:
```shell
python3 solve.py  
b'RS{oh_its_a_TEAreasure_chest}\x00\x00\x00e\x82\xfe\xc9Xl\xb9\xea'
```

`RS{oh_its_a_TEAreasure_chest}`