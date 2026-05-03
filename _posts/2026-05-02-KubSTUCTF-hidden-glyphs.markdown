---
layout: post
title:  "Hidden Glyphs"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-hidden-glyphs/
---

**Author**: [@van_1pi](https://t.me/van_1pi)

**Category**: Steganography

**Description**: Our agent received an encrypted document from an informant. Visually, it's a regular PDF, but we are confident that a secret message is hidden in it. They say that _"the breadth of one's view determines the depth of understanding."_ Find the flag. Files: `stego_challenge.pdf` — a PDF document with a hidden flag. Format: `KubSTU{...}`

I get the challenge file:
```shell
file stego_challenge.pdf  
stego_challenge.pdf: PDF document, version 1.7, 1 page(s)  

exiftool stego_challenge.pdf  
ExifTool Version Number         : 13.25  
File Name                       : stego_challenge.pdf  
Directory                       : .  
File Size                       : 9.9 kB  
File Modification Date/Time     : 2026:05:01 22:32:35-04:00  
File Access Date/Time           : 2026:05:01 22:32:35-04:00  
File Inode Change Date/Time     : 2026:05:01 22:32:54-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PDF  
File Type Extension             : pdf  
MIME Type                       : application/pdf  
PDF Version                     : 1.7  
Linearized                      : No  
Page Count                      : 1  
Title                           : Classified Document - Level 5 Clearance Required  
Author                          : Security Department  
Subject                         : Top Secret Information  
Creator                         : SecureDoc Generator v3.0  
Producer                        : PDF Steganography Engine  
Create Date                     : 2026:03:25 12:00:00  
Keywords                        : classified, secret, font, encoding
```

Looking at the pdf:

![Alt text](/images/pdfscreenshot.png)

Looking at the plaintext:
```shell
pdftotext -layout stego_challenge.pdf out.txt  

cat out.txt  
SECRETDOCUMENT  
This document contains classified information.  
  
The data has been encoded using advanced techniques.  
  
Only authorized personnel may access the hidden content.  
  
  
  
  
ABCDEFGHIJKLMNOPQRSTUVWXYZ  
  
abcdefghijklmnopqrstuvwxyz  
  
0123456789  
  
  
  
Hint: The font hides more than you see...  
  
Each glyph has a width. What do they tell?
```

I decompress the PDF and look for the `/Widths` array:
```shell
qpdf --qdf stego_challenge.pdf decompressed.pdf  

grep -A5 '/Widths' decompressed.pdf  
/Widths 12 0 R  
>>  
endobj  
  
%% Original object ID: 73 0  
9 0 obj
```

Once I have each width value I divide each by 10, and get the ASCII code for each character in the flag:
```shell
python3  
Python 3.13.5 (main, Jun 25 2025, 18:55:22) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> widths = [  
...     750, 1170, 980, 830, 840, 850, 1230, 1160, 1210, 1120,  
...     510, 950, 510, 950, 1020, 480, 1100, 1160, 950, 1190,  
...     490, 1000, 1160, 1040, 530, 950, 520, 1140, 510, 950,  
...     1160, 1140, 490, 990, 1070, 1210, 1250  
... ]  
... flag = ''.join(chr(w // 10) for w in widths)  
... print(flag)  
...  
KubSTU{typ3_3_f0nt_w1dth5_4r3_tr1cky}
```

The PDF contained a custom `Type3` font where the `Widths` array stored the horizontal advance of each glyph. Each number is 10 times the ASCII code of the hidden character. 

`KubSTU{typ3_3_f0nt_w1dth5_4r3_tr1cky}`