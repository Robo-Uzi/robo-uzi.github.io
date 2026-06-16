---
layout: post
title:  "Forensics Challenges"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /boroCTF-2026-forensics/
---
* TOC
{:toc}

## kitty kitty meow meow

**Author**: Solarity

**Category**: Forensics

**Description**: aww... so cute!

I get the challenge file:

![Alt text](/images/meow.jpg)

```shell
strings meow.jpg | grep boro  
boroCTF{f0r3nsic_@nalysis#}

# or

xxd meow.jpg | tail  
0002a280: a96e 9ca9 c653 cb18 e942 083d daf5 fcfa  .n...S...B.=....  
0002a290: fbd4 2d38 63e1 539e b511 3af9 9c7a f3ae  ..-8c.S...:..z..  
0002a2a0: 0482 3393 8a98 5216 6392 c739 1ceb b38c  ..3...R.c..9....  
0002a2b0: 6fb7 5c57 3100 6dbd 349d f9e7 a7b5 5438  o.\W1.m.4.....T8  
0002a2c0: 118d b047 a8a5 0e3a febc a99a b6f4 f2a4  ...G...:........  
0002a2d0: e668 2612 2a82 33cb 9629 0498 1839 3b7e  .h&.*.3..)...9;~  
0002a2e0: 5510 f5de 9334 1389 71b6 d83b 53d6 e0e7  U....4..q..;S...  
0002a2f0: 50c0 f3a1 b3ce 900f 5a19 7fff d962 6f72  P.......Z....bor  
0002a300: 6f43 5446 7b66 3072 336e 7369 635f 406e  oCTF{f0r3nsic_@n  
0002a310: 616c 7973 6973 237d 0a                   alysis#}.
```

`boroCTF{f0r3nsic_@nalysis#}`

___

## File Me to the Moon

**Author**: ForeverFlames

**Category**: Forensics

**Description**: Frank sinatra accidentally deleted the file extension on one of his files!!! What file extension is it supposed to be??? `boroCTF{file_extension}`

```shell
file superfile  
superfile: Microsoft PowerPoint 2007+
```

`boroCTF{pptx}`

___

## File-et Mignon

**Author**: ForeverFlames

**Category**: Forensics

**Description**: Don't try to bite off more than you can chew.

I download the challenge file:
```shell
file filet_mignon_challenge.tar.gz  
filet_mignon_challenge.tar.gz: gzip compressed data, from Unix, original size modulo 2^32 40960
```

Before trying to decompress the file I opened it in my file explorer. It was labeled as 11 terabytes!! Instead of trying to decompress the entire thing, I manually inspect the first 30KB with python:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import gzip  
... import io  
...  
... with open('filet_mignon_challenge.tar.gz', 'rb') as f:  
...     with gzip.GzipFile(fileobj=f, mode='rb') as gz:  
...         data = gz.read(30000)  
...         with open('first-part-of-data.bin', 'wb') as out:  
...             out.write(data)  
...         print("Data written")  
...  
30000  
Data written  
>>> exit

cat first-part-of-data.bin  
filet_mignon.bin0000644000175000017500000010000015212261173024417 Sustar  ameramer��ԥ00000010000�ѩJ 00000010000��}�000000010000��R�@00000010000�  
��'9P00000010000�t��`00000010000�]Ѓp00000010000�F�(�00000010000�  
00000000000boroCTF{y0u_c4rv3d_th3_v01d_l1k3_4_ch3f}
```

`boroCTF{y0u_c4rv3d_th3_v01d_l1k3_4_ch3f}`

___

## Meeting Location

**Author**: snzodiac

**Category**: Forensics

**Description**: I've got the network traffic from a well-known athlete. We don't know who the athlete is yet, but we'll address that after we confirm where they are meeting the other party. You know he will pay big bucks if we can get pictures of this athlete and why they are there for him. They're definitely talking in code about a secret meeting place in these packets. Take a look when you have a second. If you can figure out where they are heading, there's 200 boroPoints in it for you. This could be the end of all our suffering if we figure this out.

I download the challenge file:
```shell
file meeting.pcap  
meeting.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Raw IPv4, capture length 65535)
```

I opened the pcap in wireshark and saw 1504 packets. There are many different types of packets in the capture but most of them lead nowhere. I eventually found 24 ICMP packets with ID 0. They each contained a single printable ASCII byte payload. I manually concatenated the base64 string and decoded:
```shell
echo 'WWFzX01hcmluYV9DaXJjdWl0' | base64 -d  
Yas_Marina_Circuit
```

`boroCTF{Yas_Marina_Circuit}`

___

## Judgment of Solomon

**Author**: N/A

**Category**: Forensics

**Description**: The birth of reconstruction.

I get the challenge file:
```shell
file code  
code: ASCII text, with very long lines (26307)

