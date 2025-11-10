---
layout: post
title:  "Buckeye 2025 polyglot challenges"
date:   2025-11-10 18:03:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /buckeye-polyglot/
---
* TOC
{:toc}

## Monoglot

**Category**: polyglot

**Description**: Sponsored challenge by [**Battelle**](https://www.battelle.org/). The Chimera is many beasts in one. Craft a single executable that does the same operations on as many architectures as possible. 
`ncat --ssl 28QACfqXmUXQ6EWyj.polyglot.challs.pwnoh.io 1337`

Connecting to the remote service I see this:
```shell
ncat --ssl 28QACfqXmUXQ6EWyj.polyglot.challs.pwnoh.io 1337  
╭───────────────────────────────────────────────────────────────────────────────╮  
│                                                                               │  
│                                     Polyglot                                  │   
│                                                                               │  
╰───────────────────────────────────────────────────────────────────────────────╯
  
Craft a single binary chimera that is the same bytes but runs on many architectures.  
  
Supported architectures:  
 • arm_le  
 • arm_be  
 • arm_thumb_le  
 • aarch64_le  
 • mips_le  
 • mips_be  
 • x86  
 • x86_64  
 • riscv64  
 • powerpc32  
  
Goal:  
 • Syscall number: 237  
 • Arg1: 38085  
 • Arg2: pointer to the string "Battelle"  
    
Submit your shellcode (in hex):  
>
```

The shellcode must:
- put `38085` into arg1 (register `r0`)
- put the address of the ASCII string `"Battelle"` into arg2 (register `r1`)
- put the syscall number `237` into the syscall register (`r7` on 32 bit ARM)
- invoke the syscall (`svc #0`).

I run `pip install keystone-engine` which is a multi architecture assembler framework. First start with the `arm_le` architecture. 

I run this python script to generate the shellcode in hex:
```python
from keystone import Ks, KS_ARCH_ARM, KS_MODE_ARM

ks = Ks(KS_ARCH_ARM, KS_MODE_ARM)

# instructions before the mov pc sequence
# r0 = 38085
instrs_before = ["movw r0, #0x94c5"]

# instructions after the mov/add sequence
instrs_after = ["mov r7, #237", "svc #0"]

# assemble before and after
enc_before, _ = ks.asm("\n".join(instrs_before))
bytes_before = bytes(enc_before)
enc_after, _ = ks.asm("\n".join(instrs_after))
bytes_after = bytes(enc_after)

# assemble mov r1, pc and add r1, #imm with computed imm
# assemble mov r1, pc to get its size
enc_movpc, _ = ks.asm("mov r1, pc")
bytes_movpc = bytes(enc_movpc)
len_movpc = len(bytes_movpc)

# assemble placeholder add r1, #0 to get encoding size
enc_add0, _ = ks.asm("add r1, r1, #0")
bytes_add0 = bytes(enc_add0)
len_add0 = len(bytes_add0)

data = b"Battelle\x00"

pad_len = (4 - ((len(bytes_before) + len_movpc + len_add0 + len(bytes_after)) % 4)) % 4

# compute PC value at time of executing mov r1, pc:
# when mov r1, pc executes, PC = address_of_mov + 8
address_of_mov = len(bytes_before)
pc_at_mov = address_of_mov + 8

# address where data will be located (using placeholder add size initially)
address_of_data = len(bytes_before) + len_movpc + len_add0 + len(bytes_after) + pad_len

# imm we need to add to PC value to reach data
imm = address_of_data - pc_at_mov
if imm < 0 or imm > 4095:
    raise SystemExit(f"Computed imm out of range")

# assemble add with the computed imm
add_instr = f"add r1, r1, #{imm}"
enc_add, _ = ks.asm(add_instr)
bytes_add = bytes(enc_add)

# now recompute pad in case add encoding size differs
pad_len = (4 - ((len(bytes_before) + len_movpc + len(bytes_add) + len(bytes_after)) % 4)) % 4
padding = b"\x00" * pad_len

# final shell
shell = bytes_before + bytes_movpc + bytes_add + bytes_after + padding + data

pc_at_mov = len(bytes_before) + 8
address_of_data = len(bytes_before) + len_movpc + len(bytes_add) + len(bytes_after) + pad_len
loaded_address = pc_at_mov + imm
if loaded_address != address_of_data:
    raise SystemExit(f"Address mismatch after final assembly :(")

print(shell.hex())
```

Output:
```shell
python3 arm_le.py  
c50409e30f10a0e1081081e2ed70a0e3000000ef42617474656c6c6500
```

Sending it to the server gets the first flag:
```shell
Submit your shellcode (in hex):  
> c50409e30f10a0e1081081e2ed70a0e3000000ef42617474656c6c6500  
  
Shellcode length: 29 bytes  
arm_le..........✓ Success (5 instructions, 0.00042 seconds)  
arm_be..........✗ Wrong syscall number: got 0, expected 237  
arm_thumb_le....✗ Exceeded instruction limit (8192)  
aarch64_le......✗ Wrong syscall number: got 0, expected 237  
mips_le.........✗ Wrong syscall number: got 0, expected 237  
mips_be.........✗ Wrong syscall number: got 0, expected 237  
x86.............✗ Error at 0x1000000: Invalid memory read (UC_ERR_READ_UNMAPPED)
x86_64..........✗ Error at 0x1000014: Invalid instruction (UC_ERR_INSN_INVALID)  
riscv64.........✗ Error at 0x100000e: Invalid memory write (UC_ERR_WRITE_UNMAPPED)  
powerpc32.......✗ Wrong syscall number: got 0, expected 237  
  
You got 1/10.  
 • Monoglot flag:   bctf{a_good_start_4fc9c43c0d95}
```

`bctf{a_good_start_4fc9c43c0d95}`

___

## Diglot

**Category**: polyglot

**Description**: Sponsored challenge by [**Battelle**](https://www.battelle.org/). The Chimera is many beasts in one. Craft a single executable that does the same operations on as many architectures as possible. 
`ncat --ssl 28QACfqXmUXQ6EWyj.polyglot.challs.pwnoh.io 1337`

I decide to try `arm_thumb_le` because I have `arm_le` done and they are both little endian. 

I look it up and arm (32 bit) and thumb (16/32 bit) instruction sets use different instruction sizes and different decoders, but many byte sequences decode to valid instructions under both decoders. 

The first bytes are chosen so that when the emulator runs in thumb mode, those bytes form a stub which transfers control cleanly, so the rest of the code executes. 

When the emulator runs in arm mode the same first 4 bytes decode as arm nops. Then the arm shellcode can run no problem as well.

I run this script:
```python
from keystone import Ks, KS_ARCH_ARM, KS_MODE_ARM, KS_MODE_THUMB

# switches to ARM mode safely
ks_thumb = Ks(KS_ARCH_ARM, KS_MODE_THUMB)
thumb_code, _ = ks_thumb.asm("""
    bx pc
    mov r8, r8
""")
thumb_bytes = bytes(thumb_code)

# ARM payload
ks_arm = Ks(KS_ARCH_ARM, KS_MODE_ARM)
arm_code, _ = ks_arm.asm("""
    movw r0, #0x94c5
    mov  r7, #237
    add  r1, pc, #0
    svc  #0
""")
arm_bytes = bytes(arm_code)

data = b"Battelle\x00"

shellcode = thumb_bytes + arm_bytes + data

print(shellcode.hex())
```

Output:
```shell
python3 arm-le-arm-thumb-le.py  
7847c046c50409e3ed70a0e300108fe2000000ef42617474656c6c6500
```

Get the second flag:
```
Submit your shellcode (in hex):  
> 7847c046c50409e3ed70a0e300108fe2000000ef42617474656c6c6500  
  
Shellcode length: 29 bytes  
arm_le........✓ Success (5 instructions, 0.00028 seconds)  
arm_be........✗ Error at 0x1000000: Invalid memory write (UC_ERR_WRITE_UNMAPPED) 
arm_thumb_le..✓ Success (5 instructions, 0.00021 seconds)  
aarch64_le....✗ Wrong syscall number: got 0, expected 237  
mips_le.......✗ Wrong syscall number: got 0, expected 237  
mips_be.......✗ Wrong syscall number: got 0, expected 237  
x86...........✗ Error at 0x1000002: Invalid memory read (UC_ERR_READ_UNMAPPED) 
x86_64........✗ Error at 0x1000002: Invalid memory read (UC_ERR_READ_UNMAPPED) 
riscv64.......✗ Error at 0x1000000: Invalid memory read (UC_ERR_READ_UNMAPPED) 
powerpc32.....✗ Wrong syscall number: got 0, expected 237  
  
You got 2/10.  
 • Monoglot flag:   bctf{a_good_start_4fc9c43c0d95}  
 • Diglot flag:     bctf{now_you_are_bilingual_d9731b3bf6bf}
```

`bctf{now_you_are_bilingual_d9731b3bf6bf}`

___
