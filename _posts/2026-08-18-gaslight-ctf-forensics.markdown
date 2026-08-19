---
layout: post
title:  "Forensics Challenges"
date:   2026-08-18 21:37:00 -0400
author: uzi
tags: [CTF]
permalink: /gaslight-ctf-2026-forensics/
---
* TOC
{:toc}

## good-luck! 

**Category**: forensics

**Author**: riyc

**Description**: Good luck!

I get the challenge file:
```shell
file goodluck2.mp3  
goodluck2.mp3: Audio file with ID3 version 2.3.0, contains: MPEG ADTS, layer III, v1, 320 kbps, 44.1 kHz, JntStereo  

exiftool goodluck2.mp3  
ExifTool Version Number         : 13.25  
File Name                       : goodluck2.mp3  
Directory                       : .  
File Size                       : 299 kB  
File Modification Date/Time     : 2026:08:14 19:54:45-04:00  
File Access Date/Time           : 2026:08:14 19:54:45-04:00  
File Inode Change Date/Time     : 2026:08:14 19:54:48-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : MP3  
File Type Extension             : mp3  
MIME Type                       : audio/mpeg  
MPEG Audio Version              : 1  
Audio Layer                     : 3  
Audio Bitrate                   : 320 kbps  
Sample Rate                     : 44100  
Channel Mode                    : Joint Stereo  
MS Stereo                       : On  
Intensity Stereo                : Off  
Copyright Flag                  : False  
Original Media                  : True  
Emphasis                        : None  
Encoder                         : LAME3.100.�4  
ID3 Size                        : 37  
Txxx                            : {"data":{}}  
Duration                        : 7.47 s (approx)
```

I opened the file in audacity and looked at the spectrogram view:

![Alt text](/images/gaslightctf2026forensics1.png)

`gaslightCTF{c4n_u_s33_me?_u4ya}`

___

## blackout

**Category**: forensics

**Author**: riyc

**Description**: i was doing my work, then my power went off! my computer spat this out afterwards, can you recover the flag?

I get the challenge file:
```shell
pdfinfo recovered_file  
Title:           recovered_file  
Producer:        Skia/PDF m153 Google Docs Renderer  
Custom Metadata: no  
Metadata Stream: no  
Tagged:          yes  
UserProperties:  no  
Suspects:        no  
Form:            none  
JavaScript:      no  
Pages:           8  
Encrypted:       no  
Page size:       612 x 792 pts (letter)  
Page rot:        0  
File size:       105969 bytes  
Optimized:       no  
PDF version:     1.4
```

When I open the pdf it appears completely redacted. There is a lot of text but it is all blocked out. I just used `pdftotext` and `grep` to get the flag:
```shell
pdftotext recovered_file

cat recovered_file.txt | wc -l  
351

cat recovered_file.txt | grep "gaslightCTF{"
cowabunga gaslightCTF{c0w4bung4_f1le_4ev3r} tongue yaAbcd i eat cheese and i like to do
```
The rest of the file was nonsense.

`gaslightCTF{c0w4bung4_f1le_4ev3r}`

___

## icon-sketch

**Category**: forensics

**Author**: riyc

**Description**: the pee people found the first version of the gaslightCTF icon... apparently theres a flag in here?

