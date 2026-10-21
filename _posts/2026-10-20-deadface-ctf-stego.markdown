---
layout: post
title:  "Steganography Challenges"
date:   2026-10-20 20:54:00 -0400
author: uzi
tags: [CTF]
permalink: /deadface-ctf-2026-stego/
---
* TOC
{:toc}

## Headshot

**Category**: Steganography

**Created By**: `@syyntax`

**Description**: DEADFACE was able to compromise a fileshare that consisted of corporate headshots for Aurora Health employees. We believe they're tagging the photos with information that communicates to their group which employees to target in their future campaigns. Figure out which image DEADFACE tampered with - it will give us a heads up on who they plan to target. Submit the answer as `deadface{flag_text}`.

```shell
unzip headshot.zip  
Archive:  headshot.zip  
  inflating: headshot001.png  
  inflating: headshot002.png  
  inflating: headshot003.png  
  inflating: headshot003.png_original
```

```shell
strings *.png | grep "deadface{"  
deadface{b00m_h34Dsh0TTT}
```

`deadface{b00m_h34Dsh0TTT}`

___

## File Dump 01

**Category**: Steganography

**Created By**: `@RP-01?`

**Description**: We have intercepted a DEADFACE hacker's laptop and determined he had stolen a man named Daniel's entire identity. For this first challenge, let's identify which DEADFACE member is responsible!

Submit the answer as `deadface{deadface alias}`

I get the challenge files:
```shell
unzip File_Dump_Challenge.zip  
Archive:  File_Dump_Challenge.zip  
creating: File Dump Challenge/  
  inflating: File Dump Challenge/advertising_ad_1.png  
  inflating: File Dump Challenge/b649c32d-84c5-4775-9f82-34f0dcd846a6.png  
  inflating: File Dump Challenge/bad_furniture_ad.pdf  
  inflating: File Dump Challenge/budget_notes.docx  
  inflating: File Dump Challenge/businesscard_1.png  
  inflating: File Dump Challenge/chapter_one.docx  
  inflating: File Dump Challenge/cover.png  
  inflating: File Dump Challenge/draft_strategy.docx  
  inflating: File Dump Challenge/grandmas_soup.docx  
  inflating: File Dump Challenge/grocery_list.docx  
  inflating: File Dump Challenge/hoa_complaint.docx  
  inflating: File Dump Challenge/invoice.xlsx  
  inflating: File Dump Challenge/meeting_notes.docx  
  inflating: File Dump Challenge/mileage.xlsx  
  inflating: File Dump Challenge/new_year_resolutions.docx  
  inflating: File Dump Challenge/notes.docx  
  inflating: File Dump Challenge/office_memory_meme.jpg  
  inflating: File Dump Challenge/profile_pic.png  
  inflating: File Dump Challenge/reciept_1.png  
  inflating: File Dump Challenge/reminders.docx  
  inflating: File Dump Challenge/resume.docx  
  inflating: File Dump Challenge/roommate_note.docx  
  inflating: File Dump Challenge/snip.png  
  inflating: File Dump Challenge/technical_details.docx  
  inflating: File Dump Challenge/warranty.pdf  
  inflating: File Dump Challenge/wifi_sign.docx
```

`notes.docx` held the answer:
```shell
exiftool notes.docx  
ExifTool Version Number         : 13.25  
File Name                       : notes.docx  
Directory                       : .  
File Size                       : 8.7 kB  
File Modification Date/Time     : 2026:08:22 17:36:34-04:00  
File Access Date/Time           : 2026:08:22 21:02:07-04:00  
File Inode Change Date/Time     : 2026:10:18 16:08:22-04:00  
File Permissions                : -rw-rw-rw-  
File Type                       : DOCX  
File Type Extension             : docx  
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document  
Zip Required Version            : 10  
Zip Bit Flag                    : 0  
Zip Compression                 : None  
Zip Modify Date                 : 2026:08:22 21:33:04  
Zip CRC                         : 0x00000000  
Zip Compressed Size             : 0  
Zip Uncompressed Size           : 0  
Zip File Name                   : word/  
Title                           : notes  
Creator                         : ghostwriter404  
Last Modified By                : GHOST404-PC  
Revision Number                 : 1  
Create Date                     : 2026:08:22 21:33:04.269Z  
Modify Date                     : 2026:08:22 21:33:04.269Z
```

