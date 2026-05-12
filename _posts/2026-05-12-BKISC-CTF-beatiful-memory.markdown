---
layout: post
title:  "Beatiful Memory"
date:   2026-05-12 19:09:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BKISC-CTF-2026-beatiful-memory/
---

**Category**: Forensics

**Description**: I left my most precious memory here, can you find it?

I get the challenge file:
```shell
unzip chall.zip  
Archive:  chall.zip  
  inflating: chall.dmp

file chall.dmp  
chall.dmp: MS Windows 64bit crash dump, version 15.19045, 2 processors, DumpType (0x1), 1023757 pages
```
I get a `4.3 GB Windows crash dump`. 

`strings` and `grep` reveals 2 fake flags:
```shell
strings chall.dmp | grep "BKISC"  
BKISC{Dunno_whut_to_say_T^T_Whut_r_u_doing_here?}  
BKISC{Woah_woah_u_know_sst1?}
```

`strings -e l` and `grep` finds more fake flags and the real one:
```shell
strings -e l chall.dmp | grep "BKISC{"  
BKISC{flag_is_here}  
BKISC{flag_is_here}m  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}r  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{W3ll_M3mory_is_Str0nk_right_?}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}v  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
BKISC{flag_is_here}  
}BKISC{flag_
```

I did try using `vol.py` (Volatility) but the other way is a lot easier:
```shell
python3 vol.py -f chall.dmp windows.info  
Volatility 3 Framework 2.28.0  
Progress:  100.00               PDB scanning finished  
Variable        Value  
  
Kernel Base     0xf80528000000  
DTB     0x1ad000  
Symbols file:///home/user/Desktop/ctf/venv/lib/python3.13/site-packages/volatility3/symbols/windows/ntkrnlmp.pdb/F57E740B088E5056E8AF0772F1CC5BEB-1.json.xz  
Is64Bit True  
IsPAE   False  
layer_name      0 WindowsIntel32e  
memory_layer    1 WindowsCrashDump64Layer  
base_layer      2 FileLayer  
KdVersionBlock  0xf80528c0f400  
Major/Minor     15.19041  
MachineType     34404  
KeNumberProcessors      2  
SystemTime      2026-05-06 08:14:34+00:00  
NtSystemRoot    C:\WINDOWS  
NtProductType   NtProductWinNt  
NtMajorVersion  10  
NtMinorVersion  0  
PE MajorOperatingSystemVersion  10  
PE MinorOperatingSystemVersion  0  
PE Machine      34404  
PE TimeDateStamp        Sat Feb  2 23:04:03 1985
```

`BKISC{W3ll_M3mory_is_Str0nk_right_?}`
