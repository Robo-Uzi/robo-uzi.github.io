---
layout: post
title:  "Locked Temple"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Locked-Temple/
---

**Title**: Locked Temple

**Category**: rev

**Author**: slyfenon

**Description**: Step on the pressure plates with the perfect 8 plate sequence. If you prove worthy, the door will open and the ulimate golden prize will be revealed to you. Pressure plate order follows this representation:
```shell
0  1
2  3

Flag format is upCTF{PlateOrder_SECRETDIGIT} -> example upCTF{00000000_1}

Note: To solve this challange is advised to have a Linux GUI environment and the SDL2 library to run.
```

I get the challenge file:
```shell
file locked_temple  
locked_temple: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=603c1bd819a1d623dad057c954de6303fe32618c, for GNU/Linux 3.2.0, stripped
```

I see these functions in ghidra:
```shell
undefined8 main(void)

{
  undefined4 *puVar1;
  undefined4 uVar2;
  bool bVar3;
  int iVar4;
  undefined8 uVar5;
  ulong uVar6;
  undefined4 *puVar7;
  ulong uVar8;
  int iVar9;
  int iVar10;
  int iVar11;
  int iVar12;
  undefined local_88 [16];
  int local_78 [5];
  int local_64;
  
  iVar10 = 1;
  iVar9 = 1;
  SDL_Init(0x20);
  uVar5 = SDL_CreateWindow("Locked Temple",0x2fff0000,0x2fff0000,0x1e0,0x180,0);
  uVar5 = SDL_CreateRenderer(uVar5,0xffffffff,0);
  FUN_00101bc0();
  do {
    bVar3 = true;
LAB_00101260:
    iVar4 = SDL_PollEvent(local_78);
    while (iVar4 != 0) {
      if (local_78[0] != 0x100) goto code_r0x0010127f;
      bVar3 = false;
      iVar4 = SDL_PollEvent(local_78);
    }
    iVar4 = 0;
    SDL_SetRenderDrawColor(uVar5,0x14,0x14,0x19,0xff);
    SDL_RenderClear(uVar5);
    do {
      iVar11 = 0;
      do {
        iVar12 = iVar11 + 1;
        FUN_00101690(uVar5,iVar11,iVar4);
        iVar11 = iVar12;
      } while (iVar12 != 10);
      iVar4 = iVar4 + 1;
    } while (iVar4 != 8);
    puVar7 = &DAT_00104060;
    do {
      puVar1 = puVar7 + 1;
      uVar2 = *puVar7;
      puVar7 = puVar7 + 2;
      FUN_00101720(uVar5,uVar2,*puVar1);
    } while (puVar7 != (undefined4 *)&DAT_00104080);
    iVar4 = FUN_00101ce0(local_88);
    if (iVar4 != 0) {
      FUN_00101b40(uVar5);
    }
    FUN_001017f0(uVar5,DAT_00104044,DAT_00104040,iVar4);
    FUN_00101990(uVar5,iVar9,iVar10);
    SDL_RenderPresent(uVar5);
    SDL_Delay(0x10);
    if (!bVar3) {
      SDL_Quit();
      return 0;
    }
  } while( true );
code_r0x0010127f:
  if (local_78[0] == 0x300) {
    if (local_64 == 0x77) {
      iVar10 = iVar10 + -1;
    }
    else if (local_64 == 0x73) {
      iVar10 = iVar10 + 1;
    }
    else if (local_64 == 0x61) {
      iVar9 = iVar9 + -1;
    }
    else if (local_64 == 100) {
      iVar9 = iVar9 + 1;
    }
    if (iVar9 < 0) {
      iVar9 = 0;
    }
    if (9 < iVar9) {
      iVar9 = 9;
    }
    if (iVar10 < 0) {
      iVar10 = 0;
    }
    if (7 < iVar10) {
      iVar10 = 7;
    }
    uVar6 = 0;
    uVar8 = uVar6;
    iVar4 = DAT_00104060;
    while( true ) {
      if ((iVar4 == iVar9) && ((&DAT_00104064)[uVar6 * 2] == iVar10)) {
        FUN_00101c40(local_88,uVar8);
      }
      uVar6 = uVar6 + 1;
      if (uVar6 == 4) break;
      uVar8 = uVar6 & 0xffffffff;
      iVar4 = (&DAT_00104060)[uVar6 * 2];
    }
  }
  goto LAB_00101260;
}
```

