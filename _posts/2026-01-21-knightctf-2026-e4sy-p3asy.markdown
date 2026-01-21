---
layout: post
title:  "E4sy P3asy"
date:   2026-01-21 15:51:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /knightctf-2026-e4sy-p3asy/
---

**Title**: E4sy P3asy

**Author**: NomanProdhan

**Category**: Rev

**Description**: An easy RE...

I get the executable:
```shell
file E4sy_P3asy.ks  
E4sy_P3asy.ks: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ccaa420882d2b8eb1bf3a2cc82b527aa19911512, for GNU/Linux 4.4.0, stripped
```

Running `strings` I see some interesting things along with what looks like md5 hashes:
```shell
strings -n 8 E4sy_P3asy.ks  
/lib64/ld-linux-x86-64.so.2  
__gmon_start__  
_ITM_deregisterTMCloneTable  
_ITM_registerTMCloneTable  
EVP_MD_CTX_free  
EVP_MD_CTX_new  
EVP_DigestFinal_ex  
EVP_DigestUpdate  
EVP_DigestInit_ex  
snprintf  
__stack_chk_fail  
__libc_start_main  
__cxa_finalize  
libcrypto.so.3  
libc.so.6  
OPENSSL_3.0.0  
GLIBC_2.4  
GLIBC_2.34  
GLIBC_2.2.5  
AWAVAUATUSH  
[]A\A]A^A_  
D$PG00gH  
s@lt_202f  
D$VCTF_H  
D$PKnigH  
CTF_2026f  
D$_s@ltH  
GoogleCTF{  
Try again!  
Good job! You got it!  
========================================  
  E4sy P3asy - KnightCTF 2026  
[*] Enter the flag to prove your worth!  
[!] Interesting... but that's a decoy flag from a different universe.  
[!] You're in KnightCTF, not GoogleCTF :)  
[?] Format looks suspicious... but not quite.  
781011edfb2127ee5ff82b06bb1d2959  
4cf891e0ddadbcaae8e8c2dc8bb15ea0  
d06d0cbe140d0a1de7410b0b888f22b4
```

