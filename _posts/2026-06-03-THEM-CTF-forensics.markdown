---
layout: post
title:  "Forensics Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THEM-CTF-2026-part2-forensics/
---
* TOC
{:toc}

## HexDumb

**Category**: Forensics

**Author**: F3L0N :)

**Description**: me only DUMB or y'all too? It's an ole screenshot tryin to tell something but I'm unable to figure it out... can you?

I get the challenge file `hexdumb.png`:

![Alt text](/images/hexdumb.png)

You could put the image into an image to text converter or give it to an AI and it will type it out. This is the hexdump:
```shell
00000000: 50 4b 03 04 0a 00 09 00 00 00 01 75 98 5c 0f de
00000010: 90 6f 20 00 00 00 14 00 00 00 08 00 1c 00 66 6c
00000020: 61 67 2e 74 78 74 55 54 09 00 03 02 b9 eb 69 02
00000030: b9 eb 69 75 78 0b 00 01 04 e8 03 00 00 04 e8 03
00000040: 00 00 5d 81 87 1d 8c 4b 2f 2a 4d af f2 f0 3a 1b
00000050: 95 84 f3 b7 a8 c9 be 77 cf 1d 92 4a de 9d eb e9
00000060: 95 c3 50 4b 07 08 0f de 90 6f 20 00 00 00 14 00
00000070: 00 00 50 4b 01 02 1e 03 0a 00 09 00 00 00 01 75
00000080: 98 5c 0f de 90 6f 20 00 00 00 14 00 00 00 08 00
00000090: 18 00 00 00 00 00 01 00 00 00 b4 81 00 00 00 00
000000a0: 66 6c 61 67 2e 74 78 74 55 54 05 00 03 02 b9 eb
000000b0: 69 75 78 0b 00 01 04 e8 03 00 00 04 e8 03 00 00
000000c0: 50 4b 05 06 00 00 00 00 01 00 01 00 4e 00 00 00
000000d0: 72 00 00 00 00 00 
```

I put the file into cyberchef and do `from hexdump`. I get a password protected zip file:
```shell
file hexdump-image.zip  
hexdump-image.zip: Zip archive data, made by v3.0 UNIX, extract using at least v1.0, last modified Apr 24 2026 14:40:02, uncompressed size 20, method=store
```

[cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=From_Hexdump()&input=MDAwMDAwMDA6IDUwIDRiIDAzIDA0IDBhIDAwIDA5IDAwIDAwIDAwIDAxIDc1IDk4IDVjIDBmIGRlCjAwMDAwMDEwOiA5MCA2ZiAyMCAwMCAwMCAwMCAxNCAwMCAwMCAwMCAwOCAwMCAxYyAwMCA2NiA2YwowMDAwMDAyMDogNjEgNjcgMmUgNzQgNzggNzQgNTUgNTQgMDkgMDAgMDMgMDIgYjkgZWIgNjkgMDIKMDAwMDAwMzA6IGI5IGViIDY5IDc1IDc4IDBiIDAwIDAxIDA0IGU4IDAzIDAwIDAwIDA0IGU4IDAzCjAwMDAwMDQwOiAwMCAwMCA1ZCA4MSA4NyAxZCA4YyA0YiAyZiAyYSA0ZCBhZiBmMiBmMCAzYSAxYgowMDAwMDA1MDogOTUgODQgZjMgYjcgYTggYzkgYmUgNzcgY2YgMWQgOTIgNGEgZGUgOWQgZWIgZTkKMDAwMDAwNjA6IDk1IGMzIDUwIDRiIDA3IDA4IDBmIGRlIDkwIDZmIDIwIDAwIDAwIDAwIDE0IDAwCjAwMDAwMDcwOiAwMCAwMCA1MCA0YiAwMSAwMiAxZSAwMyAwYSAwMCAwOSAwMCAwMCAwMCAwMSA3NQowMDAwMDA4MDogOTggNWMgMGYgZGUgOTAgNmYgMjAgMDAgMDAgMDAgMTQgMDAgMDAgMDAgMDggMDAKMDAwMDAwOTA6IDE4IDAwIDAwIDAwIDAwIDAwIDAxIDAwIDAwIDAwIGI0IDgxIDAwIDAwIDAwIDAwCjAwMDAwMGEwOiA2NiA2YyA2MSA2NyAyZSA3NCA3OCA3NCA1NSA1NCAwNSAwMCAwMyAwMiBiOSBlYgowMDAwMDBiMDogNjkgNzUgNzggMGIgMDAgMDEgMDQgZTggMDMgMDAgMDAgMDQgZTggMDMgMDAgMDAKMDAwMDAwYzA6IDUwIDRiIDA1IDA2IDAwIDAwIDAwIDAwIDAxIDAwIDAxIDAwIDRlIDAwIDAwIDAwCjAwMDAwMGQwOiA3MiAwMCAwMCAwMCAwMCAwMCA)