I get the challenge file:
```shell
exiftool icon.png  
ExifTool Version Number         : 13.25  
File Name                       : icon.png  
Directory                       : .  
File Size                       : 43 kB  
File Modification Date/Time     : 2026:08:14 13:20:23-04:00  
File Access Date/Time           : 2026:08:14 13:23:40-04:00  
File Inode Change Date/Time     : 2026:08:15 23:59:30-04:00  
File Permissions                : -rw-rw-rw-  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 2000  
Image Height                    : 800  
Bit Depth                       : 8  
Color Type                      : RGB with Alpha  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Profile Name                    : kCGColorSpaceDisplayP3  
Profile CMM Type                : Apple Computer Inc.  
Profile Version                 : 4.0.0  
Profile Class                   : Display Device Profile  
Color Space Data                : RGB  
Profile Connection Space        : XYZ  
Profile Date Time               : 2022:01:01 00:00:00  
Profile File Signature          : acsp  
Primary Platform                : Apple Computer Inc.  
CMM Flags                       : Not Embedded, Independent  
Device Manufacturer             : Apple Computer Inc.  
Device Model                    :  
Device Attributes               : Reflective, Glossy, Positive, Color  
Rendering Intent                : Perceptual  
Connection Space Illuminant     : 0.9642 1 0.82491  
Profile Creator                 : Apple Computer Inc.  
Profile ID                      : 0  
Profile Description             : Display P3  
Profile Copyright               : Copyright Apple Inc., 2022  
Media White Point               : 0.96419 1 0.82489  
Red Matrix Column               : 0.51512 0.2412 -0.00105  
Green Matrix Column             : 0.29198 0.69225 0.04189  
Blue Matrix Column              : 0.1571 0.06657 0.78407  
Red Tone Reproduction Curve     : (Binary data 32 bytes, use -b option to extract)  
Chromatic Adaptation            : 1.04788 0.02292 -0.0502 0.02959 0.99048 -0.01706 -0.00923 0.01508 0.75168  
Blue Tone Reproduction Curve    : (Binary data 32 bytes, use -b option to extract)  
Green Tone Reproduction Curve   : (Binary data 32 bytes, use -b option to extract)  
Color Primaries                 : SMPTE EG 432-1  
Transfer Characteristics        : sRGB or sYCC  
Matrix Coefficients             : Identity matrix  
Video Full Range Flag           : 1  
Exif Byte Order                 : Big-endian (Motorola, MM)  
Photometric Interpretation      : RGB  
Document Name                   : dGhlIHBlZSBwZW9wbGUgc2FpZCB0byBkZWNvZGUgYW5kIHB1dCB0aGUgdGl0bGVzIHRvZ2V0aGVy  
X Resolution                    : 300  
Y Resolution                    : 300  
Resolution Unit                 : inches  
Exif Image Width                : 2000  
Exif Image Height               : 800  
Pixels Per Unit X               : 11811  
Pixels Per Unit Y               : 11811  
Pixel Units                     : meters  
XMP Toolkit                     : Image::ExifTool 12.57  
Artwork Title                   : MmUgMmUgMmUgMmUgMmUgMmUgMmUgMmUgNDMgNTQgNDYgMmUgMmUgMmUgNWYgMmUgNjggMzQgMmUgNWYgMmUgMmUgNzAgNzAgMzAgNzMgMzMgMmUgMmUgMzIgNjIgNWYgMmUgMzEgNzMgM  
mUgMmUgN2Q=  
Title                           : dHpob3J0c2cuLi57cjUuZy4uZy5oZi4uLi4ud18uLi5rLi5oPy4=  
Image Size                      : 2000x800  
Megapixels                      : 1.6
```

The first part of the flag needs decoded from base64 then from atbash cipher. The second part of the flag needs decoded from base64 then from hex. Then combine them.

Python solve:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import base64  
...  
... b64_1 = "MmUgMmUgMmUgMmUgMmUgMmUgMmUgMmUgNDMgNTQgNDYgMmUgMmUgMmUgNWYgMmUgNjggMzQgMmUgNWYgMmUgMmUgNzAgNzAgMzAgNzMgMzMgMmUgMmUgMzIgNjIgNWYgMmUgMzEgNzMgMmUgMmUgN2Q="  
... b64_2 = "dHpob3J0c2cuLi57cjUuZy4uZy5oZi4uLi4ud18uLi5rLi5oPy4="  
...  
... def atbash(s):  
...     return ''.join(  
...         chr(ord('z') - (ord(c) - ord('a'))) if 'a' <= c <= 'z' else  
...         chr(ord('Z') - (ord(c) - ord('A'))) if 'A' <= c <= 'Z' else c  
...         for c in s  
...     )  
...  
... hex_str = base64.b64decode(b64_1).decode()  
... part1 = bytes.fromhex(hex_str.replace(' ', '')).decode()  
... part2 = atbash(base64.b64decode(b64_2).decode())  
...  
... flag = ''.join(part1[i] if part1[i] != '.' else part2[i] for i in range(len(part1)))  
...  
... print(flag)  
...  
gaslightCTF{i5_th4t_supp0s3d_2b_p1ss?}
```

`gaslightCTF{i5_th4t_supp0s3d_2b_p1ss?}`

___

## layered-pages

**Category**: forensics

**Author**: riyc

**Description**: my desk is a mess! help me sort it out!

I get the challenge file:
```shell
file 123456789.jpg  
123456789.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 300x300, segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=6, PhotometricInterpretation=RGB], baseline, precision 8, 250x250, components 3
```

I ran `foremost` and got 6 jpgs and 3 pngs:
```shell
foremost 123456789.jpg  
Processing: 123456789.jpg  
|*|

ls output/jpg/  
00000000.jpg  00000042.jpg  00000076.jpg  00000112.jpg  00000190.jpg  00000254.jpg  

ls output/png/  
00000147.png  00000219.png  00000297.png
```

The images contained the flag. I just had to look at them and put it together in order.

`gaslightCTF{c4rv3_1t_0ut}`
