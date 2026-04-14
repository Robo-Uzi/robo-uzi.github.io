---
layout: post
title:  "Batcave Bitflips"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-batcave-bitflips/
---

**Category**: rev medium

**Author**: gregt114 (Greg)

**Description**: Batman's new state-of-the-art AI agent has deleted all of the source code to the Batcave license verification program! There's an old debug version lying around, but that thing has been hit by more cosmic rays than Superman!

I get the challenge file:
```shell
file batcave_license_checker  
batcave_license_checker: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=8f2242bd67c2413d4326d407e9bdd662490fc983, for GNU/Linux 3.2.0, not stripped
```

I put the binary in ghidra and find these functions:
```shell
void decrypt_flag(long param_1)

{
  int local_c;
  
  for (local_c = 0; local_c < 0x20; local_c = local_c + 1) {
    FLAG[local_c] = FLAG[local_c] | *(byte *)(param_1 + local_c % 0x20);
  }
  return;
}
```

```shell
int verify(EVP_PKEY_CTX *ctx,uchar *sig,size_t siglen,uchar *tbs,size_t tbslen)

{
  int iVar1;
  
  iVar1 = memcmp(ctx,EXPECTED,0x20);
  return (int)(iVar1 == 0);
}
```

```shell
undefined8 main(void)

{
  int iVar1;
  FILE *__stream;
  char *pcVar2;
  uchar *__ptr;
  uchar *in_RCX;
  size_t siglen;
  uchar *sig;
  size_t in_R8;
  long in_FS_OFFSET;
  undefined8 local_58;
  undefined8 local_50;
  undefined8 local_48;
  undefined8 local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  undefined local_18;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_38 = 0;
  local_30 = 0;
  local_28 = 0;
  local_20 = 0;
  local_18 = 0;
  local_58 = 0;
  local_50 = 0;
  local_48 = 0;
  local_40 = 0;
  puts("=================================================================================");
  puts("_-_-_-_-_-_-_-_-_-_-_- BATCAVE LICENSE VERIFICATION (Beta) _-_-_-_-_-_-_-_-_-_-_-");
  puts("=================================================================================\n");
  __stream = fdopen(0,"r");
  if (__stream == (FILE *)0x0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  printf("ENTER LICENSE KEY: ");
  pcVar2 = fgets((char *)&local_38,0x21,__stream);
  if (pcVar2 == (char *)0x0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  local_18 = 0;
  puts("COMPUTING...");
  hash(&local_38,&local_58);
  __ptr = (uchar *)bytes_to_hex(&local_58,0x20);
  sig = __ptr;
  printf("HASHED KEY: %s\n");
  free(__ptr);
  puts("VERIFYING...");
  iVar1 = verify((EVP_PKEY_CTX *)&local_58,sig,siglen,in_RCX,in_R8);
  if (iVar1 == 0) {
    puts("INVALID LICENSE - PLEASE CONTACT ALFRED");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  puts("LICENSE GOOD - DECRYPTING BAT DATA...");
  decrypt_flag(&local_58);
  printf("FLAG: %s\n",FLAG);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

I take the `FLAG` array from the binary at `0x00104060` (32 bytes) and the `EXPECTED` hash from binary at `0x00104040` (32 bytes) and XOR them to get the flag:
```python
#!/usr/bin/env python3

FLAG_ENC = bytes([
    0x6e, 0x19, 0x34, 0x49, 0x77, 0x7d, 0xf0, 0x5a,
    0x07, 0xb4, 0x33, 0xa6, 0x8c, 0xe6, 0xe6, 0x17,
    0xfb, 0xe9, 0x6f, 0xae, 0x2e, 0xe5, 0x26, 0xc3,
    0x70, 0xe3, 0xc4, 0x7d, 0x27, 0x7f, 0x2b, 0x00
])

EXPECTED = bytes([
    0x3b, 0x54, 0x75, 0x1a, 0x24, 0x06, 0xaf, 0x05,
    0x77, 0x80, 0x47, 0xc5, 0xe4, 0x83, 0xd3, 0x48,
    0xcb, 0x87, 0x30, 0xde, 0x1a, 0x91, 0x45, 0xab,
    0x15, 0xc7, 0x9b, 0x22, 0x04, 0x02, 0x2b, 0xee
])

flag = bytes(a ^ b for a, b in zip(FLAG_ENC, EXPECTED))
flag = flag.split(b'\x00', 1)[0].decode('ascii')
print(flag)
```
Because `decrypt_flag` applies a bitwise OR with the computed hash state, and the extracted `FLAG` and `EXPECTED` blobs are complementary encodings of this transformation, XOR cleanly reverses the operation.

Output:
```shell
python3 solve5.py  
UMASS{__p4tche5_0n_p4tche$__#}
```

`UMASS{__p4tche5_0n_p4tche$__#}`
