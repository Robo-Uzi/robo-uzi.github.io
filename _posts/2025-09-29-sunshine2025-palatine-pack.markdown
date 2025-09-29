---
layout: post
title:  "Palatine Pack"
date:   2025-09-29 13:16:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-palatine-pack/
---

**Title:** Palatine Pack

**Category:** rev

**Author:** Uvuv

**Description:** Caesar is raiding the Roman treasury to pay off his debts to his Gallic allies, and to you and his army. Help him find the password to make this Lucius Caecilius Metellus guy give up the money! >:) (he is sacrosanct so no violence!)

> currently has nonstandard flag format sunshine{}

I get two files. `flag.txt` and the binary `palantinepack`:
```shell
cat flag.txt  
���6▒�>��:=����▒1�0���=������<▒�2���09����7�4���3������2�6���65����=�8��K9������8�:��m<1����3�<��������>�>��12=����▒9�0���5������4�2���8▒9����?���;������:�6���>▒5����5�8��[1�:��}41����;�<��������6▒�>��Q:=����▒1�0���=������<▒�2�

file palatinepack  
palatinepack: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f889e30b5b0d5265c47d47f04584385c9e25fee6, for GNU/Linux 3.2.0, not stripped

./palatinepack  
  
May Jupiter strike you down Caeser before you seize the treasury!! You will have to tear me apart for me to tell you the flag to unlock the Roman Treasury and fund your civil war. I, Lucius Caecilius Metellus, shall not let you pass until you get this password right. (or threaten to kill me-)  
  
Segmentation fault
```

I open the binary in ghidra. This is the main function:
```d
undefined8 main(void)

{
  byte bVar1;
  undefined *puVar2;
  void *__ptr;
  int iVar3;
  long lVar4;
  ulong uVar5;
  undefined8 uVar6;
  FILE *pFVar7;
  undefined *puVar8;
  long in_FS_OFFSET;
  undefined auStack_88 [8];
  int local_80;
  int local_7c;
  FILE *local_78;
  long local_70;
  undefined *local_68;
  undefined8 local_60;
  undefined8 local_58;
  void *local_50;
  FILE *local_48;
  long local_40;
  
  local_40 = *(long *)(in_FS_OFFSET + 0x28);
  puts(
      "\nMay Jupiter strike you down Caeser before you seize the treasury!! You will have to tear me  apart"
      );
  puts(
      "for me to tell you the flag to unlock the Roman Treasury and fund your civil war. I, Lucius C aecilius"
      );
  puts(
      "Metellus, shall not let you pass until you get this password right. (or threaten to kill me-) \n"
      );
  local_78 = fopen("palatinepackflag.txt","r");
  fseek(local_78,0,2);
  lVar4 = ftell(local_78);
  local_7c = (int)lVar4 + 1;
  fseek(local_78,0,0);
  local_70 = (long)local_7c + -1;
  uVar5 = (((long)local_7c + 0xfU) / 0x10) * 0x10;
  for (puVar8 = auStack_88; puVar8 != auStack_88 + -(uVar5 & 0xfffffffffffff000);
      puVar8 = puVar8 + -0x1000) {
    *(undefined8 *)(puVar8 + -8) = *(undefined8 *)(puVar8 + -8);
  }
  lVar4 = -(ulong)((uint)uVar5 & 0xfff);
  local_68 = puVar8 + lVar4;
  if ((uVar5 & 0xfff) != 0) {
    *(undefined8 *)(puVar8 + ((ulong)((uint)uVar5 & 0xfff) - 8) + lVar4) =
         *(undefined8 *)(puVar8 + ((ulong)((uint)uVar5 & 0xfff) - 8) + lVar4);
  }
  pFVar7 = local_78;
  iVar3 = local_7c;
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101bfa;
  fgets(puVar8 + lVar4,iVar3,pFVar7);
  puVar2 = local_68;
  iVar3 = local_7c;
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101c0b;
  flipBits(puVar2,iVar3);
  puVar2 = local_68;
  iVar3 = local_7c;
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101c1c;
  uVar6 = expand(puVar2,iVar3);
  iVar3 = local_7c * 2;
  local_60 = uVar6;
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101c34;
  uVar6 = expand(uVar6,iVar3);
  iVar3 = local_7c * 4;
  local_58 = uVar6;
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101c50;
  local_50 = (void *)expand(uVar6,iVar3);
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101c5e;
  anti_debug();
  for (local_80 = 0; local_80 < local_7c * 8; local_80 = local_80 + 1) {
    bVar1 = *(byte *)((long)local_50 + (long)local_80);
    *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101c81;
    putchar((uint)bVar1);
  }
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101c9a;
  putchar(10);
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101cb3;
  pFVar7 = fopen("flag.txt","wb");
  __ptr = local_50;
  iVar3 = local_7c << 3;
  local_48 = pFVar7;
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101cd5;
  fwrite(__ptr,1,(long)iVar3,pFVar7);
  pFVar7 = local_48;
  *(undefined8 *)(puVar8 + lVar4 + -8) = 0x101ce1;
  fclose(pFVar7);
  if (local_40 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

Here is the `flipBits` function:
```d
void flipBits(long param_1,int param_2)

