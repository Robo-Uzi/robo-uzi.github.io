---
layout: post
title:  "Rev Challenges"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /boroCTF-2026-rev/
---
* TOC
{:toc}

## Hidden but definitely not

**Author**: Franklin

**Category**: Rev

**Description**: The most trite challenge concept.

I get the challenge file:
```shell
file password_protected  
password_protected: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=22484cb1eb941eb87482b6d24060bb6edce1d795, for GNU/Linux 3.2.0, stripped
```

In ghidra I found this function:
```shell
undefined8 FUN_00101229(void)

{
  int iVar1;
  size_t sVar2;
  long in_FS_OFFSET;
  int local_1ac;
  byte local_1a8 [4];
  undefined local_1a4;
  undefined local_1a3;
  undefined local_1a2;
  undefined local_1a1;
  undefined local_1a0;
  undefined local_19f;
  undefined local_19e;
  undefined local_19d;
  undefined local_19c;
  undefined local_19b;
  undefined local_19a;
  undefined local_199;
  undefined local_198;
  undefined local_197;
  undefined local_196;
  undefined local_195;
  undefined local_194;
  undefined local_193;
  undefined local_192;
  undefined local_191;
  undefined local_190;
  undefined local_18f;
  undefined local_18e;
  undefined local_18d;
  undefined local_18c;
  undefined local_18b;
  undefined local_18a;
  undefined local_189;
  undefined local_188;
  undefined local_187;
  undefined8 pass_part1;
  undefined8 pass_part2;
  undefined auStack_11a [10];
  undefined8 local_110;
  undefined8 local_108;
  undefined8 local_100;
  undefined8 local_f8;
  undefined8 local_f0;
  undefined8 local_e8;
  undefined8 local_e0;
  undefined8 local_d8;
  undefined8 local_d0;
  undefined8 local_c8;
  undefined8 local_c0;
  undefined8 local_b8;
  undefined8 local_b0;
  char local_a8 [136];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  local_1a8[0] = 0x65;
  local_1a8[1] = 0x68;
  local_1a8[2] = 0x75;
  local_1a8[3] = 0x68;
  local_1a4 = 0x44;
  local_1a3 = 0x53;
  local_1a2 = 0x41;
  local_1a1 = 0x7c;
  local_1a0 = 0x4e;
  local_19f = 0x58;
  local_19e = 0x4f;
  local_19d = 0x3f;
  local_19c = 0x58;
  local_19b = 0x4a;
  local_19a = 0x47;
  local_199 = 0x30;
  local_198 = 0x6e;
  local_197 = 0x69;
  local_196 = 0x60;
  local_195 = 0x58;
  local_194 = 0x54;
  local_193 = 0x73;
  local_192 = 0x55;
  local_191 = 0x36;
  local_190 = 0x69;
  local_18f = 0x60;
  local_18e = 0x32;
  local_18d = 0x58;
  local_18c = 100;
  local_18b = 0x4f;
  local_18a = 0x66;
  local_189 = 0x6b;
  local_188 = 0x74;
  local_187 = 0x7a;
  pass_part1 = 0x6174533565746152;
  pass_part2 = 0x7372;
  auStack_11a._2_8_ = 0;
  local_110 = 0;
  local_108 = 0;
  local_100 = 0;
  local_f8 = 0;
  local_f0 = 0;
  local_e8 = 0;
  local_e0 = 0;
  local_d8 = 0;
  local_d0 = 0;
  local_c8 = 0;
  local_c0 = 0;
  local_b8 = 0;
  local_b0 = 0;
  printf("Give me the password (youll never find it it\'s just tooooo hard)\n> ");
  fgets(local_a8,0x80,stdin);
  sVar2 = strcspn(local_a8,"\n");
  local_a8[sVar2] = '\0';
  sVar2 = strlen((char *)&pass_part1);
  *(undefined8 *)((long)&pass_part1 + sVar2) = 0x4765737561636542;
  *(undefined8 *)((long)&stack0xfffffffffffffee0 + sVar2) = 0x6c61684374616572;
  *(undefined8 *)((long)&stack0xfffffffffffffee0 + sVar2 + 6) = 0x65676e656c6c61;
  iVar1 = strcmp(local_a8,(char *)&pass_part1);
  if (iVar1 == 0) {
    puts("wow you really got me this time. if only i used better obfuscation techniques.");
    local_1ac = 0;
    while( true ) {
      sVar2 = strlen((char *)local_1a8);
      if (sVar2 <= (ulong)(long)local_1ac) break;
      putchar((int)(char)(local_1a8[local_1ac] ^ 7));
      local_1ac = local_1ac + 1;
    }
    putchar(10);
  }
  else {
    puts("My disappointment is immeasurable.");
  }
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

I took the values that make up the flag and convert them to little endian. Then just do `from hex` in cyberchef. [cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')&input=NTIgNjEgNzQgNjUgMzUgNTMgNzQgNjEgNzIgNzMgNDIgNjUgNjMgNjEgNzUgNzMgNjUgNDcgNzIgNjUgNjEgNzQgNDMgNjggNjEgNmMgNmMgNjUgNmUgNjcgNjU)

```shell
./password_protected  
Give me the password (youll never find it it's just tooooo hard)  
> Rate5StarsBecauseGreatChallenge  
wow you really got me this time. if only i used better obfuscation techniques.  
boroCTF{I_H8_M@7ing_StR1ng5_cHals}
```

`boroCTF{I_H8_M@7ing_StR1ng5_cHals}`

___

## George Orwell

**Author**: Franklin

**Category**: Rev

**Description**: Big Brother says: We're always watching. Your words, no matter how silent, will be heard. **Note** - This challenge simulates real malware but contains **NO** malicious payloads.

I download the challenge file:
```shell
file big_brother  
big_brother: PE32 executable for MS Windows 5.00 (GUI), Intel i386, 4 sections  

strings big_brother | grep boro  
:*:iloveboroctf::
```

I found out that when you see syntax like `SetWorkingDir %A_ScriptDir%`, `Gui`, and `:*:`, that this is an AutoHotkey (AHK) executable. Standard AHK "compilation" doesn't actually convert the script into machine code. 

The flag is encoded in ASCII decimal code:
```shell
strings big_brother | grep -C 7 boro  
SetWorkingDir %A_ScriptDir%  
#SingleInstance Force  
Gui, Color, Black  
Gui, Font, s14 cGreen, Consolas  
Gui, Add, Text, w250 Center, [ WE ARE LISTENING ]  
Gui, Show, w300 h100, System Monitor  
return  
:*:iloveboroctf::  
secret := Chr(98) . Chr(111) . Chr(114) . Chr(111) . Chr(67) . Chr(84) . Chr(70) . Chr(123)  
secret := secret . Chr(65) . Chr(72) . Chr(75) . Chr(95) . Chr(49) . Chr(115) . Chr(95)  
secret := secret . Chr(108) . Chr(73) . Chr(115) . Chr(43) . Chr(101) . Chr(110) . Chr(105)  
secret := secret . Chr(52) . Chr(103) . Chr(125)  
MsgBox, 64, System Notification, Access Granted!`n`nFlag: %secret%  
return  
GuiClose:
```

Print the flag:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> print(''.join(chr(c) for c in [98,111,114,111,67,84,70,123,65,72,75,95,49,115,95,108,73,115,43,101,110,105,52,103,125]))  
boroCTF{AHK_1s_lIs+eni4g}  
>>> exit
```

`boroCTF{AHK_1s_lIs+eni4g}`

___

## Perfectly Destructive File

**Author**: Franklin

**Category**: Rev

**Description**: Subject: Urgent - John's computer is acting up again

To whom it may concern,

John said that he downloaded a financial report for this quarter, yet now his computer has a "virus". He says that all of his files suddenly have a weird double extension. Like they all have ".boroCTF" appended onto them.

Can you help figure out what happened? Probably pirating video games again if you ask me.

Thanks, Jill

**Note** - This challenge does **NOT** contain any functional malware.

I download the challenge file:
```shell
file financial_report  
financial_report: PDF document, version 1.5

exiftool financial_report  
ExifTool Version Number         : 13.25  
File Name                       : financial_report  
Directory                       : .  
File Size                       : 1080 bytes  
File Modification Date/Time     : 2026:06:13 14:42:37-04:00  
File Access Date/Time           : 2026:06:13 14:42:36-04:00  
File Inode Change Date/Time     : 2026:06:13 14:42:39-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PDF  
File Type Extension             : pdf  
MIME Type                       : application/pdf  
PDF Version                     : 1.5  
Linearized                      : No  
Producer                        : pypdf  
Has XFA                         : No  
Page Count                      : 1
```

I dropped the file into [https://pdfcrowd.com/inspect-pdf/](https://pdfcrowd.com/inspect-pdf/) and I found some interesting objects:
```shell
<<
  /JS \n    var a = 7;\n    var b = 13;\n    var c = a * b;\n    var d = c - a;\n    var e = [a, b, c, d];\n    var f = "noise";\n    var g = "more_noise";\n    var h = e.join\("-"\);\n    var i = h.length;\n    var j = i ^ 42;\n    var k = [f, g, h, j];\n    var l = k.slice\(0, 3\).join\("|"\);\n\n    var encoded = "Ym9yb0NURnswbjFfRiFsZV9JNV9AMTFfaXRfdEFrZSR9";\n    var decoded = util.printd\("yyyy", new Date\(\)\);\n    
  /S /JavaScript
  /Type /Action
>>

<<
  /AA <<
    /U <<
      /JS app.alert\({cMsg:"Ya, I'm not making it that easy.", nIcon:3, cTitle:"boroCTF 2026"}\);
      /S /JavaScript
    >>
  >>
  /BS <<
    /S /S
    /W 1
  >>
  /F 4
  /FT /Btn
  /Ff 65536
  /MK <<
    /CA Click me for free flag!
  >>
  /P [8 0 R](https://pdfcrowd.com/inspect-pdf/#obj8-0)
  /Rect [
    140
    320
    360
    360
  ]
  /Subtype /Widget
  /T hacked_button
  /Type /Annot
>>
```

```shell
echo 'Ym9yb0NURnswbjFfRiFsZV9JNV9AMTFfaXRfdEFrZSR9' | base64 -d  
boroCTF{0n1_F!le_I5_@11_it_tAke$}
```

`boroCTF{0n1_F!le_I5_@11_it_tAke$}`