Putting it in ghidra I see this main function
```d

undefined8 FUN_00101140(void)

{
  ulong uVar1;
  int iVar2;
  char *pcVar3;
  size_t sVar4;
  ulong uVar5;
  long lVar6;
  char *pcVar7;
  char *pcVar8;
  undefined4 *puVar9;
  long in_FS_OFFSET;
  byte bVar10;
  char local_338 [48];
  undefined4 local_308;
  undefined2 local_304;
  undefined4 local_302;
  undefined4 uStack_2fe;
  undefined4 local_2fa;
  undefined uStack_2f6;
  undefined uStack_2f5;
  char local_2c8 [127];
  char acStack_249 [6];
  char local_243 [5];
  char local_23e [235];
  undefined auStack_153 [5];
  undefined auStack_14e [6];
  char local_148 [264];
  long local_40;
  
  bVar10 = 0;
  local_40 = *(long *)(in_FS_OFFSET + 0x28);
  pcVar7 = acStack_249 + 1;
  puts("========================================");
  puts("   E4sy P3asy - KnightCTF 2026");
  puts("========================================");
  puts("[*] Enter the flag to prove your worth!");
  puts("");
  printf("flag> ");
  pcVar3 = fgets(pcVar7,0x100,stdin);
  if (pcVar3 != (char *)0x0) {
    sVar4 = strlen(pcVar7);
    if (sVar4 != 0) {
      pcVar3 = acStack_249 + sVar4;
      do {
        if ((*pcVar3 != '\n') && (*pcVar3 != '\r')) break;
        *pcVar3 = '\0';
        pcVar3 = pcVar3 + -1;
      } while (pcVar3 + (1 - (long)pcVar7) != (char *)0x0);
    }
    sVar4 = strlen(pcVar7);
    FUN_00101660(pcVar7,sVar4,local_148);
    iVar2 = FUN_00101760(pcVar7,"GoogleCTF{");
    if ((iVar2 != 0) && (iVar2 = FUN_001017a0(pcVar7), iVar2 != 0)) {
      sVar4 = strlen(pcVar7);
      uVar1 = sVar4 - 0xb;
      if (uVar1 < 0x100) {
        pcVar3 = local_23e;
        pcVar8 = local_148;
        for (uVar5 = uVar1; uVar5 != 0; uVar5 = uVar5 - 1) {
          *pcVar8 = *pcVar3;
          pcVar3 = pcVar3 + (ulong)bVar10 * -2 + 1;
          pcVar8 = pcVar8 + (ulong)bVar10 * -2 + 1;
        }
        auStack_153[sVar4] = 0;
        puVar9 = &local_308;
        for (lVar6 = 0x10; lVar6 != 0; lVar6 = lVar6 + -1) {
          *puVar9 = 0;
          puVar9 = puVar9 + (ulong)bVar10 * -2 + 1;
        }
        local_308 = 0x67303047;
        local_304 = 0x656c;
        local_302 = 0x5f465443;
        uStack_2fe = 0x746c4073;
        local_2fa = 0x3230325f;
        uStack_2f6 = 0x36;
        uStack_2f5 = 0;
        if (uVar1 == 0xd) {
          lVar6 = 0;
          do {
            snprintf(local_2c8,0x80,"%s%zu%c",&local_308,lVar6,(ulong)(uint)(int)local_148[lVar6]);
            sVar4 = strlen(local_2c8);
            FUN_00101660(local_2c8,sVar4,local_338);
            iVar2 = strcmp(local_338,(&PTR_s_8af7f29b21564b87336ed4e4cdfb1a20_00103d60)[lVar6]);
            if (iVar2 != 0) goto LAB_00101289;
            lVar6 = lVar6 + 1;
          } while (lVar6 != 0xd);
          puts("[!] Interesting... but that\'s a decoy flag from a different universe.");
          puts("[!] You\'re in KnightCTF, not GoogleCTF :)");
          puts("Try again!");
          goto LAB_00101300;
        }
      }
    }
LAB_00101289:
    iVar2 = FUN_00101760(pcVar7,&DAT_00102035);
    if ((iVar2 == 0) && (iVar2 = FUN_00101760(pcVar7,"FLAG{"), iVar2 == 0)) {
      iVar2 = FUN_00101760(pcVar7,&DAT_00102034);
      if ((iVar2 != 0) && (iVar2 = FUN_001017a0(pcVar7), iVar2 != 0)) {
        sVar4 = strlen(pcVar7);
        uVar1 = sVar4 - 6;
        if (uVar1 < 0x100) {
          pcVar7 = local_243;
          pcVar3 = local_148;
          for (uVar5 = uVar1; uVar5 != 0; uVar5 = uVar5 - 1) {
            *pcVar3 = *pcVar7;
            pcVar7 = pcVar7 + (ulong)bVar10 * -2 + 1;
            pcVar3 = pcVar3 + (ulong)bVar10 * -2 + 1;
          }
          auStack_14e[sVar4] = 0;
          puVar9 = &local_308;
          for (lVar6 = 0x10; lVar6 != 0; lVar6 = lVar6 + -1) {
            *puVar9 = 0;
            puVar9 = puVar9 + (ulong)bVar10 * -2 + 1;
          }
          local_308 = 0x67696e4b;
          local_304 = 0x7468;
          local_302 = 0x5f465443;
          uStack_2fe = 0x36323032;
          local_2fa = 0x6c40735f;
          uStack_2f6 = 0x74;
          if (uVar1 == 0x17) {
            lVar6 = 0;
            do {
              snprintf(local_2c8,0x80,"%s%zu%c",&local_308,lVar6,(ulong)(uint)(int)local_148[lVar6])
              ;
              sVar4 = strlen(local_2c8);
              FUN_00101660(local_2c8,sVar4,local_338);
              iVar2 = strcmp(local_338,(&PTR_s_781011edfb2127ee5ff82b06bb1d2959_00103ca0)[lVar6]);
              if (iVar2 != 0) goto LAB_001012f2;
              lVar6 = lVar6 + 1;
            } while (lVar6 != 0x17);
            puts("Good job! You got it!");
            goto LAB_00101300;
          }
        }
      }
LAB_001012f2:
      puts("Try again!");
    }
    else {
      puts("[?] Format looks suspicious... but not quite.");
      puts("Try again!");
    }
  }
LAB_00101300:
  if (local_40 == *(long *)(in_FS_OFFSET + 0x28)) {
    return 0;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

The little endian constants decode to this ascii (`KnightCTF_2026_s@lt`):
```shell
|Hex        |    ASCII |

