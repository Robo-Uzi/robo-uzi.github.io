---
layout: post
title:  "Forensics Challenges"
date:   2026-09-28 17:28:00 -0400
author: uzi
tags: [CTF]
permalink: /sunshine-ctf-2026-forensics/
---
* TOC
{:toc}

## NAS coal

**Category**: forensics

**Author**: oatzs

**Description**: someone put coal in my gem collection :^(

I get the challenge file:
```shell
file gem_collection.pptm  
gem_collection.pptm: Microsoft PowerPoint 2007+
```

Unzip the file:
```shell
unzip gem_collection.pptm  
Archive:  gem_collection.pptm  
  inflating: [Content_Types].xml  
  inflating: _rels/.rels  
  inflating: ppt/presentation.xml  
  inflating: ppt/slides/_rels/slide2.xml.rels  
  inflating: ppt/slides/_rels/slide1.xml.rels  
  inflating: ppt/_rels/presentation.xml.rels  
  inflating: ppt/slides/_rels/slide5.xml.rels  
  inflating: ppt/slides/_rels/slide3.xml.rels  
  inflating: ppt/slides/slide1.xml  
  inflating: ppt/slides/slide2.xml  

  ... etc ...

  inflating: customXml/_rels/item3.xml.rels  
  inflating: customXml/itemProps3.xml  
  inflating: ppt/changesInfos/changesInfo1.xml  
  inflating: docProps/core.xml  
  inflating: docProps/app.xml  
  inflating: docProps/custom.xml  
  inflating: customXml/_rels/item1.xml.rels  
  inflating: customXml/_rels/item2.xml.rels  
  inflating: ppt/vbaProject.bin
```

Using the `oletools` python package I can get the the VBA source from `vbaProject.bin`:
```shell
olevba ppt/vbaProject.bin  
olevba 0.60.2 on Python 3.13.5 - http://decalage.info/python/oletools  
===============================================================================  
FILE: ppt/vbaProject.bin  
Type: OLE  
-------------------------------------------------------------------------------  
VBA MACRO MediaCache.bas  
in file: ppt/vbaProject.bin - OLE stream: 'VBA/MediaCache'  
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  
Option Explicit  
  
Public Sub RefreshCache()  
	Dim encoded As String  
	Dim commandLine As String  
encoded = "JABjAGEAbQBwAGEAaQBnAG4AIAA9ACAAJwBzAHUAbgB7AHkAdQBwAF8AaQBzAHMAYQBfAGcAZQBtAH0AJwANAAoAJABzAG8AdQByAGMAZQAgAD0AIAAnAGgAdAB0AHAAcwA6AC8ALwBnAGUAbQAtAGMAYQBjAGgAZQAuAGUAeABhAG0AcABsAGUALgBpAG4AdgBhAGwAaQBkAC8AYwBvAGEAbAAuAGIAaQBuACcADQAKACQAZABlAHMAdABpAG4AYQB0AGkAbwBuACAAPQAgACcAYwBvAGEAbAAuAGIAaQBuACcADQAKAFsAcABzAGMAdQBzAHQAbwBtAG8AYgBqAGUAYwB0AF0AQAB7AE8AcABlAHIAYQB0AGkAbwBuAD0AJwBkAG8AdwBuAGwAbwBhAGQAJwA7ACAAQwBhAG0AcABhAGkAZwBuAD0AJABjAGEAbQBwAGEAaQBnAG4AOwAgAFMAbwB1AHIAYwBlAD0AJABzAG8AdQByAGMAZQA7ACAARABlAHMAdABpAG4AYQB0AGkAbwBuAD0AJABkAGUAcwB0AGkAbgBhAHQAaQBvAG4AfQANAAoA"  
	commandLine = "powershell.exe -NoProfile -EncodedCommand " & encoded  
	Debug.Print commandLine  
End Sub  
+----------+--------------------+---------------------------------------------+  
|Type      |Keyword             |Description                                  |  
+----------+--------------------+---------------------------------------------+  
|Suspicious|powershell          |May run PowerShell commands                  |  
|Suspicious|NoProfile           |May run PowerShell commands                  |  
|Suspicious|EncodedCommand      |May run PowerShell commands                  |  
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |  
|          |                    |used to obfuscate strings (option --decode to|  
|          |                    |see all)                                     |  
|Suspicious|Base64 Strings      |Base64-encoded strings were detected, may be |  
|          |                    |used to obfuscate strings (option --decode to|  
|          |                    |see all)                                     |  
|IOC       |powershell.exe      |Executable file name                         |  
+----------+--------------------+---------------------------------------------+
```

Decoding the base64 gives me the flag:
```shell
$campaign = 'sun{yup_issa_gem}'  
$source = 'https://gem-cache.example.invalid/coal.bin'  
$destination = 'coal.bin'  
[pscustomobject]@{Operation='download'; Campaign=$campaign; Source=$source; Destination=$destination}
```

`sun{yup_issa_gem}`

___

## you cut me off

**Category**: forensics

**Author**: oatzs

**Description**: Here's a flag! It's uhhh ....... .............. .....................uhhhhhhhhh......................... hmm.....

I get the challenge image:

![Alt text](/images/hereyougo.png)

```shell
file hereyougo.png  
hereyougo.png: PNG image data, 492 x 382, 8-bit/color RGBA, non-interlaced

xxd hereyougo.png | head  
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR  
00000010: 0000 01ec 0000 017e 0806 0000 0040 8429  .......~.....@.)  
00000020: 2800 0000 0173 5247 4200 aece 1ce9 0000  (....sRGB.......  
00000030: 0004 6741 4d41 0000 b18f 0bfc 6105 0000  ..gAMA......a...  
00000040: 0009 7048 5973 0000 0ec3 0000 0ec3 01c7  ..pHYs..........  
00000050: 6fa8 6400 0096 fc49 4441 5478 5eed fd07  o.d....IDATx^...  
00000060: 7455 5517 fe0b bf77 7cf7 beef ff5e a525  tUU....w|....^.%  
00000070: 9440 7a25 0921 1d42 efbd f7de 7b53 8a14  .@z%.!.B....{S..  
00000080: 29d2 9b80 7444 5091 2228 202a a0d2 4411  )...tDP."( *..D.  
00000090: a488 d800 01e9 bdf7 8ea0 cf37 9e95 acb0  ...........7....
```

The pngs IHDR chunk claims the image is 382 pixels tall but the IDAT stream (when fully decompressed) contains enough data for 418 rows. 

I need to overwrite the height field (offset `0x14:0x18`) from `382` to `418`. Then recompute the CRC32 over bytes `0x0C:0x1D` (the `IHDR` type + 13 data bytes) and write it at `0x1D:0x21`.

Solve script:
```python
import struct
import zlib

path = "hereyougo.png"

with open(path, "rb") as f:
    data = bytearray(f.read())

struct.pack_into(">I", data, 0x14, 418)

crc = zlib.crc32(data[0x0c:0x1d]) & 0xffffffff
struct.pack_into(">I", data, 0x1d, crc)

with open("fixed73792759.png", "wb") as f:
    f.write(data)

print("Wrote fixed73792759.png")
print("IHDR height: 418")
print(f"IHDR CRC: {crc:08x}")
```

Output:
```shell
python3 uncropimage.py  
Wrote fixed73792759.png  
IHDR height: 418  
IHDR CRC: 2578f57c
```

Fixed image:

![Alt text](/images/fixed73792759.png)

`sun{totallyoriginalchallengeidea}`

___

## Welcome Call!

**Category**: forensics

**Author**: oatzs

**Description**: Welcome to Bsides Orlando! I just got a call from the flag factory, they said they were looking for their favorite CTFer?

I get the challenge file:
```shell
file welcomecall.pcap  
welcomecall.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

In wireshark I see mostly `SIP` and `RTP` packets. I saved the rtp payload like this:
```shell
tshark -r welcomecall.pcap -Y 'rtp && ip.src == 192.0.2.10 && udp.srcport == 4000' -T fields -e rtp.payload | tr -d '\n' | xxd -r -p > rtp.raw
```

Then use `ffmpeg` to convert it into something I can listen to:
```shell
ffmpeg -f mulaw -ar 8000 -ac 1 -i rtp.raw rtp.wav
```

I listened to the audio and it was reversed. I used `ffmpeg` again to now reverse the audio:
```shell
ffmpeg -i rtp.wav -af areverse rtp_reversed.wav
```
This file explained how to spell the flag!

`sun{thankyouforplaying}`