```shell
void FUN_00101c40(int *param_1,undefined param_2)

{
  byte bVar1;
  bool bVar2;
  byte bVar3;
  int iVar4;
  long lVar5;
  byte bVar6;
  
  if (param_1[3] == 0) {
    iVar4 = *param_1;
    if (iVar4 < 8) {
      lVar5 = (long)iVar4;
      iVar4 = iVar4 + 1;
      *(undefined *)((long)param_1 + lVar5 + 4) = param_2;
      *param_1 = iVar4;
    }
    if (iVar4 == 8) {
      bVar6 = 0xb;
      lVar5 = 1;
      bVar2 = true;
      bVar3 = DAT_00104020 ^ 0x55;
      while( true ) {
        bVar1 = *(byte *)((long)param_1 + lVar5 + 3);
        if (bVar1 != (bVar3 & 3)) {
          bVar2 = false;
        }
        if (lVar5 == 8) break;
        bVar3 = bVar6 ^ 0x55 ^ (&DAT_00104020)[lVar5];
        if ((bVar1 & 1) != 0) {
          bVar3 = bVar3 >> 4;
        }
        lVar5 = lVar5 + 1;
        bVar6 = bVar6 + 0xb;
      }
      if (bVar2) {
        param_1[3] = 1;
        return;
      }
      *param_1 = 0;
      *(undefined8 *)(param_1 + 1) = 0;
      return;
    }
  }
  return;
}
```

```shell
void FUN_00101bc0(undefined4 *param_1)

{
  byte *pbVar1;
  byte bVar2;
  
  pbVar1 = &DAT_00104020;
  bVar2 = 0;
  do {
    *pbVar1 = *pbVar1 ^ bVar2;
    pbVar1 = pbVar1 + 1;
    bVar2 = bVar2 + 5;
  } while (pbVar1 != &DAT_00104028);
  pbVar1 = &DAT_00104018;
  bVar2 = 0;
  do {
    *pbVar1 = *pbVar1 ^ bVar2;
    pbVar1 = pbVar1 + 1;
    bVar2 = bVar2 + 8;
  } while (pbVar1 != &DAT_00104020);
  pbVar1 = &DAT_00104010;
  bVar2 = 0;
  do {
    *pbVar1 = *pbVar1 ^ bVar2;
    bVar2 = bVar2 + 10;
    pbVar1 = pbVar1 + 1;
  } while (bVar2 != 0x50);
  *param_1 = 0;
  param_1[3] = 0;
  *(undefined8 *)(param_1 + 1) = 0;
  return;
}
```

- `FUN_00101bc0` XOR decodes three 8 byte blobs in `.data`
- The third blob (`DAT_00104020`) becomes the key used in the validator
- `FUN_00101c40` checks the 8 step plate sequence
- Each step expects `(bVar3 & 3)` → a value 0-3 (the plate index)

Solve script:
```python
#!/usr/bin/env python3
from itertools import product
from elftools.elf.elffile import ELFFile

BIN = "locked_temple"

# virtual addresses from the binary
ADDR_4010 = 0x4010
ADDR_4018 = 0x4018
ADDR_4020 = 0x4020


def vaddr_to_offset(elf, vaddr):
    for seg in elf.iter_segments():
        start = seg["p_vaddr"]
        end = start + seg["p_memsz"]
        if start <= vaddr < end:
            return seg["p_offset"] + (vaddr - start)
    raise ValueError("address not mapped")


def decode_data(data):
    data = bytearray(data)

    # third block decode (DAT_4020)
    b = 0
    for i in range(8):
        data[0x10 + i] ^= b
        b = (b + 5) & 0xff

    # second block decode
    b = 0
    for i in range(8):
        data[0x08 + i] ^= b
        b = (b + 8) & 0xff

    # first block decode
    b = 0
    for i in range(8):
        data[i] ^= b
        b = (b + 10) & 0xff

    return data


def check(seq, key):
    bVar6 = 0x0B
    bVar3 = key[0] ^ 0x55
    ok = True

    for i in range(8):
        plate = seq[i]
        if plate != (bVar3 & 3):
            ok = False

        if i == 7:
            break

        bVar3 = bVar6 ^ 0x55 ^ key[i + 1]
        if plate & 1:
            bVar3 >>= 4

        bVar6 = (bVar6 + 0x0B) & 0xFF

    return ok


with open(BIN, "rb") as f:
    elf = ELFFile(f)

    off = vaddr_to_offset(elf, ADDR_4010)
    f.seek(off)
    raw = f.read(0x18)

decoded = decode_data(raw)
key = decoded[0x10:0x18]

for seq in product(range(4), repeat=8):
    if check(seq, key):
        order = "".join(map(str, seq))
        print("Plate order:", order)
        break
```

Output:
```shell
python3 solve2.py  
Plate order: 01122301
```
Once you step on these plates in order in the game a `7` appears making the flag: `upCTF{01122301_7}`.