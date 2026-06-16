---
layout: post
title:  "Misc Challenges"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /boroCTF-2026-misc/
---
* TOC
{:toc}

## AI Slop

**Author**: ForeverFlames

**Category**: Misc

**Description**: What is this AI slop??? Which AI made this!!! format: `boroCTF{ai_in_lowercase}`. WARNING ONLY 5 ATTEMPTS.

I download the challenge file:

![Alt text](/images/slop.jpg)

The watermark in the bottom right corner is from gemini.

`boroCTF{gemini}`

___

## 64 is life

**Author**: ForeverFlames

**Category**: Misc

**Description**: Truth, broken into sixty-four.

I download the challenge file:
```shell
ile 64.zip  
64.zip: Zip archive data, made by v3.0 UNIX, extract using at least v1.0, last modified May 20 2026 20:26:06, uncompressed size 0, method=store
```

Unzipping the file:
```shell
unzip 64.zip  
Archive:  64.zip  
   creating: ctf_chunks/  
 extracting: ctf_chunks/Mg==  
 extracting: ctf_chunks/MjA=  
 extracting: ctf_chunks/Mjc=  
 extracting: ctf_chunks/MjE=  
 extracting: ctf_chunks/Mjg=  
 extracting: ctf_chunks/MjI=  
 extracting: ctf_chunks/Mjk=  
 extracting: ctf_chunks/MjM=  
 extracting: ctf_chunks/MjQ=  
 extracting: ctf_chunks/MjU=  
 extracting: ctf_chunks/MjY=  
 extracting: ctf_chunks/MQ==  
 extracting: ctf_chunks/MTA=  
 extracting: ctf_chunks/MTc=  
 extracting: ctf_chunks/MTE=  
 extracting: ctf_chunks/MTg=  
 extracting: ctf_chunks/MTI=  
 extracting: ctf_chunks/MTk=  
 extracting: ctf_chunks/MTM=  
 extracting: ctf_chunks/MTQ=  
 extracting: ctf_chunks/MTU=  
 extracting: ctf_chunks/MTY=  
 extracting: ctf_chunks/Mw==  
 extracting: ctf_chunks/MzA=  
 extracting: ctf_chunks/Mzc=  
 extracting: ctf_chunks/MzE=  
 extracting: ctf_chunks/Mzg=  
 extracting: ctf_chunks/MzI=  
 extracting: ctf_chunks/Mzk=  
 extracting: ctf_chunks/MzM=  
 extracting: ctf_chunks/MzQ=  
 extracting: ctf_chunks/MzU=  
 extracting: ctf_chunks/MzY=  
 extracting: ctf_chunks/NA==  
 extracting: ctf_chunks/NDA=  
 extracting: ctf_chunks/NDc=  
 extracting: ctf_chunks/NDE=  
 extracting: ctf_chunks/NDg=  
 extracting: ctf_chunks/NDI=  
 extracting: ctf_chunks/NDk=  
 extracting: ctf_chunks/NDM=  
 extracting: ctf_chunks/NDQ=  
 extracting: ctf_chunks/NDU=  
 extracting: ctf_chunks/NDY=  
 extracting: ctf_chunks/Ng==  
 extracting: ctf_chunks/NjA=  
 extracting: ctf_chunks/NjE=  
 extracting: ctf_chunks/NjI=  
 extracting: ctf_chunks/NjM=  
 extracting: ctf_chunks/NjQ=  
 extracting: ctf_chunks/NQ==  
 extracting: ctf_chunks/NTA=  
 extracting: ctf_chunks/NTc=  
 extracting: ctf_chunks/NTE=  
 extracting: ctf_chunks/NTg=  
 extracting: ctf_chunks/NTI=  
 extracting: ctf_chunks/NTk=  
 extracting: ctf_chunks/NTM=  
 extracting: ctf_chunks/NTQ=  
 extracting: ctf_chunks/NTU=  
 extracting: ctf_chunks/NTY=  
 extracting: ctf_chunks/Nw==  
 extracting: ctf_chunks/OA==  
 extracting: ctf_chunks/OQ==
```

Each file is numbered. It is just base64 encoded:
```shell
echo 'Mg==' | base64 -d  
2
```

