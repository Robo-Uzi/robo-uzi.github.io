---
layout: post
title:  "Steganography Challenges"
date:   2026-04-19 16:59:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /CTF@CIT-2026-stego/
---
* TOC
{:toc}

## [INSERT CHALLENGE TITLE HERE]

**Author**: boom

**Category**: Steganography

**Description**: `[[INSERT CHALLENGE DESCRIPTION HERE]` SHA1: `1cdc80f8c797645b9d92e34a4c6c09022e5378aa`

I get the challenge file:

![Alt text](/images/flag.jpg)

I find the flag with `file`:
```shell
sha1sum flag.jpg  
1cdc80f8c797645b9d92e34a4c6c09022e5378aa  flag.jpg  

file flag.jpg  
flag.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=5, description=CIT{ur_w4rm1ng_up_n0w}, xresolution=98, yresolution=106, resolutionunit=2], baseline, precision 8, 1080x1080, components 3
```

Or `exiftool`:
```shell
exiftool flag.jpg | grep CIT
Image Description               : CIT{ur_w4rm1ng_up_n0w}
```

`CIT{ur_w4rm1ng_up_n0w}`

___

## There's no room left

**Author**: boom

**Category**: Steganography

**Description**: It almost feels like the walls are closing in. Digitally, that is ;) SHA1: `0f425ef58c8922ef1bad11929b317bc9d7b21a73`

I get the challenge file:
```shell
file flag.txt  
flag.txt: Unicode text, UTF-8 text, with very long lines (391), with no line terminators

cat flag.txt  
Another ‌‌‌‌‍‬﻿﻿year‌‌‌‌‍‬‬﻿‌, ‌‌‌‌‍‬﻿‍another‌‌‌‌‍‬‌‍ ‌‌‌‌‍﻿‬‍steg challenge.. Something-‌‌‌‌‍‬‌‬something‌‌‌‌‍‬‍‍ the flag is‌‌‌‌‍‍﻿﻿ hidden ‌‌‌‌‍‬﻿‬in‌‌‌‌‍‬﻿﻿‌ plain ‌‌‌‌‍﻿‌‌sight‌‌‌‌‍‬﻿‌,‌‌‌‌‍‬‌‍ but I'll‌‌‌‌‍‬‬‍ leave it up ‌‌‌‌‍‬﻿‬to you to see‌‌‌‌‍‍﻿﻿ if‌‌‌‌‍﻿‌﻿‌ that really‌‌‌‌‍‬‍﻿‌ is‌‌‌‌‍﻿‍‌ true or‌‌‌‌‍﻿﻿‍ not!

xxd flag.txt | head  
00000000: e280 8ce2 808c e280 8ce2 808c e280 8de2  ................  
00000010: 808c e280 8cef bbbf e280 8ce2 808c e280  ................  
00000020: 8ce2 808c e280 8de2 808c e280 ace2 808d  ................  
00000030: e280 8ce2 808c e280 8ce2 808c e280 8de2  ................  
00000040: 808d e280 8de2 808c e280 8ce2 808c e280  ................  
00000050: 8ce2 808c e280 8def bbbf e280 acef bbbf  ................  
00000060: 416e 6f74 6865 7220 e280 8ce2 808c e280  Another ........  
00000070: 8ce2 808c e280 8de2 80ac efbb bfef bbbf  ................  
00000080: 7965 6172 e280 8ce2 808c e280 8ce2 808c  year............  
00000090: e280 8de2 80ac e280 acef bbbf e280 8ce2  ................

xxd flag.txt | tail  
000002c0: efbb bfe2 808c e280 8ce2 808c e280 8ce2  ................  
000002d0: 808d e280 ace2 80ac e280 8d20 7468 6174  ........... that  
000002e0: 2072 6561 6c6c 79e2 808c e280 8ce2 808c   really.........  
000002f0: e280 8ce2 808d e280 ace2 808d efbb bfe2  ................  
00000300: 808c e280 8ce2 808c e280 8ce2 808d e280  ................  
00000310: ace2 80ac e280 8c20 6973 e280 8ce2 808c  ....... is......  
00000320: e280 8ce2 808c e280 8def bbbf e280 8de2  ................  
00000330: 808c 2074 7275 6520 6f72 e280 8ce2 808c  .. true or......  
00000340: e280 8ce2 808c e280 8def bbbf efbb bfe2  ................  
00000350: 808d 206e 6f74 21                        .. not!
```

