---
layout: post
title:  "Spoiled Cheese Pull"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-spoiled-cheese-pull/
---

**Title**: Spoiled Cheese Pull

**Author**: ASmilyBun

**Category**: Forensics

**Description**: My cheese got pulled and now I can't eat it. Can you help me find out who did it?

I get the challenge file:
```shell
file chall.png  
chall.png: JPEG image data, JFIF standard 13.73, density 17748x0, segment length 16, thumbnail 3x42
```

It has `.png` but says its a jpeg. I look at the file with `xxd`:
```shell
xxd chall.png | head  
00000000: ffd8 ffe0 0010 4a46 4946 000d 4948 4554  ......JFIF..IHET  
00000010: 0000 032a 0000 006e 0806 0000 00c8 e8e7  ...*...n........  
00000020: 2000 0004 0949 5341 4478 9ced dd31 6ee4   ....ISADx...1n.  
00000030: 3010 0041 e9e0 ff7f 59fe c005 0496 c6f4  0..A....Y.......  
00000040: 5255 8933 7945 535c 3714 ccfd 3ccf 735d  RU.3yES\7...<.s]  
00000050: d77d 5d97 9fd6 c13e f01c 3807 9c03 ce01  .}]....>..8.....  
00000060: e780 73c0 39e0 1c70 0e3c 85e7 e0df 0500  ..s.9..p.<......  
00000070: 0010 2354 0000 801c a102 0000 e408 1500  ..#T............  
00000080: 0020 47a8 0000 0039 4205 0000 c811 2a00  . G....9B.....*.  
00000090: 0040 8e50 0100 0072 840a 0000 9023 5400  .@.P...r.....#T.  

xxd chall.png | tail  
000003d0: 112a 0000 408e 5001 0000 7284 0a00 0090  .*..@.P...r.....  
000003e0: 2354 0000 801c a102 0000 e408 1500 0020  #T.............  
000003f0: 47a8 0000 0039 4205 0000 c811 2a00 0040  G....9B.....*..@  
00000400: 8e50 0100 0072 840a 0000 9023 5400 0080  .P...r.....#T...  
00000410: 1ca1 0200 00e4 0815 0000 2047 a800 0000  .......... G....  
00000420: 3942 0500 00c8 112a 0000 c055 f30b 6b35  9B.....*...U..k5  
00000430: 7edd d015 3dd6 0000 0000 5345 4e44 ae42  ~...=.....SEND.B  
00000440: 6082 4e6f 7468 696e 6732 5365 6548 6572  `.Nothing2SeeHer  
00000450: 6547 6f4c 6f6f 6b53 6f6d 6577 6865 7265  eGoLookSomewhere  
00000460: 456c 7365                                Else
```

The png headers need fixed. I run this python:
```python
with open('chall.png', 'rb') as f:  
    data = f.read()  
 
# PNG signature
png = b'\x89PNG\r\n\x1a\n'  
 
png += b'\x00\x00\x00\x0d'  # correct chunk length  
png += b'IHDR'              # correct chunk type  
png += data[0x10:0x1d]      # 13 bytes: width, height, color, etc.  
png += data[0x1d:0x21]      # original CRC
 
rest = data[0x21:0x442]  
rest = rest.replace(b'ISAD', b'IDAT') # fix IDAT type  
rest = rest.replace(b'SEND', b'IEND') # fix IEND type  
png += rest  
 
with open('recovered.png', 'wb') as f:  
    f.write(png)  
 
print("Saved recovered.png")
```

I get `recovered.png`:

![Alt text](/images/long-ah-qr-code.png)

Upload it to [https://qrscanner.net/](https://qrscanner.net/) and get the flag!

`dalCTF{WhY_$O_L0N5}`