cat code | wc  
1       2   26316

cat code
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF000000000000000000000000000000000000FF0000FF0000000000000000FFFFFFFFFFFF000000000000000000000000FF0000FF0000FF0000FF0000000000000000FFFFFFFFFFFF000000000000FFFFFFFFFFFF000000000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0AFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF000000000000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0000000000000000000000000000000... etc
```
`26316` characters. 

I found a fake flag in the file:
```shell
xxd code | grep -C 3 boro  
00002e00: 3030 3030 3030 4646 4646 4646 4646 4646  000000FFFFFFFFFF  
00002e10: 4646 3030 3030 3030 3030 3030 3030 4646  FF000000000000FF  
00002e20: 4646 4646 4646 4646 4646 3030 3030 3030  FFFFFFFFFF000000  
00002e30: 3030 3030 3030 626f 726f 4354 467b 495f  000000boroCTF{I_  
00002e40: 4330 784c 366e 222b 5f64 305f 6974 5f53  C0xL6n"+_d0_it_S  
00002e50: 7431 316e 7a5f 6e30 775f 676f 7d46 4646  t11nz_n0w_go}FFF  
00002e60: 4646 4646 4646 4646 4630 3030 3030 3030  FFFFFFFFF0000000
```

I run this python to interpret the file as a grid of 6 character hex triplets (RRGGBB) separated by the characters `0A` as row delimiters:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import re  
... from PIL import Image  
... from collections import Counter  
...  
... with open('code', 'r') as f:  
...     content = f.read().strip()  
...  
... # remove any newline characters just in case  
... content = content.replace('\n', '').replace('\r', '')  
...  
... # split the text by the literal string 0A
... rows = content.split('0A')  
...  
... # remove a trailing empty row if the file ended with 0A 
... if rows and not rows[-1]:  
...     rows.pop()  
...  
... # dynamically find the correct width
... lengths = [len(r) for r in rows]  
... most_common_len = Counter(lengths).most_common(1)[0][0]  
... width = most_common_len // 6  
...  
... img_data = []  
...  
... for row in rows:  
...     # replace the fake flag with 0. this will add some noise i think
...     cleaned_row = re.sub(r'[^0-9A-Fa-f]', '0', row)  
...  
...     # ensure perfect grid alignment :)
...     target_len = width * 6  
...     if len(cleaned_row) < target_len:  
...         cleaned_row = cleaned_row.ljust(target_len, '0')  
...     elif len(cleaned_row) > target_len:  
...         cleaned_row = cleaned_row[:target_len]  
...  
...     # Convert hex chunks into RGB tuples  
...     row_pixels = []  
...     for i in range(0, len(cleaned_row), 6):  
...         hex_color = cleaned_row[i:i+6].ljust(6, '0')  
...         r = int(hex_color[0:2], 16)  
...         g = int(hex_color[2:4], 16)  
...         b = int(hex_color[4:6], 16)  
...         row_pixels.append((r, g, b))  
...  
...     img_data.append(row_pixels)  
...  
... height = len(img_data)  
... print(f"Corrected grid size found: {width}x{height}")  
...  
... # create base pixel image  
... img = Image.new('RGB', (width, height))  
... pixels = img.load()  
...  
... for y in range(height):  
...     for x in range(width):  
...         pixels[x, y] = img_data[y][x]  
...  
... # upscale so it's actually scannable
... scale = 10  
... img_resized = img.resize((width * scale, height * scale), Image.Resampling.NEAREST)  
...  
... img_resized.save('real_challenge_image.png')  
... print("image saved as 'real_challenge_image.png'")  
...  
Corrected grid size found: 66x67  
image saved as 'real_challenge_image.png'  
>>> exit
```

