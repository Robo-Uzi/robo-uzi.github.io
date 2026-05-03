---
layout: post
title:  "Capybara Secret"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-capybara-secret/
---

**Author**: [@van_1pi](https://t.me/van_1pi)

**Category**: Steganography

**Description**: The guardian capybara hid a message in her portrait. They say the secret is visible only to those who can look beyond the surface. Find the hidden message.

I get the challenge file:

![Alt text](/images/challenge934.jpg)

ROT13 after looking at the metadata:
```shell
exiftool challenge.jpg  
ExifTool Version Number         : 13.25  
File Name                       : challenge.jpg  
Directory                       : .  
File Size                       : 145 kB  
File Modification Date/Time     : 2026:05:01 21:55:49-04:00  
File Access Date/Time           : 2026:05:01 21:55:49-04:00  
File Inode Change Date/Time     : 2026:05:01 21:55:53-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : JPEG  
File Type Extension             : jpg  
MIME Type                       : image/jpeg  
JFIF Version                    : 1.01  
Resolution Unit                 : inches  
X Resolution                    : 144  
Y Resolution                    : 144  
Exif Byte Order                 : Little-endian (Intel, II)  
XP Comment                      : XhoFGH{J0J_1aperq1oyr_pnclon6n}  
Image Width                     : 1024  
Image Height                    : 559  
Encoding Process                : Progressive DCT, Huffman coding  
Bits Per Sample                 : 8  
Color Components                : 3  
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)  
Image Size                      : 1024x559  
Megapixels                      : 0.572
```

`KubSTU{W0W_1ncred1ble_capyba6a}`