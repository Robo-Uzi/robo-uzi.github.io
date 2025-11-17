---
layout: post
title:  "Operation: Lost Data?"
date:   2025-11-16 22:39:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /platypwn-operation-lost-data/
---

**Title**: Operation: Lost Data?

**Category**: Forensics

**Description**: Briefing: Agent: Agent P (a.k.a. Perry the Platypus) Subject: Recovered backup — suspected to belong to Dr. Heinz Doofenshmirtz Objective: Extract and analyze critical intelligence fragments before they self-destruct (metaphorically, we hope).

During a recent OWCA operation, Agent P recovered a backup from a damaged device belonging to Dr. Heinz Doofenshmirtz. The phone itself is beyond repair, but traces of data remain. Analysis suggests Doofenshmirtz intentionally scattered three hidden fragments across different parts of his system. His final voice memo offers only riddles — a frozen memory, a restless idea, and in resting thoughts. Your task: extract all fragments before whatever he was planning comes to life.

I get an iPhone backup:
```shell
ls  
00  08  10  18  20  28  30  38  40  48  50  58  60  68  70  78  80  88  90  98  a0  a8  b0  b8  c0  c8  d0  d8  
e0  e8  f0  f8  01  09  11  19  21  29  31  39  41  49  51  59  61  69  71  79  81  89  91  99  a1  a9  b1  b9  
c1  c9  d1  d9  e1  e9  f1  f9  02  0a  12  1a  22  2a  32  3a  42  4a  52  5a  62  6a  72  7a  82  8a  92  9a  
a2  aa  b2  ba  c2  ca  d2  da  e2  ea  f2  fa  03  0b  13  1b  23  2b  33  3b  43  4b  53  5b  63  6b  73  7b  
83  8b  93  9b  a3  ab  b3  bb  c3  cb  d3  db  e3  eb  f3  fb  04  0c  14  1c  24  2c  34  3c  44  4c  54  5c 
64  6c  74  7c  84  8c  94  9c  a4  ac  b4  bc  c4  cc  d4  dc  e4  ec  f4  fc  05  0d  15  1d  25  2d  35  3d  
45  4d  55  5d  65  6d  75  7d  85  8d  95  9d  a5  ad  b5  bd  c5  cd  d5  dd  e5  ed  f5  fd  06  0e  16  1e  
26  2e  36  3e  46  4e  56  5e  66  6e  76  7e  86  8e  96  9e  a6  ae  b6  be  c6  ce  d6  de  e6  ee  f6  fe  
07  0f  17  1f  27  2f  37  3f  47  4f  57  5f  67  6f  77  7f  87  8f  97  9f  a7  af  b7  bf  c7  cf  d7  df  
e7  ef  f7  ff  Info.plist  Status.plist  Manifest.plist  Manifest.db 
```

I read about `.plist` files here: [https://en.wikipedia.org/wiki/Property_list](https://en.wikipedia.org/wiki/Property_list) 
```shell
Property list files are files that store serialized objects. Property list files are often used to store 
a users settings. They are also used to store information about bundle and applications, a task served by 
the resource fork in the old Mac OS.
```

At first I tried using the python package [plistlib](https://docs.python.org/3/library/plistlib.html) to parse the files. There are so many of them that it was hard to do. Then I found [iLEAPP](https://github.com/abrignoni/iLEAPP) which can parse the files and has some good features. 

I ran `python3 ileapp.py -t itunes -i /home/user/Desktop/ctf/platypwnCTF/forensics-lostdata/backup -o ileapp_output`. 

When looking through the output files I find the image `ileapp_output/iLEAPP_Reports_2025-11-15_Saturday_142900/data/private/var/mobile/Media/DCIM/100APPLE/IMG_0002.HEIC`:

![Alt text](/images/Screenshot_backup.png)

For the second flag I navigate to:
```shell
ileapp_output/iLEAPP_Reports_2025-11-15_Saturday_142900/data/private/var/mobile/Containers/Shared/AppGroup/group.com.apple.notes
```

I opened the file `NoteStore.sqlite` with DB browser for SQLite and went to the table `ZICCLOUDSYNCINGOBJECT`. I found a deleted note with the string `:f)LL1Mejh1Lsj10Q]*l?SldZ?Z%ZC1NHi80Q&[p`. 

Decode it from base85 on cyberchef ([https://gchq.github.io/CyberChef/#recipe=From_Base85('!-u',true,'z')&input=OmYpTEwxTWVqaDFMc2oxMFFdKmw/U2xkWj9aJVpDMU5IaTgwUSZbcA](https://gchq.github.io/CyberChef/#recipe=From_Base85('!-u',true,'z')&input=OmYpTEwxTWVqaDFMc2oxMFFdKmw/U2xkWj9aJVpDMU5IaTgwUSZbcA)).

There is 1 more flag here. There is a lot of information about the phone (obviously) but I cant find the last one in the backup. 

`PP{b4ckup_l0c4l_ph0t0s_f0und}`

`PP{d3l3t3d_n0t3s_4r3_n3v3r_g0n3}`