|0x67696e4b |    Knig  |
|0x7468     |    ht    |
|0x5f465443 |    CTF_  |
|0x36323032 |    2026  |
|0x6c40735f |    _s@l  |
|0x74       |    t     |
```

So the program does `MD5("KnightCTF_2026_s@lt" + index + character)` for each character and compares it against the array of md5 hashes I saw in the strings output.

Solve script:
```python
import hashlib
import string

hashes = [
    "781011edfb2127ee5ff82b06bb1d2959",
    "4cf891e0ddadbcaae8e8c2dc8bb15ea0",
    "d06d0cbe140d0a1de7410b0b888f22b4",
    "d44c9a9b9f9d1c28d0904d6a2ee3e109",
    "e20ab37bee9d2a1f9ca3d914b0e98f09",
    "d0beea4ce1c12190db64d10a82b96ef8",
    "ac87da74d381d253820bcf4e5f19fcea",
    "ce3f3a34a04ba5e5142f5db272b6cb1f",
    "13843aca227ef709694bbfe4e5a32203",
    "ca19a4c4eb435cb44d74c1e589e51a10",
    "19edec8e46bdf97e3018569c0a60baa3",
    "972e078458ce3cb6e32f795ff4972718",
    "071824f6039981e9c57725453e005beb",
    "66cd6098426b0e69e30e7fa360310728",
    "f78d152df5d277d0ab7d25fb7d1841f3",
    "dba3a36431c4aaf593566f7421abaa22",
    "8820bbdad85ebee06632c379231cfb6b",
    "722bc7cde7d548b81c5996519e1b0f0f",
    "c2862c390c830eb3c740ade576d64773",
    "94da978fe383b341f9588f9bab246774",
    "bea3bb724dbd1704cf45aea8e73c01e1",
    "ade2289739760fa27fd4f7d4ffbc722d",
    "3cd0538114fe416b32cdd814e2ee57b3",
    "8af7f29b21564b87336ed4e4cdfb1a20",
    "4d3f509284784ab67818b13e74fd5ebe",
    "5a45666e1387ff739eb470f840532099",
    "c96997e23323e502d5d0b07d24a68d50",
    "efb388057adc3fe734d8b6ffb2bdd1e1",
    "6df2aaf58e39f89d7a31a72e19e0efbf",
    "9bbd077d3df7faedfd22fae85043d6c0",
    "dc72837ea6ba778eebbd0401a35182f4",
    "efa0bc7c5da3545ba15548b4b85eaf76",
    "1c36dbad9144b1e1d23ddecb5d4df3e9",
    "980663d212aed1ba720f7735873fa73c",
    "9df295c8874bcac93c98babe78b9c946",
    "2672e4d30c7da0d30749f09bc1b1eefa"
]

salt = "KnightCTF_2026_s@lt"
charset = string.printable

flag = ""

for i, target in enumerate(hashes):
    for c in charset:
        data = f"{salt}{i}{c}".encode()
        if hashlib.md5(data).hexdigest() == target:
            flag += c
            print(flag)
            break

print("\nFINAL FLAG:")
print("KCTF{" + flag + "}")
```

Output:
```shell
python3 solve.py  
_  
_L  
_L0  
_L0T  
_L0TS  
_L0TS_  
_L0TS_o  
_L0TS_oF  
_L0TS_oF_  
_L0TS_oF_b  
_L0TS_oF_bR  
_L0TS_oF_bRu  
_L0TS_oF_bRuT  
_L0TS_oF_bRuTE  
_L0TS_oF_bRuTE_  
_L0TS_oF_bRuTE_f  
_L0TS_oF_bRuTE_fo  
_L0TS_oF_bRuTE_foR  
_L0TS_oF_bRuTE_foRC  
_L0TS_oF_bRuTE_foRCE  
_L0TS_oF_bRuTE_foRCE_  
_L0TS_oF_bRuTE_foRCE_:  
_L0TS_oF_bRuTE_foRCE_:P  
  
FINAL FLAG:  
KCTF{_L0TS_oF_bRuTE_foRCE_:P}

## verify

./E4sy_P3asy.ks  
========================================  
  E4sy P3asy - KnightCTF 2026  
========================================  
[*] Enter the flag to prove your worth!  
  
flag> KCTF{_L0TS_oF_bRuTE_foRCE_:P}  
Good job! You got it!
```

`KCTF{_L0TS_oF_bRuTE_foRCE_:P}`