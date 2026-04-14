---
layout: post
title:  "Ninja-Nerds"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-ninja-nerds/
---

**Category**: forensics medium

**Author**: mrgreenhathacker

**Description**: where are your little ninja-nerds?

I get the challenge file:
```shell
file challenge.png  
challenge.png: PNG image data, 640 x 360, 8-bit/color RGB, non-interlaced  

exiftool challenge.png  
ExifTool Version Number         : 13.25  
File Name                       : challenge.png  
Directory                       : .  
File Size                       : 257 kB  
File Modification Date/Time     : 2026:04:10 13:16:41-04:00  
File Access Date/Time           : 2026:04:10 13:16:41-04:00  
File Inode Change Date/Time     : 2026:04:10 18:48:36-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 640  
Image Height                    : 360  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Image Size                      : 640x360  
Megapixels                      : 0.230
```

![Alt text](/images/ninja-nerds.png)

The flag is hidden in the LSB of the blue channel only, reading row by row. Solve script:
```python
from PIL import Image
import numpy as np

img = Image.open('challenge.png').convert('RGB')
pixels = np.array(img)

# try per channel extraction
for channel_name, channel_idx in [('R',0),('G',1),('B',2)]:
    channel = pixels[:,:,channel_idx].flatten()
    for bit in range(8):
        bits = ((channel >> bit) & 1).tolist()
        result = bytearray()
        for i in range(0, len(bits)-7, 8):
            byte = 0
            for b in range(8):
                byte = (byte << 1) | bits[i+b]
            result.append(byte)
        text = bytes(result)
        if b'UMASS' in text:
            idx = text.find(b'UMASS')
            print(f'Found! Channel {channel_name} bit {bit}:', text[idx:idx+50])
```

Output:
```shell
python3 solve1.py  
Found! Channel B bit 0: b'UMASS{perfectly-hidden-ready-to-strike}\x00\xac\x0f\xcf\xf0\xec\x0b\xa9\x80\x00\n'
```

`UMASS{perfectly-hidden-ready-to-strike}`