Crack the password with `zip2john` and `john` to get the flag:
```shell
zip2john hexdump-image.zip > zip.hash  
ver 1.0 efh 5455 efh 7875 hexdump-image.zip/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=32, decmplen=20, crc=6F90DE0F ts=7501 cs=7501 type=0  

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
love@123         (hexdump-image.zip/flag.txt)  
1g 0:00:00:00 DONE (2026-05-31 02:26) 2.000g/s 696320p/s 696320c/s 696320C/s martakis..kingstreet  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.  

unzip hexdump-image.zip  
Archive:  hexdump-image.zip  
[hexdump-image.zip] flag.txt password:  
 extracting: flag.txt  

cat flag.txt  
THEM?!CTF{XXD_0R_XD}
```

`THEM?!CTF{XXD_0R_XD}`

___

## Confidential - I

**Category**: Forensics

**Author**: F3L0N :)

**Description**: During an analysis, it was found that some documents were recovered from an unknown device. We suspect there was something suspicious, even tho it looks like an official gov document but can you find something more?

Flag format: THEM?!CTF{...}

I download the challenge file:
```shell
file CONFIDENTIAL.pdf  
CONFIDENTIAL.pdf: PDF document, version 1.4, 3 page(s)
```

Sections of the pdf appear redacted:

![Alt text](/images/fake-sketchy-pdf.png)

You could use `ctrl+a` to select all the text in the pdf, then copy it into a text file. Or use `pdftotext`:
```shell
pdftotext CONFIDENTIAL.pdf

cat CONFIDENTIAL.txt | grep THEM  
THEM?!CTF{N0T_3V3RYTH1NG_TH4T_1SNT_V1S1BL3_1S_N0N3X1S71NG}  
IDENT_STRING: THEM?!CTF{R3TR1V3D_SUCC3SSFULLY}  
FLAG1=THEM?!CTF{N0T_3V3RYTH1NG_TH4T_1SNT_V1S1BL3_1S_N0N3X1S71NG}
```


`THEM?!CTF{N0T_3V3RYTH1NG_TH4T_1SNT_V1S1BL3_1S_N0N3X1S71NG}`

___

## Confidential - II

**Category**: Forensics

**Author**: F3L0N :)

**Description**: Description: During an analysis, it was found that some documents were recovered from an unknown device. We suspect there was something suspicious, something was redacted, can you recover that? Note: Use same file present in CONFIDENTIAL - I 
Flag format: `THEM?!CTF{...}`

I uploaded the pdf to [https://pdfcrowd.com/inspect-pdf/](https://pdfcrowd.com/inspect-pdf/) and found the second flag in the raw PDF content stream:
```shell
BT /F2 7 Tf 8.4 TL ET
BT 1 0 0 1 76 630 Tm (RECOVERED IDENTIFIER \227 PARTIAL REDACTION APPLIED \227 REF: NSA-SCS-OPSEC-12-04) Tj T* ET
0 0 0 rg
BT /F5 9 Tf 10.8 TL ET
BT 1 0 0 1 80 616 Tm (IDENT_STRING: ) Tj T* ET
BT /F5 9 Tf 10.8 TL ET
0 0 0 rg
BT 1 0 0 1 155.6 616 Tm (THEM?!CTF{R3TR1V3D_SUCC3SSFULLY}) Tj T* ET
0 0 0 rg
n 155.6 610 172.8 14 re f*
1 1 1 rg
BT /F1 1 Tf 1.2 TL ET
```

It was also in the output from `cat CONFIDENTIAL.txt | grep THEM`.

`THEM?!CTF{R3TR1V3D_SUCC3SSFULLY}`