Output image:

![Alt text](/images/real_challenge_image.png)

It kind of looks like a QR code but the star and the smiley face in the top and bottom left corners need to go. In a last ditch effort I put the image into [https://viliusle.github.io/miniPaint/](https://viliusle.github.io/miniPaint/) and just started editing the QR code into what I thought would be readable. My final image:

![Alt text](/images/final-fixed-qr-code.png)

Sites like [https://qrscanner.net/](https://qrscanner.net/) still would not scan this, but my phone was able to scan it! I think the error correction for scanning QR codes on our phones is better than it might be on a site like qrscanner.net.

`boroCTF{I_f1%ed_wHat_w4$_br0Ken}`

___

## Grep'n it

**Author**: Franklin

**Category**: Forensics

**Description**: I lost my flag in a stockpile of fake flags :(. Can you help me find it?

I download the challenge file `chal`:
```shell
file chal  
chal3: ASCII text  

cat chal | wc  
10746   10746  225663

cat chal | grep 'boroCTF{'  
boroCTF{G63p_G0d}
```

`boroCTF{G63p_G0d}`

___

## Billie Eilish

**Author**: ForeverFlames

**Category**: Forensics

**Description**: Although she is so beautiful, its whats on the inside that matters.

I get the challenge file `billie.jpg`:
```shell
file billie.jpg  
billie.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, progressive, precision 8, 1500x1000, components 3
```

There is a zip file appended:
```shell
xxd billie.jpg | tail  
0003cfd0: c62f b79a f0d8 5225 080e e819 a2f0 3e66  ./....R%......>f  
0003cfe0: 2cfb 3ef8 cdea e587 b9aa 4a50 00a5 7d04  ,.>.......JP..}.  
0003cff0: 504b 0708 81a6 3921 f1c0 0100 2d84 0200  PK....9!....-...  
0003d000: 504b 0102 1e03 1400 0900 0800 34ae b45c  PK..........4..\  
0003d010: 81a6 3921 f1c0 0100 2d84 0200 0a00 1800  ..9!....-.......  
0003d020: 0000 0000 0000 0000 ff81 0000 0000 6569  ..............ei  
0003d030: 6c69 7368 2e70 6e67 5554 0500 03b4 640e  lish.pngUT....d.  
0003d040: 6a75 780b 0001 04e8 0300 0004 e803 0000  jux.............  
0003d050: 504b 0506 0000 0000 0100 0100 5000 0000  PK..........P...  
0003d060: 45c1 0100 0000                           E.....
```

I extract the zip. It is password protected:
```shell
binwalk -e billie.jpg  
  
DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
134843        0x20EBB         Zip archive data, encrypted at least v2.0 to extract, compressed size: 114929, uncompressed size: 164909, name: eilish.png  
  
WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

Crack the password for the zip archive:
```shell
zip2john 20EBB.zip > zip.hash  
ver 2.0 efh 5455 efh 7875 20EBB.zip/eilish.png PKZIP Encr: TS_chk, cmplen=114929, decmplen=164909, crc=2139A681 ts=AE34 cs=ae34 type=8  

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
badguy           (20EBB.zip/eilish.png)  
1g 0:00:00:00 DONE (2026-06-13 19:40) 2.040g/s 83591p/s 83591c/s 83591C/s holly2..louis123  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

Unzip it with the password `badguy`:
```shell
unzip 20EBB.zip  
Archive:  20EBB.zip  
[20EBB.zip] eilish.png password:  
  inflating: eilish.png
```

The flag is on her shirt in the image:

![Alt text](/images/thumbnail564782.jpeg)

`boroCTF{im_a_good_guy}`

___

## The Shattered Needle

**Author**: ForeverFlames

**Category**: Forensics

**Description**: Man I hate haystacks. Where is my needle!!!

I get the challenge file `the_haystack.zip`:
```shell
file the_haystack.zip  
the_haystack.zip: Zip archive data, at least v2.0 to extract, compression method=store
```

The zip file contains `110100` files:
```shell
zipinfo the_haystack.zip | head  
Archive:  the_haystack.zip  
Zip file size: 16533967 bytes, number of entries: 110100  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_1/  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_10/  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_100/  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_11/  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_12/  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_13/  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_14/  
drwxrwxrwx  2.0 fat        0 b- stor 26-Mar-18 15:16 dir_15/  

zipinfo the_haystack.zip | tail  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_10.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_2.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_3.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_4.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_5.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_6.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_7.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_8.txt  
-rw-rw-rw-  2.0 fat       29 b- defN 26-Mar-18 15:16 dir_99/sub_99/data_9.txt  
110100 files, 2900085 bytes uncompressed, 3100085 bytes compressed:  -6.9%
```

It is all text files nested inside directories. I got the contents of all the text files into one file like this:
```shell
unzip -p the_haystack.zip > haystack.txt
```

Every time I grepped `haystack.txt` it would output the entire file. I ended up putting it into cyberchef and manually searching for the flag parts. I found this:
```shell
[FLAG_FRAGMENT_1/5]: boroCTF{gr3p_
[FLAG_FRAGMENT_2/5]: 1s_y0ur_b3st_
[FLAG_FRAGMENT_3/5]: fr13nd_f0r_
[FLAG_FRAGMENT_4/5]: 1nc1d3nt_
[FLAG_FRAGMENT_5/5]: r3sp0ns3}
```

`boroCTF{gr3p_1s_y0ur_b3st_fr13nd_f0r_1nc1d3nt_r3sp0ns3}`

___

## Silent Sentinel

**Author**: ForeverFlames

**Category**: Forensics

**Description**: We intercepted a packet capture containing a mix of modern station telemetry and an anomalous TCP file transfer. Analysts believe the rogue uplink successfully recovered an image of an iconic historical spacecraft where the crime must have taken place originally. Flag Format: `boroCTF{satellite_name_with_underscores}`

I opened the pcap in wireshark and found one udp stream and one tcp stream. The udp stream didnt have much interesting. The tcp stream contained this at the top:
```shell
......JFIF.....H.H...../Satellite, Vanguard 1, Backup (A19761019000)...1.Photoshop 3.0.8BIM.%........B..,cn.hd.....8BIM........<?xml version="1.0" encoding="UTF-8"?> ... etc
```

`boroCTF{Vanguard_1}`

___

## Retinal Burn

**Author**: Franklin

**Category**: Forensics

**Description**: My friend Jonas Wagner sent me a challenge but I can't be bothered to do it. He was always one to be working on his own sorts of projects and stuff. You do it.

I get the challenge file `burn.png`:

![Alt text](/images/burn.png)

I put the file into stegsolve and eventually a random color map made the flag readable:

![Alt text](/images/ow-my-eyes.bmp)

There is a fake flag in the red plane I think which made it hard to read the real flag underneath,

`boroCTF{0W_^MY_E7ES!}`

___

## Looking through Windows

**Author**: Solarity

**Category**: Forensics

**Description**: My friend thinks he can hide his secrets from me by deleting them... Who's gonna tell him?

I download the challenge file:
```shell
unzip challenge.zip  
Archive:  challenge.zip  
  inflating: challenge.vhd  

file challenge.vhd  
challenge.vhd: DOS/MBR boot sector MS-MBR Windows 7 english at offset 0x163 "Invalid partition table" at offset 0x17b "Error loading operating system" at offset 0x19a "Missing operating system", disk signature 0x991778b6; partition 1 : ID=0x7, start-CHS (0x0,2,3), end-CHS (0x5,254,57), startsector 128, 96256 sectors
```

I get a VHD (virtual hard disk) file.

List files in the filesystem with `fls`:
```shell
fls -o 128 -r challenge.vhd  
r/r 4-128-1:    $AttrDef  
r/r 8-128-2:    $BadClus  
r/r 8-128-1:    $BadClus:$Bad  
r/r 6-128-4:    $Bitmap  
r/r 7-128-1:    $Boot  
d/d 11-144-4:   $Extend  
+ d/d 29-144-2: $Deleted  
+ r/r 25-144-2: $ObjId:$O  
+ r/r 24-144-3: $Quota:$O  
+ r/r 24-144-2: $Quota:$Q  
+ r/r 26-144-2: $Reparse:$R  
+ d/d 27-144-2: $RmMetadata  
++ r/r 28-128-4:        $Repair  
++ r/r 28-128-2:        $Repair:$Config  
++ d/d 31-144-2:        $Txf  
++ d/d 30-144-2:        $TxfLog  
+++ r/r 32-128-2:       $Tops  
+++ r/r 32-128-4:       $Tops:$T  
+++ r/r 33-128-1:       $TxfLog.blf  
+++ r/r 34-128-1:       $TxfLogContainer00000000000000000001  
+++ r/r 35-128-1:       $TxfLogContainer00000000000000000002  
r/r 2-128-1:    $LogFile  
r/r 0-128-6:    $MFT  
r/r 1-128-1:    $MFTMirr  
d/d 40-144-1:   $RECYCLE.BIN  
+ d/d 41-144-1: S-1-5-21-560082540-3504724815-20780683-1001  
++ r/r 42-128-1:        desktop.ini  
++ -/r * 39-128-1:      $RIFYI8L.zip  
++ -/r * 43-128-1:      $IIFYI8L.zip  
r/r 9-128-8:    $Secure:$SDS  
r/r 9-144-11:   $Secure:$SDH  
r/r 9-144-14:   $Secure:$SII  
r/r 10-128-1:   $UpCase  
r/r 10-128-4:   $UpCase:$Info  
r/r 3-128-3:    $Volume  
d/d 36-144-1:   System Volume Information  
+ r/r 38-128-1: IndexerVolumeGuid  
+ r/r 37-128-1: WPSettings.dat  
V/V 256:        $OrphanFiles
```

There is a deleted zip file. `$RIFYI8L.zip` is the actual deleted archive. `$IIFYI8L.zip` contains metadata. 

Use `icat` (from The Sleuth Kit) with the partition offset (`-o 128`) and the inode number of the `$R` file:
```shell
icat -o 128 challenge.vhd 39 > recovered-deleted.zip
```

The zip is password protected:
```shell
unzip recovered-deleted.zip  
Archive:  recovered-deleted.zip  
[recovered-deleted.zip] flag.txt password:
```

Crack the password:
```shell
zip2john recovered-deleted.zip > zip.hash2  
ver 2.0 recovered-deleted.zip/flag.txt PKZIP Encr: cmplen=41, decmplen=29, crc=FA2B3A69 ts=8366 cs=fa2b type=0  

john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash2  
Using default input encoding: UTF-8  
Loaded 1 password hash (PKZIP [32/64])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
forget92936281   (recovered-deleted.zip/flag.txt)  
1g 0:00:00:03 DONE (2026-06-13 21:06) 0.3125g/s 2521Kp/s 2521Kc/s 2521KC/s formeplus..ford5001  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

Unzip with the password `forget92936281`:
```shell
unzip recovered-deleted.zip  
Archive:  recovered-deleted.zip  
[recovered-deleted.zip] flag.txt password:  
replace flag.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: y  
 extracting: flag.txt  

cat flag.txt  
boroCTF{f!l3_f0r3nsics_FTW!!}
```

`boroCTF{f!l3_f0r3nsics_FTW!!}`

___

## Satoshi: A Memory of The Past

**Author**: ForeverFlames (yours truly)

**Category**: Forensics

**Description**: All these crimes. Its not him. His soul is gone. It took his soul and trapped it inside the cache. I still hear his pulse.... Please don't lose his memory. I can't live without my beloved Satoshi.

> Satoshi's wife, Yukiko Nakamuda

(Note: the flag is already wrapped in boroCTF{} when you find it)

**UNRELATED TO THE OSINT CHALLENGES _(other than the plot)_**

I get the challenge file  `satoshi_pulse_v2`:
```shell
file satoshi_pulse_v2  
satoshi_pulse_v2: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=68afa4ac35b6a5fb2d7d0ec69  
cef42fe6c12ec6e, for GNU/Linux 3.2.0, not stripped  

./satoshi_pulse_v2 | head  
私はまだここにいます... (I am still here...)  
The static mind is blind. The demons have learned your tricks.  
  
318  
10435  
7536  
314  
281  
315  
7518
```

I put the binary into ghidra and see this main function:
```shell
undefined8 main(void)

{
  undefined4 uVar1;
  long lVar2;
  ulong uVar3;
  undefined4 extraout_var;
  undefined1 *puVar4;
  undefined1 *puVar5;
  undefined4 uVar6;
  undefined1 *puVar7;
  long in_FS_OFFSET;
  int iVar8;
  int iVar9;
  int iVar10;
  
  puVar4 = &stack0xfffffffffffffff8;
  do {
    puVar7 = puVar4;
    *(undefined8 *)(puVar7 + -0x1000) = *(undefined8 *)(puVar7 + -0x1000);
    puVar4 = puVar7 + -0x1000;
  } while (puVar7 + -0x1000 != &stack0xffffffffffdffff8);
  lVar2 = *(long *)(in_FS_OFFSET + 0x28);
  *(undefined8 *)(puVar7 + -0x10e8) = 0x10121e;
  puts(&DAT_00102008);
  *(undefined8 *)(puVar7 + -0x10e8) = 0x10122d;
  puts("The static mind is blind. The demons have learned your tricks.\n");
  for (iVar8 = 0; iVar8 < 0x80000; iVar8 = iVar8 + 1) {
    *(undefined4 *)(&stack0xffffffffffdfffe8 + (long)iVar8 * 4) = 1;
  }
  for (iVar8 = 0; iVar8 < 0x1d; iVar8 = iVar8 + 1) {
    uVar1 = *(undefined4 *)(&stack0xffffffffffdfff68 + (long)iVar8 * 4);
    for (iVar9 = 7; -1 < iVar9; iVar9 = iVar9 + -1) {
      *(undefined8 *)(puVar7 + -0x10e8) = 0x1013fc;
      demonic_logic(iVar9 + iVar8);
      uVar3 = rdtsc();
      puVar4 = (undefined1 *)(CONCAT44(extraout_var,(int)uVar3) | uVar3 & 0xffffffff00000000);
      if (((int)(char)((byte)uVar1 ^ 0x42) >> ((byte)iVar9 & 0x1f) & 1U) == 0) {
        puVar5 = puVar4;
        for (iVar10 = 0; uVar6 = (undefined4)((ulong)puVar5 >> 0x20), iVar10 < 0x40;
            iVar10 = iVar10 + 1) {
          puVar5 = (undefined1 *)0x0;
        }
      }
      else {
        puVar5 = puVar4;
        for (iVar10 = 0; iVar10 < 0x80000; iVar10 = iVar10 + 0x1000) {
          puVar5 = &stack0xffffffffffdfffe8 + (long)iVar10 * 4;
          clflush(*puVar5);
        }
        for (iVar10 = 0; uVar6 = (undefined4)((ulong)puVar5 >> 0x20), iVar10 < 0x80000;
            iVar10 = iVar10 + 0x1000) {
          puVar5 = (undefined1 *)0x0;
        }
      }
      uVar3 = rdtsc();
      *(undefined8 *)(puVar7 + -0x10e8) = 0x10151d;
      printf("%llu\n",(CONCAT44(uVar6,(int)uVar3) | uVar3 & 0xffffffff00000000) - (long)puVar4);
    }
  }
  if (lVar2 == *(long *)(in_FS_OFFSET + 0x28)) {
    return 0;
  }
                    // WARNING: Subroutine does not return
  *(undefined8 *)(puVar7 + -0x10e8) = 0x101563;
  __stack_chk_fail();
}
```

The binary takes a character from the stack (`uVar1`), XORs it with `0x42`, and checks if the current bit is `0` or `1`:
```c
if (((int)(char)((byte)uVar1 ^ 0x42) >> ((byte)iVar9 & 0x1f) & 1U) == 0)
```

If the bit is 0 it executes a small dummy loop 64 times (`iVar10 < 0x40`) that does almost nothing. It executes very quickly.

If the bit is 1 it executes a loop 128 times that actively clears CPU cache lines using the `clflush` instruction. This takes a lot longer.

This leaks the bits of the result via the cache.

I run the program and put the output numbers into a text file. Then run this python:
```python
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> # load timing numbers  
... with open("timings-clean.txt") as f:  
...     timings = [int(line.strip()) for line in f if line.strip()]  
...  
... # define a safe threshold between the fast and slow cache hits  
... THRESHOLD = 2000  
...  
... bits = [1 if t > THRESHOLD else 0 for t in timings]  
... flag = ""  
...  
... # process 8 bits at a time (MSB to LSB)  
... for i in range(0, len(bits), 8):  
...     byte_bits = bits[i:i+8]  
...  
...     byte_value = 0  
...     for bit in byte_bits:  
...         byte_value = (byte_value << 1) | bit  
...  
...     flag += chr(byte_value)  
...  
... print(f"Flag: {flag}")  
...  
Flag: boroCTF{s4t0sh1_1n_th3_c4ch3}
```

`boroCTF{s4t0sh1_1n_th3_c4ch3}`

___

## Eschew

**Author**: Franklin

**Category**: Forensics

**Description**: My image is flipped and flopped :(.

I get the challenge file `eschew.png`:

![Alt text](/images/eschew.png)

I put the image into gimp and used the transform tool `shear`. I set the second shear value to `900` and after looking at it for a while I was able to read the flag:

![Alt text](/images/best-image-i-can-get.png)

`boroCTF{SAT_1s_H@rd}`

___

## Mark Zuckerburg

**Author**: Franklin

**Category**: Forensics

**Description**: Mark (from Meta) called me the other day and attached this photo. He told me he needed to remember what model camera he took his picture with. Could you help me?

```shell
exiftool Mark-Zuckerberg.png  
ExifTool Version Number         : 13.25  
File Name                       : Mark-Zuckerberg.png  
Directory                       : .  
File Size                       : 574 kB  
File Modification Date/Time     : 2026:06:14 17:15:50-04:00  
File Access Date/Time           : 2026:06:14 17:15:46-04:00  
File Inode Change Date/Time     : 2026:06:14 17:15:50-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 1638  
Image Height                    : 922  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
White Point X                   : 0.3127  
White Point Y                   : 0.329  
Red X                           : 0.64  
Red Y                           : 0.33  
Green X                         : 0.3  
Green Y                         : 0.6  
Blue X                          : 0.15  
Blue Y                          : 0.06  
Pixels Per Unit X               : 3779  
Pixels Per Unit Y               : 3779  
Pixel Units                     : meters  
Background Color                : 255 255 255  
Datecreate                      : 2026-02-10T17:49:04+00:00  
Datemodify                      : 2026-02-10T17:49:04+00:00  
Datetimestamp                   : 2026-02-10T17:49:04+00:00  
About                           : uuid:faf5bdd5-ba3d-11da-ad31-d33d75182f1b  
Focal Length In 35mm Format     : 10000 mm  
User Comment                    : I LOVE my new job at meta  
Date/Time Original              : 2190:02:10 12:53:41.947  
Flash Fired                     : True  
Flash Return                    : No return detection  
Flash Mode                      : Auto  
Flash Function                  : False  
Flash Red Eye Mode              : True  
ISO                             : 1000  
Metering Mode                   : Unknown (0)  
Creator                         : Mark  
Make                            : Sonya  
Camera Model Name               : boroCTF{M3+a_d@ta_1s_M7_Fa40rite}  
Rights                          : BoroCTF  
Subject                         : #FirstDAY  
Title                           : Mark's Big Day  
Create Date                     : 2190:02:10 12:53:41.947  
Creation Time                   : 2190:02:10 12:53:41  
Image Size                      : 1638x922  
Megapixels                      : 1.5  
Flash                           : Auto, Fired, Red-eye reduction
```

`boroCTF{M3+a_d@ta_1s_M7_Fa40rite}`

___

## Listen Close

**Author**: Franklin

**Category**: Forensics

**Description**: Those who listen closely can read between the lines. It is not always what is being said but the meanings behind it.

I get the challenge file:
```shell
file chal.wav  
chal.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 48000 Hz
```

I opened the wav in audacity and looked at the spectrogram view:

![Alt text](/images/spectrogram-view-563725.png)

`boroCTF{Sp3c^R0}`
