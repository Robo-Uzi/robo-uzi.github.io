---
layout: post
title:  "Forensics Challenges"
date:   2026-11-20 01:01:00 -0400
author: uzi
tags: [CTF]
permalink: /spooky-ctf-2026-forensics/
---
* TOC
{:toc}

## The Haunted Portrait

**Category**: forensics

**Author**: Justin Pence

**Description**: An old portrait was recently discovered in a storage room at NJIT just before Halloween. Students claim the painting changes when viewed at night, and some believe a secret message is hidden inside it. The university's security team examined the image but couldn't find anything unusual. Can you uncover what the portrait has been hiding all these years?

I get the challenge image:

![Alt text](/images/haunted_portrait.png)

I put the challenge image into [https://georgeom.net/StegOnline/upload](https://georgeom.net/StegOnline/upload). Looking at the blue bit plane:

![Alt text](/images/bitplaneblue0ayyy.png)

Use rot13 to get the flag:
```shell
echo 'fcbbxl{cbegenvgf_xrrc_frpergf}' | rot13  
spooky{portraits_keep_secrets}
```

`spooky{portraits_keep_secrets}`

___

## Arachnophobia

**Category**: forensics

**Author**: Hiral Z

**Description**: Something about the SpookyCTF logo feels off… like something’s tangled in the webs

I get the challenge file:
```shell
file tangled.png  
tangled.png: PNG image data, 2000 x 2000, 8-bit/color RGBA, non-interlaced  
```

```shell
exiftool tangled.png  
ExifTool Version Number         : 13.25  
File Name                       : tangled.png  
Directory                       : .  
File Size                       : 977 kB  
File Modification Date/Time     : 2026:11:06 16:53:27-05:00  
File Access Date/Time           : 2026:11:06 16:53:26-05:00  
File Inode Change Date/Time     : 2026:11:06 16:53:30-05:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 2000  
Image Height                    : 2000  
Bit Depth                       : 8  
Color Type                      : RGB with Alpha  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
SRGB Rendering                  : Perceptual  
Gamma                           : 2.2  
Pixels Per Unit X               : 8504  
Pixels Per Unit Y               : 8504  
Pixel Units                     : meters  
Exif Byte Order                 : Big-endian (Motorola, MM)  
Orientation                     : Horizontal (normal)  
X Resolution                    : 216  
Y Resolution                    : 216  
Resolution Unit                 : inches  
Color Space                     : sRGB  
Exif Image Width                : 2000  
Exif Image Height               : 2000  
Apple Data Offsets              : (Binary data 28 bytes, use -b option to extract)  
About                           : uuid:faf5bdd5-ba3d-11da-ad31-d33d75182f1b  
Creator Tool                    : Microsoft Windows Photo Viewer 10.0.26100.1882 
Creatortool                     : Microsoft Windows Photo Viewer 10.0.26100.1882 
Warning                         : [minor] Trailer data after PNG IEND chunk  
Image Size                      : 2000x2000  
Megapixels                      : 4.0
```

`exiftool` gives me a warning saying there a trailing data:
```shell
Warning                         : [minor] Trailer data after PNG IEND chunk 
```

I ran `foremost` and got a zip file:
```shell
foremost tangled.png  
Processing: tangled.png  
|foundat=flag.txtux  
*|
```

Unzip it and get the flag:
```shell
unzip output/zip/00001908.zip  
Archive:  output/zip/00001908.zip  
  inflating: flag.txt
  
cat flag.txt  
spooky{8_l3gs_4nd_4_z1p}
```

`spooky{8_l3gs_4nd_4_z1p}`

___

## Tormented Tanny

**Category**: forensics

**Author**: jtjnjit

**Description**: An image was captured of a vicious beast while guarding his castle. The cameraman may have found just what makes this beast so furious. Unfortunately, the cameraman seems to have gone missing. Maybe the photo contains the answer.

I get the challenge file:
```shell
exiftool TormentedTanny.png  
ExifTool Version Number         : 13.25  
File Name                       : TormentedTanny.png  
Directory                       : .  
File Size                       : 2.6 MB  
File Modification Date/Time     : 2026:11:06 16:58:48-05:00  
File Access Date/Time           : 2026:11:06 16:58:48-05:00  
File Inode Change Date/Time     : 2026:11:06 16:58:51-05:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 1086  
Image Height                    : 1448  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Title                           : spooky{tormented_tanner_  
XMP Toolkit                     : Image::ExifTool 12.76  
Description                     : hates_his_  
Exif Byte Order                 : Big-endian (Motorola, MM)  
Image Description               : vetsulin}  
X Resolution                    : 72  
Y Resolution                    : 72  
Resolution Unit                 : inches  
Y Cb Cr Positioning             : Centered  
Image Size                      : 1086x1448  
Megapixels                      : 1.6
```

The flag is in the metadata. Just concatenate.

`spooky{tormented_tanner_hates_his_vetsulin}`

___

## Eisoptrophobia

**Category**: forensics

**Author**: Makuta A

**Description**: Something strange is happening in the mirror. What appears to be an ordinary reflection may be hiding something more than meets the eye. Look closely, investigate the image, and uncover what lies beneath the surface.

I get the challenge image:
```shell
file Eisoptrophobiactf.png  
Eisoptrophobiactf.png: PNG image data, 1365 x 2048, 8-bit/color RGB, non-interlaced  

exiftool Eisoptrophobiactf.png  
ExifTool Version Number         : 13.25  
File Name                       : Eisoptrophobiactf.png  
Directory                       : .  
File Size                       : 2.2 MB  
File Modification Date/Time     : 2026:11:06 17:20:15-05:00  
File Access Date/Time           : 2026:11:06 17:20:14-05:00  
File Inode Change Date/Time     : 2026:11:06 17:20:17-05:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 1365  
Image Height                    : 2048  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Image Size                      : 1365x2048  
Megapixels                      : 2.8
```

I found the flag with `zsteg`:
```shell
zsteg Eisoptrophobiactf.png  
imagedata           .. file: shared library  
chunk:0:IHDR        .. file: Adobe Photoshop Color swatch, version 0, 1365 colors; 1st RGB space (0), w 0x800, x 0x802, y 0, z 0; 2nd space (258), w 0x1fe, x 0xfeff, y 0xff, z  
0x3  
b1,r,lsb,xy         .. text: "$rGsrYtz"  
b1,g,msb,xy         .. text: "\rlSA~:\t6"  
b1,rgb,lsb,xy       .. text: "c3Bvb2t5e3RoZV9yZWZsZWN0aW9uX25ldmVyX2xpZXN9"  
b1,rgb,msb,xy       .. file: OpenPGP Public Key  
b1,bgr,msb,xy       .. text: "*Um[IU%I"  
b4,r,lsb,xy         .. text: "EDTCFbtG#X"  
b4,g,lsb,xy         .. file: OpenPGP Public Key  
b4,b,lsb,xy         .. text: "V2C5$u43"  
b4,bgr,msb,xy       .. file: OpenPGP Public Key
```

It was base64 encoded:
```shell
echo 'c3Bvb2t5e3RoZV9yZWZsZWN0aW9uX25ldmVyX2xpZXN9' | base64 -d  
spooky{the_reflection_never_lies}
```

`spooky{the_reflection_never_lies}`

___

## The Phantom Login

**Category**: forensics

**Author**: JMPenceHACKZ

**Description**: It's Halloween night at NJIT, and something strange has been happening on the network.

At 3:13 AM, the university's security team detected unusual authentication activity involving an account that should have been forgotten long ago. The SOC recovered Windows security logs and web access records, but the evidence is buried among hundreds of seemingly normal events. Your task is to investigate the suspicious activity, determine what happened, and uncover the secret left behind. The ghosts of NJIT don't give up their secrets easily.

I get the challenge files:
```shell
unzip Phantom_Login.zip  
Archive:  Phantom_Login.zip  
  inflating: PLAYER_README.md  
  inflating: case_notes.txt  
  inflating: njit_security.csv  
  inflating: njit_web.log
  
cat PLAYER_README.md  
# The Phantom Login  
**Category:** Forensics | **Difficulty:** Medium | **Points:** 250  
  
It's Halloween night at NJIT. A long-forgotten account has awakened, and something has been taken from the restricted archives. The SOC captured authentication records and web traffic, but most activity appears ordinary. Follow the phantom's trail and recover the message it left behind.  
  
## Files  
- `njit_security.csv`: Windows security event records  
- `njit_web.log`: Web application access records  
- `case_notes.txt`: Incident background  
  
**Goal:** Submit the flag in `spooky{...}` format. No network access or hosted service is needed.

cat njit_security.csv | wc -l  
237  

cat njit_web.log | wc -l  
178
```

The actions were made by `evelyn.graves`:
```shell
cat njit_web.log | grep 'evelyn.graves'  
2026-10-31T03:16:10Z src=203.0.113.77 user=evelyn.graves "GET /archives/ HTTP/1.1" status=200  
2026-10-31T03:19:08Z src=203.0.113.77 user=evelyn.graves "GET /archives/export?note=c3Bvb2t5e3RoZV9naG9zdF9sZWZ0X2FfdHJhY2V9 HTTP/1.1" status=200  
2026-10-31T03:18:20Z src=203.0.113.77 user=evelyn.graves "GET /archives/records?year=1893 HTTP/1.1" status=200  

echo 'c3Bvb2t5e3RoZV9naG9zdF9sZWZ0X2FfdHJhY2V9' | base64 -d
spooky{the_ghost_left_a_trace}
```

`spooky{the_ghost_left_a_trace}`

___

## Cleithrophobia

**Category**: forensics

**Author**: Raindrops095

**Description**: You are stuck inside of the Enterprise with no way out. The rest of the crew has gone missing. They need help finding a way back to a Federation outpost but the ship is stuck dead in space. Maybe the packet file they have can find a way to start the ship.

I get the challenge file:
```shell
file Cleithrophobia.pcap  
Cleithrophobia.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

I opened the pcap in wireshark and followed this tcp stream:
```shell
POST /bridge/ignition HTTP/1.1
Host: bridge.ncc1701.starfleet.local
User-Agent: StarfleetOS/47.3 (LCARS; NCC-1701-D)
Authorization: Basic Y2FwdGFpbi5raXJrOkVudGVycHJpc2UtMTcwMS1B
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/plain
X-Bridge-Telemetry: deflector_part1=U3Bvb2t5e0JlMA==
X-Deflector-Status: deflector_part2=bV9NZV9VcF9TYw==
X-Warp-Core-Reading: deflector_part3=MHR0eV8yMjY2fQ==

IGNITION AUTHORIZED - WARP CORE PREPARING
```

base64 decode the flag:
```shell
echo 'U3Bvb2t5e0JlMA==bV9NZV9VcF9TYw==MHR0eV8yMjY2fQ==' | base64 -d  
Spooky{Be0m_Me_Up_Sc0tty_2266}
```

`Spooky{Be0m_Me_Up_Sc0tty_2266}`

___

## Stash the Hash

**Category**: forensics

**Author**: jtjnjit

**Description**: A file was found that seems to contain detailed information about an infamous and high-profile cryptid. There account is known to contain forbidden supernatural information. If you can find out what their password is you may be able to obtain this information for yourself

I get the challenge files:
```shell
unzip StashTheHash.zip  
Archive:  StashTheHash.zip  
creating: StashTheHash/  
  inflating: StashTheHash/secretKnowledgeFolder.7z  
  inflating: StashTheHash/UserAccountInfo.txt
```

```shell
cat StashTheHash/UserAccountInfo.txt  
Username: dleeds  
Password: ad7bcd7ab3cc113f496e1840bc582bbd  
Full Name: Daniel Leeds  
Display Name: DevilLeeds  
Email: dleeds@galloway.com  
Department: Horror Management  
Employee ID: NJ-062805  
Account Created: 2022-06-08  
Last Login: 2026-10-31 18:06:06  
Account Status: Active
```

The 7z archive is password protected. The password is an md5 hash. I put it into crackstation and got the password: `newjerseydevils`

```shell
7z x StashTheHash/secretKnowledgeFolder.7z -p'newjerseydevils'  
  
7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03  
64-bit locale=en_US.UTF-8 Threads:128 OPEN_MAX:1024, ASM  
  
Scanning the drive for archives:  
1 file, 1467698 bytes (1434 KiB)  
  
Extracting archive: StashTheHash/secretKnowledgeFolder.7z  
--  
Path = StashTheHash/secretKnowledgeFolder.7z  
Type = 7z  
Physical Size = 1467698  
Headers Size = 242  
Method = LZMA2:1536k 7zAES  
Solid = -  
Blocks = 1  
  
Everything is Ok  
  
Size:       1467360  
Compressed: 1467698
```

I got this image:

![Alt text](/images/secretKnowledge.png)

I looked at the red plane of the image:

![Alt text](/images/secretknowlegeforrealtho.png)

`spooky{dleeds_isa_devil}`