I put the text into [https://www.babelstone.co.uk/Unicode/whatisit.html](https://www.babelstone.co.uk/Unicode/whatisit.html) and find they are using invisible unicode in the text like `ZERO WIDTH NON-JOINER`, `ZERO WIDTH JOINER`, `POP DIRECTIONAL FORMATTING`, and `ZERO WIDTH NO-BREAK SPACE`:
```shell
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+FEFF : ZERO WIDTH NO-BREAK SPACE [ZWNBSP] (alias BYTE ORDER MARK [BOM]) {BOM, ZWNBSP}
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+202C : POP DIRECTIONAL FORMATTING [PDF]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+FEFF : ZERO WIDTH NO-BREAK SPACE [ZWNBSP] (alias BYTE ORDER MARK [BOM]) {BOM, ZWNBSP}
U+202C : POP DIRECTIONAL FORMATTING [PDF]
U+FEFF : ZERO WIDTH NO-BREAK SPACE [ZWNBSP] (alias BYTE ORDER MARK [BOM]) {BOM, ZWNBSP}
U+0041 : LATIN CAPITAL LETTER A
U+006E : LATIN SMALL LETTER N
U+006F : LATIN SMALL LETTER O
U+0074 : LATIN SMALL LETTER T
U+0068 : LATIN SMALL LETTER H
U+0065 : LATIN SMALL LETTER E
U+0072 : LATIN SMALL LETTER R
U+0020 : SPACE [SP]
... etc
```

The flag is obtained by extracting the zero width characters from `flag.txt`, mapping them to base4 digits (`U+200C=0`, `U+200D=1`, `U+202C=2`, `U+FEFF=3`), converting the entire sequence to bytes, then finally taking every other byte (the odd indexed ones) to form the flag string:
```python
data = open("flag.txt", encoding="utf-8").read()

# base 4 digits
mapping = {
    '\u200c': '00',  # ZWNJ = 0
    '\u200d': '01',  # ZWJ = 1
    '\u202c': '10',  # PDF = 2
    '\ufeff': '11',  # BOM = 3
}

# extract only relevant chars and build bitstream
bits = []
for c in data:
    if c in mapping:
        bits.append(mapping[c])

bitstream = ''.join(bits)

# group into bytes (8 bits)
bytes_out = [
    int(bitstream[i:i+8], 2)
    for i in range(0, len(bitstream), 8)
    if len(bitstream[i:i+8]) == 8
]

# take every other byte (odd index)
filtered_bytes = bytes_out[1::2]

# convert to string
flag = ''.join(chr(b) for b in filtered_bytes)

print(flag)
```

Output:
```shell
python3 solve1.py  
CIT{ok_maybe_not_plain_sight}
```

`CIT{ok_maybe_not_plain_sight}`

___

## Cool Car

**Author**: ronnie

**Category**: Steganography

**Description**: I got this cool car here, maybe you can find a flag. Flag Format: `CIT{example_flag}`

I get the challenge file:

![Alt text](/images/cool_car.png)

```shell
file cool_car.png  
cool_car.png: PNG image data, 3000 x 1687, 8-bit/color RGBA, non-interlaced

exiftool cool_car.png  
ExifTool Version Number         : 13.25  
File Name                       : cool_car.png  
Directory                       : .  
File Size                       : 7.2 MB  
File Modification Date/Time     : 2026:04:18 13:15:06-04:00  
File Access Date/Time           : 2026:04:18 13:15:05-04:00  
File Inode Change Date/Time     : 2026:04:18 13:15:10-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 3000  
Image Height                    : 1687  
Bit Depth                       : 8  
Color Type                      : RGB with Alpha  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Profile Name                    : ICC Profile  
Profile CMM Type                : Linotronic  
Profile Version                 : 2.1.0  
Profile Class                   : Display Device Profile  
Color Space Data                : RGB  
Profile Connection Space        : XYZ  
Profile Date Time               : 1998:02:09 06:49:00  
Profile File Signature          : acsp  
Primary Platform                : Microsoft Corporation  
CMM Flags                       : Not Embedded, Independent  
Device Manufacturer             : Hewlett-Packard  
Device Model                    : sRGB  
Device Attributes               : Reflective, Glossy, Positive, Color  
Rendering Intent                : Perceptual  
Connection Space Illuminant     : 0.9642 1 0.82491  
Profile Creator                 : Hewlett-Packard  
Profile ID                      : 0  
Profile Copyright               : Copyright (c) 1998 Hewlett-Packard Company  
Profile Description             : sRGB IEC61966-2.1  
Media White Point               : 0.95045 1 1.08905  
Media Black Point               : 0 0 0  
Red Matrix Column               : 0.43607 0.22249 0.01392  
Green Matrix Column             : 0.38515 0.71687 0.09708  
Blue Matrix Column              : 0.14307 0.06061 0.7141  
Device Mfg Desc                 : IEC http://www.iec.ch  
Device Model Desc               : IEC 61966-2.1 Default RGB colour space - sRGB  
Viewing Cond Desc               : Reference Viewing Condition in IEC61966-2.1  
Viewing Cond Illuminant         : 19.6445 20.3718 16.8089  
Viewing Cond Surround           : 3.92889 4.07439 3.36179  
Viewing Cond Illuminant Type    : D50  
Luminance                       : 76.03647 80 87.12462  
Measurement Observer            : CIE 1931  
Measurement Backing             : 0 0 0  
Measurement Geometry            : Unknown  
Measurement Flare               : 0.999%  
Measurement Illuminant          : D65  
Technology                      : Cathode Ray Tube Display  
Red Tone Reproduction Curve     : (Binary data 2060 bytes, use -b option to extract)  
Green Tone Reproduction Curve   : (Binary data 2060 bytes, use -b option to extract)  
Blue Tone Reproduction Curve    : (Binary data 2060 bytes, use -b option to extract)  
X-random-00                     : mhfTqX6RDOY7VexDEarS1ydOVgDFVHezSsrrWfdDpuYBjkr/Kq664fCDwpozGrGYXUC70POmQ4eyERqD1tFVmdEsti5bMwzR77ckCFbE  
X-random-01                     : iK6dPGHKaBUcCu6UbHB0K04cVgJOTzopefEy047ZHkUeJyIp4HD+VMYUn2/R8XF/AycsNC+dLPIJaDKS+SuTXqtp4FC0n1zTJBuTPCrjAgT197zF+vOx  
X-random-02                     : 0ryC80KZTeyYq1H7Qtq+S8Hf8oUiUkXY+lfANmdHyY2HKqvp0KVwEr/I8UDk66NjdS2QWPIiWVYW+cw4q+8/GKzbkL/15yZekA1CrT  
X-random-03                     : AVUxKYR8qHNsmIrbzk4tsDOBDhVDoaAe2YRtnv553h/0SnsnEBPEKWY9CdWcK0Jy0FUPkF/EKfn2rLLR/bJnULcACo54ONM  
X-random-04                     : d7tQ1cCTD4up09I/TzlcxYyZqm56KixRv/phHwihan0mww53SmAFHyE  
X-random-05                     : abWWi8YgCnMLX4pwK2Mw6ASvLj300KPBiM5VdSd3ww0/wQnGM  
Image Size                      : 3000x1687  
Megapixels                      : 5.1
```

Not sure what `X-random-00` through `X-random-05` is about. I put the image into stegsolve (`java -jar StegSolve-1.4.jar`). When looking at `alpha plane 0` I find a lot of text and a base64 string in the center of the image:

![Alt text](/images/cool_car_alpha_plane_0.bmp)

```shell
echo "Q0lUezRWdTF1MXpofQ==" | base64 -d  
CIT{4Vu1u1zh}
```

`CIT{4Vu1u1zh}`