{
  bool bVar1;
  byte local_11;
  int local_c;
  
  bVar1 = false;
  local_11 = 0x69;
  for (local_c = 0; local_c < param_2; local_c = local_c + 1) {
    if (bVar1) {
      *(byte *)(param_1 + local_c) = *(byte *)(param_1 + local_c) ^ local_11;
      local_11 = local_11 + 0x20;
    }
    else {
      *(byte *)(param_1 + local_c) = ~*(byte *)(param_1 + local_c);
    }
    bVar1 = !bVar1;
  }
  return;
}
```

Here is the `expand` function:
```d
void * expand(long param_1,int param_2)

{
  bool bVar1;
  void *pvVar2;
  byte local_1d;
  int local_18;
  
  bVar1 = false;
  local_1d = 0x69;
  pvVar2 = malloc((long)(param_2 * 2));
  for (local_18 = 0; local_18 < param_2; local_18 = local_18 + 1) {
    if (bVar1) {
      *(byte *)((long)pvVar2 + (long)(local_18 * 2)) =
           *(byte *)(param_1 + local_18) & 0xf0 | local_1d >> 4;
      *(byte *)((long)pvVar2 + (long)(local_18 * 2) + 1) =
           *(byte *)(param_1 + local_18) & 0xf | local_1d << 4;
    }
    else {
      *(byte *)((long)pvVar2 + (long)(local_18 * 2)) =
           *(byte *)(param_1 + local_18) & 0xf | local_1d << 4;
      *(byte *)((long)pvVar2 + (long)(local_18 * 2) + 1) =
           *(byte *)(param_1 + local_18) & 0xf0 | local_1d >> 4;
    }
    local_1d = local_1d * '\v';
    bVar1 = !bVar1;
  }
  printf("fie");
  return pvVar2;
}
```

The `anti_debug` function:
```d
void anti_debug(void)

{
  long lVar1;
  
  lVar1 = ptrace(PTRACE_TRACEME,0,1,0);
  if (lVar1 == -1) {
    puts("THOU SHALL NOT READ MY MIND WITH GOTHIC MAGIC CAESER!!!\n");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  return;
}
```

So so binary changes the flag using the `flipBits` function and three rounds of `expand`.

I run this script to recover the flag:
```python
#!/usr/bin/env python3
from pathlib import Path
import re

# Inverse of flipBits
def unflipBits(data: bytes) -> bytearray:
    result = bytearray(len(data))
    b = 0x69
    flip = False
    for i, x in enumerate(data):
        if flip:
            result[i] = x ^ (b & 0xFF)
            b = (b + 0x20) & 0xFF
        else:
            result[i] = (~x) & 0xFF
        flip = not flip
    return result

# Inverse of expand
def shrink(data: bytes) -> bytearray:
    if len(data) % 2 != 0:
        raise ValueError("data length must be even")
    n = len(data) // 2
    out = bytearray(n)
    b = 0x69
    toggle = False
    for i in range(n):
        hi = data[2*i]
        lo = data[2*i+1]
        if toggle:
            out[i] = ((hi & 0xF0) | (lo & 0x0F)) & 0xFF
        else:
            out[i] = ((hi & 0x0F) | (lo & 0xF0)) & 0xFF
        b = (b * 11) & 0xFF
        toggle = not toggle
    return out

def main():
    path = Path("palatinepackflag.txt")
    if not path.exists():
        print("palatinepackflag.txt not found.")
        return

    data = path.read_bytes()

    # shrink 3 times then unflipBits
    for _ in range(3):
        data = shrink(data)

    data = unflipBits(data)

    m = re.search(rb"(sunshine\{.*?\})", data, re.IGNORECASE | re.DOTALL)
    if m:
        print("Found flag:", m.group(1).decode(errors="replace"))
    else:
        print("No flag found.")

if __name__ == "__main__":
    main()
```

The `shrink` function does the opposite of `expand` in the binary. The C `expand` function doubles each byte into two output bytes using alternating bitmixing with a running key `b`. `shrink` takes every pair of bytes and reconstructs the original byte. Do this 3 times also like in the binary. 

The `flipBits` function alternates between bitwise NOT on one byte and XOR with a changing key `b` on the next byte. My `unflipBits` function implements this in reverse.

`sunshine{C3A5ER_CR055ED_TH3_RUB1C0N}`