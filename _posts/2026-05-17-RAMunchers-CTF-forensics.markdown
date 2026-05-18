---
layout: post
title:  "Forensics Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RAMunchers-CTF-2026-forensics/
---
* TOC
{:toc}

## C2 No Evil

**Author**: @DuppyBad

**Description**: Employees of steelsecure's rival company, safeguard, have recently alerted us to a potential incident. They've handed us a .pcap of traffic captured from what they suspect to be an infected machine, analyse it and see if you can find how they avoided detection.

I get the challenge file:
```shell
file challenge.pcap  
challenge.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

I opened the pcap in wireshark and saw a lot of `http` packets as well as `icmp`, `tcp`, and `dns`. In the dns packets I find the suspicious domain `telemetry-cdn.net`. They make requests to subdomains like `00-kjau26zrg42v6tbrkqzv.telemetry-cdn.net`. I find 3 in total:
```shell
00-kjau26zrg42v6tbrkqzv.telemetry-cdn.net: type TXT, class IN
01-eqkmjrmv6nbrk42fsnk7.telemetry-cdn.net: type TXT, class IN
02-irhdk7i.telemetry-cdn.net: type TXT, class IN
```

If I combine each subdomain in order I get a `base32` encoded string:
```shell
kjau26zrg42v6tbrkqzveqkmjrmv6nbrk42fsnk7irhdk7i
```

The letters need to be caps for decoding:
```shell
echo "KJAU26ZRG42V6TBRKQZVEQKMJRMV6NBRK42FSNK7IRHDK7I" | base32 -d  
RAM{175_L1T3RALLY_41W4Y5_DN5}
```

`RAM{175_L1T3RALLY_41W4Y5_DN5}`

___

## Pixel Perfect

**Author**: @DuppyBad

**Description**: Recently, many of these weird .jpgs have been popping up in the office. We don't know what they're for, but we suspect it's a means of data exfiltration. See if you can get anything out of it for us.

I get the challenge file:
```shell
file challenge.jpg  
challenge.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=1], baseline, precision 8, 400x400, components 3
```

`exiftool` reveals a password `super_secret_recovery_key_2026`:
```shell
exiftool challenge.jpg  
ExifTool Version Number         : 13.25  
File Name                       : challenge.jpg  
Directory                       : .  
File Size                       : 3.4 kB  
File Modification Date/Time     : 2026:05:12 20:30:09-04:00  
File Access Date/Time           : 2026:05:12 20:30:09-04:00  
File Inode Change Date/Time     : 2026:05:12 20:30:12-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : JPEG  
File Type Extension             : jpg  
MIME Type                       : image/jpeg  
JFIF Version                    : 1.01  
Resolution Unit                 : None  
X Resolution                    : 1  
Y Resolution                    : 1  
Exif Byte Order                 : Big-endian (Motorola, MM)  
Artist                          : Password: super_secret_recovery_key_2026  
Image Width                     : 400  
Image Height                    : 400  
Encoding Process                : Baseline DCT, Huffman coding  
Bits Per Sample                 : 8  
Color Components                : 3  
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)  
Image Size                      : 400x400  
Megapixels                      : 0.160
```

`binwalk` extracts a zip archive:
```shell
binwalk -e challenge.jpg  
  
DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
3206          0xC86           Zip archive data, encrypted at least v2.0 to extract, compressed size: 85, uncompressed size: 55, name: flag.txt  
  
WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

Unzip it with the password `super_secret_recovery_key_2026` and get the flag:
```shell
cat _challenge.jpg.extracted/flag.txt  
Well done! Here is your flag: RAM{m3t4d4t4_n0t_c13an3d}
```

`RAM{m3t4d4t4_n0t_c13an3d}`