I run this python to concatenate the files in order:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import os  
... import base64  
...  
... chunk_dir = "ctf_chunks"   
... chunks = []
... for fname in os.listdir(chunk_dir): 
...     idx_str = base64.b64decode(fname).decode()  
...     idx = int(idx_str)  
...     chunks.append((idx, fname))  
...  
... # sort by index  
... chunks.sort(key=lambda x: x[0])  
...  
... # concatenate in order  
... with open("reconstructed.bin", "wb") as out:  
...     for _, fname in chunks:  
...         with open(os.path.join(chunk_dir, fname), "rb") as f:  
...             out.write(f.read())  
...  
... print("data saved as reconstructed.bin")  
...  
data saved as reconstructed.bin
```

Output:
```shell
cat reconstructed.bin  
Y40m40940y40b40040N40U40R40n40t40z40M40X40h40040e40V40940m40M40H40V40y40X40240I40z40Y40X40V40040e40X40040=4040404040404040404040404040404040404040404040404040404040
```

I need to remove all instances of `40` and decode from base64. Decoding:
```shell
echo 'Ym9yb0NURntzMXh0eV9mMHVyX2IzYXV0eX0=' | base64 -d  
boroCTF{s1xty_f0ur_b3auty}
```

[cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Regex','string':'40'%7D,'%20',true,false,true,false)From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=WTQwbTQwOTQweTQwYjQwMDQwTjQwVTQwUjQwbjQwdDQwejQwTTQwWDQwaDQwMDQwZTQwVjQwOTQwbTQwTTQwSDQwVjQweTQwWDQwMjQwSTQwejQwWTQwWDQwVjQwMDQwZTQwWDQwMDQwPTQwNDA0MDQwNDA0MDQwNDA0MDQwNDA0MDQwNDA0MDQwNDA0MDQwNDA0MDQwNDA0MDQwNDA0MDQwNDA)

`boroCTF{s1xty_f0ur_b3auty}`

___

## File File Crocodile

**Author**: ForeverFlames

**Category**: Misc

**Description**: We managed to snap a picture of the infamous File File Crocodile, but right before the flash went off, he swallowed a locked archive containing our flag! He's a master of disguise and his stomach acid has slightly digested the file signatures. Interrogating him didn't work as the only word he seemed to know was "croc". Can you cut him open, perform some surgery, and get our archive back?

I get the file `file_file_crocodile.png`:

![Alt text](/images/file_file_crocodile.png)

I looked at the png until I found what looked like a zip file:
```shell
xxd file_file_crocodile.png | tail  
000dca00: 3244 fcb6 0454 8f84 7a2a 76f7 79f7 7d31  2D...T..z*v.y.}1  
000dca10: 4e4e bb08 e06a cb97 365f 1d99 d56b 884c  NN...j..6_...k.L  
000dca20: 504b 0708 5f19 e053 3900 0000 2d00 0000  PK.._..S9...-...  
000dca30: 4643 0102 1e03 0a00 0900 0000 ef5e c85c  FC...........^.\  
000dca40: 5f19 e053 3900 0000 2d00 0000 0800 1800  _..S9...-.......  
000dca50: 0000 0000 0100 0000 ff81 0000 0000 666c  ..............fl  
000dca60: 6167 2e74 7874 5554 0500 03f2 e526 6a75  ag.txtUT.....&ju  
000dca70: 780b 0001 04e8 0300 0004 e803 0000 4643  x.............FC  
000dca80: 0506 0000 0000 0100 0100 4e00 0000 8b00  ..........N.....  
000dca90: 0000 0000                                ....
```

The magic bytes that identify a ZIP archive are altered. For example near the end it has `FC` instead of `PK`. I fix all instances of this:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> with open('file_file_crocodile.png', 'rb') as f:  
...     data = f.read()  
... data = data.replace(b'\x46\x43\x01\x02', b'\x50\x4b\x01\x02')  
... data = data.replace(b'\x46\x43\x03\x04', b'\x50\x4b\x03\x04')  
... data = data.replace(b'\x46\x43\x05\x06', b'\x50\x4b\x05\x06')  
... with open('fixed.png', 'wb') as f:  
...     f.write(data)  
...
```

Now I can unzip the image with the password `croc`:
```shell
unzip fixed.png  
Archive:  fixed.png  
warning [fixed.png]:  903589 extra bytes at beginning or within zipfile  
  (attempting to process anyway)  
[fixed.png] flag.txt password:  
 extracting: flag.txt  

cat flag.txt  
boroCTF{n3v3r_sm1l3_4t_4_p0lygl0t_cr0c0d1l3}
```

`boroCTF{n3v3r_sm1l3_4t_4_p0lygl0t_cr0c0d1l3}`

___

## Worthful Glory

**Author**: ForeverFlames

**Category**: Misc

**Description**: The sysadmin left behind a single photo of the Boro football field. He said the key was a field goal on game day. We need to recover the corrupted log file he hid, unlock it, and find the flag.

