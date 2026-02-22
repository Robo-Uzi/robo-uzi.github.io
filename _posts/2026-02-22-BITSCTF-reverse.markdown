---
layout: post
title:  "Reverse Challenges"
date:   2026-02-22 18:14:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BITSCTF-2026-reverse-challenges/
---
* TOC
{:toc}

## gcc(Ghost C Compiler)

**Author**: Aur0r4

**Category**: reverse

**Description**: I have made a successful safe to use C compiler, try it once it's amazing and manages to be as fast as gcc in respect of runtime.

I get the challenge files. Contents of `README.md`:
```markdown
# Desc  
  
I have made a successful safe to use C compiler, try it once it's amazing and manages to be as fast as gcc in respect of runtime.  
  
## Files  
  
- `ghost_compiler` - The amazing and the most dashing compiler itself.  
  
## Usage  
  
- `./ghost_compiler filename.c` - The compiler is able to compile C binaries which can then be run, for starters -- try it with a basic "Hello World" program in respect of Brian Kernighan.  
  
## Flag Format  
  
`BITSCTF{...}`
```

The binary `ghost_compiler`:
```shell
file ghost_compiler  
ghost_compiler: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=d64d20a0400553456624de78cd58afb878a5eb02, for GNU/Linux 3.2.0, stripped
```

In ghidra I see these functions:
```shell
long FUN_00101349(char *param_1)

{
  bool bVar1;
  FILE *__stream;
  long __off;
  size_t sVar2;
  long in_FS_OFFSET;
  int local_34;
  long local_30;
  char local_18 [8];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  __stream = fopen(param_1,"rb");
  if (__stream == (FILE *)0x0) {
    local_30 = -1;
  }
  else {
    local_30 = 0;
    while( true ) {
      sVar2 = fread(local_18,1,1,__stream);
      if (sVar2 != 1) break;
      if (local_18[0] == DAT_00104020) {
        __off = ftell(__stream);
        sVar2 = fread(local_18,1,7,__stream);
        if (sVar2 == 7) {
          bVar1 = true;
          local_34 = 0;
          while ((local_34 < 7 && (bVar1))) {
            if (local_18[local_34] != (&DAT_00104020)[local_34 + 1]) {
              bVar1 = false;
            }
            local_34 = local_34 + 1;
          }
          if (bVar1) {
            fclose(__stream);
            goto LAB_0010149f;
          }
        }
        fseek(__stream,__off,0);
      }
      local_30 = local_30 + 1;
    }
    fclose(__stream);
    local_30 = -1;
  }
LAB_0010149f:
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return local_30;
}
```

```shell
ulong FUN_001014b5(char *param_1,long param_2)

{
  int iVar1;
  FILE *__stream;
  ulong local_20;
  long local_18;
  
  __stream = fopen(param_1,"rb");
  if (__stream == (FILE *)0x0) {
    local_20 = 0;
  }
  else {
    local_20 = 0xcbf29ce484222325;
    local_18 = 0;
    while( true ) {
      iVar1 = fgetc(__stream);
      if (iVar1 == -1) break;
      if (((param_2 < 0) || (local_18 < param_2)) || (param_2 + 0x3f < local_18)) {
        local_20 = ((long)iVar1 ^ local_20) * 0x100000001b3;
        local_18 = local_18 + 1;
      }
      else {
        local_18 = local_18 + 1;
      }
    }
    fclose(__stream);
    local_20 = local_20 ^ 0xcafebabe00000000;
  }
  return local_20;
}
```

```shell
undefined8 FUN_00101583(long param_1,long param_2,ulong param_3)

{
  undefined8 uVar1;
  long in_FS_OFFSET;
  int local_24;
  ulong local_20;
  byte local_18 [8];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_20 = param_3;
  for (local_24 = 0; local_24 < 8; local_24 = local_24 + 1) {
    local_18[local_24] = (byte)local_20 ^ *(byte *)(param_1 + param_2 + local_24);
    local_20 = local_20 >> 1 | (ulong)((local_20 & 1) != 0) << 0x3f;
  }
  if (((((local_18[0] == 0x42) && (local_18[1] == 'I')) && (local_18[2] == 'T')) &&
      ((local_18[3] == 'S' && (local_18[4] == 'C')))) &&
     ((local_18[5] == 'T' && ((local_18[6] == 'F' && (local_18[7] == '{')))))) {
    uVar1 = 1;
  }
  else {
    uVar1 = 0;
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return uVar1;
}

```