`deadface{gh0st404}`

___

## Exfiltration

**Category**: Steganography

**Created By**: `@G2Gh0st`

**Description**: DEADFACE frequently uses unconventional methods to exchange information and avoid detection. Analysts recovered a set of images from a compromised communications channel that appear unrelated at first glance. We suspect the collection contains a hidden message intended for other members of the group. Recover the message and identify what DEADFACE is trying to say.

The challenge files are 20 images:
```shell
ls  
exfiltration.zip  image_02.jpg  image_04.jpg  image_06.jpg  image_08.jpg  image_10.jpg  image_12.jpg  image_14.jpg  image_16.jpg  image_18.jpg  image_20.jpg  image_01.jpg  image_03.jpg  image_05.jpg  image_07.jpg  image_09.jpg  image_11.jpg  image_13.jpg  image_15.jpg  image_17.jpg  image_19.jpg
```

I looked at the metadata:
```shell
exiftool image_01.jpg  
ExifTool Version Number         : 13.25  
File Name                       : image_01.jpg  
Directory                       : .  
File Size                       : 29 kB  
File Modification Date/Time     : 2026:06:02 16:55:13-04:00  
File Access Date/Time           : 2026:06:02 17:07:34-04:00  
File Inode Change Date/Time     : 2026:10:18 17:05:48-04:00  
File Permissions                : -rwx------  
File Type                       : JPEG  
File Type Extension             : jpg  
MIME Type                       : image/jpeg  
JFIF Version                    : 1.01  
Resolution Unit                 : None  
X Resolution                    : 72  
Y Resolution                    : 72  
Exif Byte Order                 : Big-endian (Motorola, MM)  
Date/Time Original              : 2026:06:01 14:13:09  
Color Space                     : sRGB  
Exif Image Width                : 307  
Exif Image Height               : 256  
Image Unique ID                 : IMG_6365  
Current IPTC Digest             : d41d8cd98f00b204e9800998ecf8427e  
IPTC Digest                     : d41d8cd98f00b204e9800998ecf8427e  
Image Width                     : 307  
Image Height                    : 256  
Encoding Process                : Baseline DCT, Huffman coding  
Bits Per Sample                 : 8  
Color Components                : 3  
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)  
Image Size                      : 307x256  
Megapixels                      : 0.079
```
`Image Unique ID` is a unique field.

Sorting by time, concatenating, and decoded from hex gives me the flag:
```shell
for f in image_*.jpg; do  
  ts=$(exiftool -s3 -DateTimeOriginal "$f")  
  id=$(exiftool -s3 -ImageUniqueID "$f")  
  hex=${id#IMG_}  
  echo "$ts $f $hex"  
done | sort  
2026:06:01 14:02:17 image_07.jpg 6465  
2026:06:01 14:04:52 image_14.jpg 61  
2026:06:01 14:08:11 image_03.jpg 6466  
2026:06:01 14:11:36 image_19.jpg 61  
2026:06:01 14:13:09 image_01.jpg 6365  
2026:06:01 14:18:44 image_12.jpg 7b  
2026:06:01 14:21:03 image_08.jpg 7469  
2026:06:01 14:24:58 image_20.jpg 6d  
2026:06:01 14:29:12 image_05.jpg 696e  
2026:06:01 14:31:47 image_16.jpg 67  
2026:06:01 14:35:20 image_02.jpg 2d69  
2026:06:01 14:39:06 image_18.jpg 73  
2026:06:01 14:42:55 image_09.jpg 2d65  
2026:06:01 14:46:13 image_13.jpg 76  
2026:06:01 14:51:38 image_04.jpg 6572  
2026:06:01 14:55:04 image_15.jpg 79  
2026:06:01 14:59:27 image_10.jpg 7468  
2026:06:01 15:03:10 image_06.jpg 69  
2026:06:01 15:07:42 image_17.jpg 6e67  
2026:06:01 15:12:29 image_11.jpg 7d  

echo '64656164666163657b74696d696e672d69732d65766572797468696e677d' | xxd -r -p  
deadface{timing-is-everything}
```

`deadface{timing-is-everything}`
