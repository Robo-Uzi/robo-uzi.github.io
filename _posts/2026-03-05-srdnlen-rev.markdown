---
layout: post
title:  "Artistic Warmup"
date:   2026-03-05 20:18:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /srdnlen-2026-artistic-warmup/
---

**Title**: Artistic warmup

**Category**: rev

**Author**: salsa

**Description**: just an easy warmup

I get the challenge file:
```shell
file rev_artistic_warmup.exe  
rev_artistic_warmup.exe: PE32+ executable for MS Windows 5.02 (console), x86-64 (stripped to external PDB), 10 sections
```

I see this in ghidra:
```shell
undefined8 FUN_1400bfb00(void)

{
  code *pcVar1;
  longlong lVar2;
  code *pcVar3;
  code *pcVar4;
  code *pcVar5;
  code *pcVar6;
  code *pcVar7;
  code *pcVar8;
  code *pcVar9;
  code *pcVar10;
  code *pcVar11;
  undefined8 uVar12;
  undefined8 uVar13;
  undefined8 uVar14;
  longlong unaff_GS_OFFSET;
  undefined4 uVar16;
  undefined8 uVar15;
  longlong local_c0;
  undefined8 local_b6;
  undefined local_ae;
  undefined8 local_ad;
  undefined2 local_a5;
  undefined7 local_a3;
  undefined4 uStack_9c;
  longlong *local_98;
  undefined8 local_90;
  undefined local_88;
  undefined8 local_78;
  undefined8 local_70;
  undefined local_68 [12];
  undefined4 uStack_5c;
  undefined auStack_58 [12];
  
  FUN_14000cc90();
  pcVar1 = (code *)FUN_140001510(*(longlong *)
                                  (***(longlong ***)
                                      (*(longlong *)(*(longlong *)(unaff_GS_OFFSET + 0x60) + 0x18) +
                                      0x20) + 0x20),0x5fbff0fb);
  local_a3 = 0x2e323372657375;
  local_ad = 0x6c642e3233696467;
  uStack_9c = 0x6c6c64;
  local_a5 = 0x6c;
  (*pcVar1)(&local_a3);
  lVar2 = (*pcVar1)(&local_ad);
  pcVar1 = (code *)FUN_140001510(lVar2,-0x5fa34520);
  pcVar3 = (code *)FUN_140001510(lVar2,-0xa48c3);
  pcVar4 = (code *)FUN_140001510(lVar2,-0x1465e54f);
  pcVar5 = (code *)FUN_140001510(lVar2,0x7cf4fd7c);
  pcVar6 = (code *)FUN_140001510(lVar2,0x5f1ebf5d);
  pcVar7 = (code *)FUN_140001510(lVar2,0x41936715);
  pcVar8 = (code *)FUN_140001510(lVar2,-0x7fad6b3d);
  pcVar9 = (code *)FUN_140001510(lVar2,0xdd2a6fb);
  pcVar10 = (code *)FUN_140001510(lVar2,-0x3397e791);
  pcVar11 = (code *)FUN_140001510(lVar2,-0x60c410a1);
  FUN_1400b9580((longlong *)&DAT_1400c3a80,&DAT_1400c5000,(int **)0x2);
  local_98 = (longlong *)&local_88;
  local_90 = 0;
  local_88 = 0;
  FUN_1400bd9c0(&DAT_1400c3720,&local_98);
  uVar12 = (*pcVar1)(0);
  local_c0 = 0;
  local_78 = 0x1c200000028;
  local_70 = 0x200001ffffffce;
  local_68 = SUB1612((undefined  [16])0x0,0);
  uVar16 = 0;
  uStack_5c = 0;
  auStack_58 = SUB1612((undefined  [16])0x0,4);
  uVar13 = (*pcVar3)(uVar12,&local_78,0,&local_c0,0,0);
  (*pcVar5)(uVar12,uVar13);
  local_ae = 0;
  local_b6 = 0x73616c6f736e6f43;
  uVar15 = CONCAT44(uVar16,400);
  uVar14 = (*pcVar4)(0x18,0,0,0,uVar15,0,0,0,1,0,0,3,0,&local_b6);
  uVar16 = (undefined4)((ulonglong)uVar15 >> 0x20);
  (*pcVar5)(uVar12,uVar14);
  (*pcVar6)(uVar12,0);
  (*pcVar7)(uVar12,0xffffff);
  (*pcVar8)(uVar12,0,0,local_98,CONCAT44(uVar16,(int)local_90));
  (*pcVar9)();
  lVar2 = 0;
  do {
    if ((*(byte *)(local_c0 + lVar2) ^ 0xaa) != (&DAT_1400c5020)[lVar2]) {
      FUN_1400b9580((longlong *)&DAT_1400c3a80,"Invalid flag.\n",(int **)0xe);
      goto LAB_1400bfe32;
    }
    lVar2 = lVar2 + 1;
  } while (lVar2 != 90000);
  FUN_1400b9580((longlong *)&DAT_1400c3a80,"Valid flag!\n",(int **)0xc);
LAB_1400bfe32:
  (*pcVar10)(uVar14);
  (*pcVar10)(uVar13);
  (*pcVar11)(uVar12);
  FUN_1400a9fe0(&local_98);
  return 0;
}
```