```shell
int FUN_0010165f(int param_1,char **param_2)

{
  int iVar1;
  long lVar2;
  long lVar3;
  FILE *pFVar4;
  size_t sVar5;
  void *__ptr;
  size_t sVar6;
  undefined8 *puVar7;
  long in_FS_OFFSET;
  byte bVar8;
  int local_458;
  undefined8 local_418;
  undefined8 local_410;
  undefined8 local_408 [127];
  long local_10;
  
  bVar8 = 0;
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  lVar2 = FUN_00101349(*param_2);
  if (lVar2 == -1) {
    iVar1 = 1;
  }
  else {
    lVar3 = FUN_001014b5(*param_2,lVar2);
    if (lVar3 == 0) {
      iVar1 = 1;
    }
    else {
      pFVar4 = fopen(*param_2,"rb");
      if (pFVar4 == (FILE *)0x0) {
        iVar1 = 1;
      }
      else {
        fseek(pFVar4,0,2);
        sVar5 = ftell(pFVar4);
        fseek(pFVar4,0,0);
        __ptr = malloc(sVar5);
        if (__ptr == (void *)0x0) {
          fclose(pFVar4);
          iVar1 = 1;
        }
        else {
          sVar6 = fread(__ptr,1,sVar5,pFVar4);
          if (sVar6 == sVar5) {
            fclose(pFVar4);
            if (lVar2 + 0x3f < (long)sVar5) {
              iVar1 = FUN_00101583(__ptr,lVar2,lVar3);
              if (iVar1 == 0) {
                free(__ptr);
                iVar1 = 1;
                goto LAB_00101a95;
              }
              memset((void *)((long)__ptr + lVar2),0,0x40);
            }
            iVar1 = unlink(*param_2);
            if (iVar1 == 0) {
              pFVar4 = fopen(*param_2,"wb");
              if (pFVar4 == (FILE *)0x0) {
                free(__ptr);
                iVar1 = 1;
              }
              else {
                fwrite(__ptr,1,sVar5,pFVar4);
                fclose(pFVar4);
                free(__ptr);
                chmod(*param_2,0x1ed);
                local_418 = 0x20636367;
                local_410 = 0;
                puVar7 = local_408;
                for (lVar2 = 0x7e; lVar2 != 0; lVar2 = lVar2 + -1) {
                  *puVar7 = 0;
                  puVar7 = puVar7 + (ulong)bVar8 * -2 + 1;
                }
                for (local_458 = 1; local_458 < param_1; local_458 = local_458 + 1) {
                  strcat((char *)&local_418,param_2[local_458]);
                  sVar5 = strlen((char *)&local_418);
                  *(undefined2 *)((long)&local_418 + sVar5) = 0x20;
                  strcmp(param_2[local_458],"-o");
                }
                iVar1 = system((char *)&local_418);
                if (iVar1 == 0) {
                  iVar1 = 0;
                }
              }
            }
            else {
              free(__ptr);
              iVar1 = 1;
            }
          }
          else {
            free(__ptr);
            fclose(pFVar4);
            iVar1 = 1;
          }
        }
      }
    }
  }
LAB_00101a95:
  if (local_10 == *(long *)(in_FS_OFFSET + 0x28)) {
    return iVar1;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

In ghidra I also see this:
```shell
			DAT_00104020                                    XREF[2]:     FUN_00101349:001013a6(R), 
FUN_00101349:0010140a(*)  
00104020 9a              undefined1 9Ah
            DAT_00104021                                    XREF[1]:     FUN_00101349:00101411(*)  
00104021 a5              ??         A5h
00104022 22              ??         22h    "
00104023 e8              ??         E8h
00104024 1e              ??         1Eh
00104025 fa              ??         FAh
00104026 91              ??         91h
00104027 90              ??         90h
```
The flag is hidden in the 64 byte data blob starting at the magic sequence (`0x9a 0xa5 0x22 0xe8 0x1e 0xfa 0x91 0x90`) in the `ghost_compiler` binary. This blob is encrypted using a rolling XOR based on a custom FNV-1a hash of the binarys contents (skipping those 64 bytes). The compilers logic checks only the first 8 decrypted bytes for `BITSCTF{` but the encryption scheme extends to the full 64 bytes which contain the entire flag.

Solve script:
```python
import os
import sys

def main():
    filename = 'ghost_compiler'
    if not os.path.exists(filename):
        print(f"Error: {filename} not found.")
        sys.exit(1)

    with open(filename, 'rb') as f:
        content = f.read()

    magic = b'\x9a\xa5\x22\xe8\x1e\xfa\x91\x90'
    offset = content.find(magic)
    if offset == -1:
        print("Error: Magic sequence not found.")
        sys.exit(1)

    encrypted = content[offset:offset + 64]
    if len(encrypted) != 64:
        print("Error: Incomplete encrypted blob.")
        sys.exit(1)

    # Compute hash. skipping the 64 byte blob
    h = 0xcbf29ce484222325
    prime = 0x100000001b3
    mask = (1 << 64) - 1
    for i in range(len(content)):
        if not (offset <= i < offset + 64):
            h = ((h ^ content[i]) * prime) & mask
    h ^= 0xcafebabe00000000

    # Decrypt with rolling XOR
    flag_bytes = []
    current_h = h
    for b in encrypted:
        plain = b ^ (current_h & 0xff)
        flag_bytes.append(plain)
        current_h = (current_h >> 1) | ((current_h & 1) << 63)

    try:
        flag = bytes(flag_bytes).decode('ascii')
        print(f"Recovered flag: {flag}")
    except UnicodeDecodeError:
        print("Error: Decrypted data is not valid ASCII.")

if __name__ == "__main__":
    main()
```
To get the flag the script reads the `ghost_compiler` file into memory. It finds the file offset of the magic sequence embedded in the `.data` section. Then computes the hash of the file, skipping the 64 bytes starting at that offset. Then use the hash to decrypt the 64 bytes with rolling XOR (right rotate by 1 after each byte). 

`BITSCTF{n4n0m1t3s_4nd_s3lf_d3struct_0ur0b0r0s}`