I download the challenge file:
```shell
file football_field.jpg  
football_field.jpg: data

xxd football_field.jpg | head  
00000000: 6767 ffe0 0010 4a46 4946 0001 0100 0001  gg....JFIF......  
00000010: 0001 0000 ffdb 0043 0003 0202 0202 0203  .......C........  
00000020: 0202 0203 0303 0304 0604 0404 0404 0806  ................  
00000030: 0605 0609 080a 0a09 0809 090a 0c0f 0c0a  ................  
00000040: 0b0e 0b09 090d 110d 0e0f 1010 1110 0a0c  ................  
00000050: 1213 1210 130f 1010 10ff db00 4301 0303  ............C...  
00000060: 0304 0304 0804 0408 100b 090b 1010 1010  ................  
00000070: 1010 1010 1010 1010 1010 1010 1010 1010  ................  
00000080: 1010 1010 1010 1010 1010 1010 1010 1010  ................  
00000090: 1010 1010 1010 1010 1010 1010 1010 ffc0  ................
```

The file is corrupt. I need to fix the file signature back to `ÿØÿà`. Change `6767 ffe0` to `ffd8 ffe0`:
```shell
xxd 3pointerbaby.jpg | head  
00000000: ffd8 ffe0 0010 4a46 4946 0001 0100 0001  ......JFIF......  
00000010: 0001 0000 ffdb 0043 0003 0202 0202 0203  .......C........  
00000020: 0202 0203 0303 0304 0604 0404 0404 0806  ................  
00000030: 0605 0609 080a 0a09 0809 090a 0c0f 0c0a  ................  
00000040: 0b0e 0b09 090d 110d 0e0f 1010 1110 0a0c  ................  
00000050: 1213 1210 130f 1010 10ff db00 4301 0303  ............C...  
00000060: 0304 0304 0804 0408 100b 090b 1010 1010  ................  
00000070: 1010 1010 1010 1010 1010 1010 1010 1010  ................  
00000080: 1010 1010 1010 1010 1010 1010 1010 1010  ................  
00000090: 1010 1010 1010 1010 1010 1010 1010 ffc0  ................
```

Now I have the fixed image:
```shell
file 3pointerbaby.jpg  
3pointerbaby.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 2392x1346, components 3

exiftool 3pointerbaby.jpg  
ExifTool Version Number         : 13.25  
File Name                       : 3pointerbaby.jpg  
Directory                       : .  
File Size                       : 1227 kB  
File Modification Date/Time     : 2026:06:13 12:33:43-04:00  
File Access Date/Time           : 2026:06:13 12:33:43-04:00  
File Inode Change Date/Time     : 2026:06:13 12:33:47-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : JPEG  
File Type Extension             : jpg  
MIME Type                       : image/jpeg  
JFIF Version                    : 1.01  
Resolution Unit                 : None  
X Resolution                    : 1  
Y Resolution                    : 1  
Image Width                     : 2392  
Image Height                    : 1346  
Encoding Process                : Baseline DCT, Huffman coding  
Bits Per Sample                 : 8  
Color Components                : 3  
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)  
Image Size                      : 2392x1346  
Megapixels                      : 3.2
```

I didnt find anything hidden with `binwalk`, `foremost`, or `zsteg`. I could see the string `3pointerbaby` in the image just above the field goal. I decide to try `steghide`:
```shell
steghide extract -sf 3pointerbaby.jpg  
Enter passphrase:  
wrote extracted data to "alexander_log.txt".
```

It worked with the password `3P0INTERBABY`.

`alexander_log.txt` is a password protected zip file:
```shell
file alexander_log.txt  
alexander_log.txt: Zip archive data, made by v3.0 UNIX, extract using at least v1.0, last modified Feb 06 2026 23:07:10, uncompressed size 39, method=store
```

I probably could have guessed this password but I just cracked it with `john`. The password is `football`:
```shell
zip2john alexander_log.txt > zip.hash  
ver 1.0 efh 5455 efh 7875 alexander_log.txt/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=51, decmplen=39, crc=B6498E76 ts=B8E5 cs=b8e5 type=0  

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
football         (alexander_log.txt/flag.txt)  
1g 0:00:00:00 DONE (2026-06-13 12:42) 14.28g/s 58514p/s 58514c/s 58514C/s 123456..bigman  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

Unzip it and get the flag:
```shell
unzip alexander_log.txt  
Archive:  alexander_log.txt  
[alexander_log.txt] flag.txt password:  
replace flag.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: y  
 extracting: flag.txt  

cat flag.txt  
boroctf{pixels_and_passwords_dont_mix}
```

`boroctf{pixels_and_passwords_dont_mix}`

___

## Nature's Delight

**Author**: ForeverFlames

**Category**: Misc

**Description**: I found this tag stuck on the back of my shirt! My friend must have put it... but who even makes these?

flag format: `boroCTF{tag_maker}`

I get the challenge file `weirdtag.jpg`:

![Alt text](/images/weirdtag.jpg)

I did a reverse image search on the tag and the google results were nothing but Poland Spring water.

`boroCTF{Poland_Spring}`