The encrypted data at `DAT_1400c5020` (90,000 bytes) should be XORed with `0xAA` to get the flag. I needed to change the dimensions of the output image to get a readable flag. First I ran this python where I got bad images and the flag was unreadable:
```python
import pefile
import struct
from PIL import Image
import numpy as np

pe = pefile.PE('./rev_artistic_warmup.exe')
image_base = pe.OPTIONAL_HEADER.ImageBase

target_rva = 0x1400c5020 - image_base

for section in pe.sections:
    if section.contains_rva(target_rva):
        print(f"Found in section: {section.Name.decode().rstrip('\x00')}")
        offset = target_rva - section.VirtualAddress + section.PointerToRawData
        print(f"File offset: 0x{offset:08x}")
        
        # Read 90000 bytes
        with open('./rev_artistic_warmup.exe', 'rb') as f:
            f.seek(offset)
            data = f.read(90000)
        
        print(f"Data length: {len(data)} bytes")
        
        xored = bytes([b ^ 0xAA for b in data])
        
        print("\nFirst 100 bytes of XORed data:")
        print(xored[:100])
        
        text = ''.join(chr(b) if 32 <= b < 127 else '.' for b in xored[:200])
        
        possible_dims = [
            (300, 300),
            (400, 225),
            (360, 250),
            (320, 281),
            (450, 200),
        ]
        
        for width, height in possible_dims:
            if width * height == 90000:
                print(f"\nTrying {width}x{height} grayscale")
                try:
                    img = Image.frombytes('L', (width, height), xored)
                    img.save(f'flag_{width}x{height}_gray.png')
                    print(f"Saved as flag_{width}x{height}_gray.png")
                except:
                    pass
        
        rgb_size = 90000 // 3
        if rgb_size * 3 == 90000:
            possible_rgb_dims = [
                (200, 150),
                (300, 100),
                (250, 120),
                (180, 166),
            ]
            for width, height in possible_rgb_dims:
                if width * height == rgb_size:
                    print(f"\nTrying {width}x{height} RGB")
                    try:
                        # Convert to numpy array and reshape
                        arr = np.frombuffer(xored, dtype=np.uint8).reshape((height, width, 3))
                        img = Image.fromarray(arr, 'RGB')
                        img.save(f'flag_{width}x{height}_rgb.png')
                        print(f"Saved as flag_{width}x{height}_rgb.png")
                    except:
                        pass
        
        break
```

At this point the best image I could see was `flag_300x100_rgb.png`. The XOR decoded bytes represent raw image data, but the correct width and height were not immediately obvious:

![Alt text](/images/flag_300x100_rgb.png)

I took this image and ran this python:
```python
from PIL import Image
import numpy as np

# Load the RGB image
img = Image.open('flag_300x100_rgb.png')
data = np.array(img)

# Extract each channel
r = data[:,:,0]
g = data[:,:,1]
b = data[:,:,2]

for channel, name in [(r, 'red'), (g, 'green'), (b, 'blue')]:
    channel_data = channel.flatten()
    
    print(f"\nTrying {name} channel:")
    for width in [600]:
        if len(channel_data) % width == 0:
            height = len(channel_data) // width
            try:
                img_ch = Image.fromarray(channel_data.reshape(height, width))
                img_ch.save(f'{name}_channel_{width}x{height}.png')
                print(f"  Saved {width}x{height}")
            except:
                pass
```

Finally I got a readable image `blue_channel_600x50.png`:

![Alt text](/images/blue_channel_600x50.png)

The color channel has nothing to do with it! There is clearly no color. Its just a grayscale bitmap image. 

`srdnlen{pl5_Charles_w1n_th3_champ1on5hip}`
