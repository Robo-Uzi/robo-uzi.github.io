---
layout: post
title:  "Forensics Challenges"
date:   2026-07-12 12:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /bronco-CTF-2026-forensics/
---
* TOC
{:toc}

## Bundle 99

**Category**: forensics

**Author**: yoshie878

**Description**: Yoshie found this random file laying around. Do you have any idea what this "bundle" is?

Apparently someone told him it's the 99th bundle of brushes?

What is that supposed to mean...

P.S. This challenge can be solved without downloading any software, but you'll have to hunt for a way to run the related program online and send the bundle to it.

I get the challenge file `Bundle_99`:
```shell
file Bundle_99  
Bundle_99: Zip data (MIME type "application/x-krita-resourcebundle"?)
```

I tried unzipping it:
```shell
unzip Bundle_99 
Archive:  Bundle_99
 extracting: mimetype  
  inflating: paintoppresets/Brush 99.kpp  
  inflating: preview.png  
  inflating: META-INF/manifest.xml  
  inflating: meta.xml
```

I flag was in the metadata of `Brush 99.kpp`:
```shell
exiftool 'paintoppresets/Brush 99.kpp'
... etc ...
<![CDATA[<Brush font="Segoe UI,9,-1,5, 50,0,0,0,0,0" spacing="0.2" pipe="false" type="kis_text_brush" BrushVersion="2" text="bronco{1m4n4rt15ttru5t}"/> ]]></param> <param name="hSensor" type="string"><![CDATA
... etc ...
```

`bronco{1m4n4rt15ttru5t}`

___

## LEts a GO

**Category**: forensics

**Author**: .tidalw

**Description**: Go step by step, brick by brick.

I get the challenge file `lego_bricks_challenge.zip`:
```shell
unzip lego_bricks_challenge.zip  
Archive:  lego_bricks_challenge.zip  
warning:  lego_bricks_challenge.zip appears to use backslashes as path separators  
  inflating: lego_bricks_challenge/.DS_Store  
  inflating: lego_bricks_challenge/.part_65  
  inflating: lego_bricks_challenge/.part_85  
  inflating: lego_bricks_challenge/go.sum  
 extracting: lego_bricks_challenge/.cache/.part_32  
  inflating: lego_bricks_challenge/.cache/.part_76  
  inflating: lego_bricks_challenge/.cache/sessions/.npmrc  
  inflating: lego_bricks_challenge/.cache/sessions/.part_189  
  inflating: lego_bricks_challenge/.cache/sessions/.part_91  
  inflating: lego_bricks_challenge/.cache/thumbnails/.part_179  
  inflating: lego_bricks_challenge/.cache/thumbnails/.part_90  
  inflating: lego_bricks_challenge/.cache/thumbnails/.meta/.part_194  
  inflating: lego_bricks_challenge/.cache/thumbnails/.meta/.part_201  
  inflating: lego_bricks_challenge/.cache/thumbnails/.meta/.part_226  
  inflating: lego_bricks_challenge/.cache/thumbnails/large/.part_158  
 extracting: lego_bricks_challenge/.cache/thumbnails/large/.part_31  
  inflating: lego_bricks_challenge/.cache/thumbnails/large/.part_69  
  inflating: lego_bricks_challenge/.cache/thumbnails/large/Dockerfile  
  inflating: lego_bricks_challenge/.cache/thumbnails/large/.normal/.part_151  
  inflating: lego_bricks_challenge/.cache/thumbnails/large/.normal/.part_183  
  inflating: lego_bricks_challenge/.cache/thumbnails/large/.normal/.part_207  
  inflating: lego_bricks_challenge/.cache/thumbnails/large/.normal/.part_84  
  inflating: lego_bricks_challenge/.config/.part_117
... etc
```

There are many `.part_<index>` files in different directories. I assemble the pieces in order like this:
```shell
find . -type f -name ".part_*" | sed 's/.*\.part_//' | sort -n | while read n; do  
find . -type f -name ".part_$n" -exec cat {} \;  
done > combined.txt
```

My `combined.txt` file contained the flag:
```shell
cat combined.txt  
bronco{3ve4yth1ng_1s_aw3s0me}64l2IAANevPLlEIjS0X7SMcHlV7tnsBiYVoCf5E9c6KhpdXH5JFKNaj85TbXUOCXPUvNWfN1jeJp4duIDm0wJ6dI7G4jsxVtfXMyElAz2FBg5tw9bw4n3y8XaNSftGf8UcjkmZxxZ2PpTV2jOWhTTzml6GSsuscvf3HcuA2kxGelRxXEI9KyeuSpgvzOR5PIxVg5oRRKJMv647SbMBBNRyUpkatbryvVWJrB6zIcoDTJ2HHKIzvgv0V6qM4EIQ3Kz92iwmZi9tZCr6bmTfPTkwMM7AA3t2q4M1lF09CAP6irEeHZVWUSZuHax5Dz8pQQOdNPGVM8Ezop9lESycrXPUMGBWTvNgvvz42OM73QCurtamNGFzH8ab65vNoteug86ijpTgI7nsXZ1Sf8bTMIaaZYL2iUfjqEb1XaoY3QYndD6PLURq4eqNrcmZpqplB4iC5zX4y4OuGtCktMHkXIFh7AAL9o4vq9sNtAe75Orn1nnVemC15qB9lKROTPSvqKFZM
```

`bronco{3ve4yth1ng_1s_aw3s0me}`

___

## Magic Ways

**Category**: forensics

**Author**: .tidalw

**Description**: I rubbed this lamp and out came `challenge.png`. But, I can't open it for some reason. Help me out and maybe I'll grant you a wish (flag).

`challenge.png` is corrupted:
```shell
file challenge.png  
challenge.png: data

xxd challenge.png | head  
00000000: dead beef 0000 0000 0000 000d 4948 4452  ............IHDR  
00000010: 0000 01f4 0000 0000 0802 0000 0000 0000  ................  
00000020: 0000 0019 3d49 4441 5478 9ced dd77 7c14  ....=IDATx...w|.  
00000030: 65c2 07f0 996d d9be e9d9 f464 494f 2801  e....m.....dIO(.  
00000040: 02a1 4815 90d0 e5c4 7248 4445 bd17 1405  ..H.....rHDE....  
00000050: 3cf5 540e 2c94 83f3 446c 14e1 ce82 8880  <.T.,...Dl......  
00000060: 1411 2922 4a0b 484f 420a 84f4 decb 6e76  ..)"J.HOB.....nv  
00000070: b3ed fdcc 4e5c 361b 201b 0c20 cffe be1f  ....N\6. .. ....  
00000080: fe80 cd6e 6666 9f67 7ef3 cc53 063a e8e9  ...nff.g~..S.:..  
00000090: 140a 0000 c8c2 b9db 3b00 0000 5d0f e10e  ........;...]...
```

I ran a python script which will find the intact zlib compressed data (`78 9c`), decompress it and compute `height = 200` (with `width = 500` from the original IHDR), and finally write a new valid PNG file called `repaired.png`:
```python
python3  
Python 3.13.5 (main, Jun 13 2026, 14:18:01) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import struct, zlib  
...  
... def crc32(data):  
...     return zlib.crc32(data) & 0xffffffff  
...  
... def make_chunk(typ, data):  
...     chunk = struct.pack('>I', len(data)) + typ + data  
...     chunk += struct.pack('>I', crc32(typ + data))  
...     return chunk  
...  
... # read corrupted file  
... with open('challenge.png', 'rb') as f:  
...     raw = f.read()  
...  
... # find the zlib stream (starts with 0x78 0x9c)  
... start = raw.find(b'\x78\x9c')  
... if start == -1:  
...     raise RuntimeError('zlib header not found')  
...  
... # remove garbage before the zlib stream  
... compressed = raw[start:]  
...  
... # decompress to get the raw pixel data  
... d = zlib.decompressobj()  
... try:  
...     raw_data = d.decompress(compressed) + d.flush()  
... except:  
...     # if it fails trim trailing junk incrementally  
...     for i in range(len(compressed), 0, -1):  
...         try:  
...             raw_data = zlib.decompress(compressed[:i])  
...             print(f'Stripped {len(compressed)-i} trailing bytes')  
...             break  
...         except:  
...             continue  
...     else:  
...         raise RuntimeError('Cannot decompress')  
...  
... # width is known from IHDR  
... width = 500  
... row_bytes = width * 3 + 1
... height, rem = divmod(len(raw_data), row_bytes)  
... if rem:  
...     print(f'Warning: {rem} extra bytes, height may be approximate')  
... print(f'Recovered size: {width} x {height}')  
...  
... # build a new PNG  
... sig = b'\x89PNG\r\n\x1a\n'  
... ihdr = make_chunk(b'IHDR', struct.pack('>IIBBBBB', width, height, 8, 2, 0, 0, 0))  
... idat = make_chunk(b'IDAT', compressed)
... iend = make_chunk(b'IEND', b'')  
...  
... with open('repaired.png', 'wb') as f:  
...     f.write(sig + ihdr + idat + iend)  
...  
... print('repaired.png created successfully')  
...  
Recovered size: 500 x 200  
repaired.png created successfully
```

`repaired.png`:

![Alt text](/images/repaired847688.png)

`bronco{wh4t_ar3_mag1c_byt3s}`

___

## EX-BOOST

**Category**: forensics

**Author**: blunderous_wonders

**Description**: Lost Judgment has been one of my favorite visuals in the Yakuza Series, especially the styles and their colors. As much as Fully Baked loves Boxer, I much prefer the RGB trifecta.

But what's with the heat bar amount on each style? Are they trying to tell me something?

format: `bronco{}`, if you find multiple parts, no spaces.

I get 3 images:
```shell
file Crane.png  
Crane.png: PNG image data, 154 x 136, 8-bit/color RGBA, non-interlaced  

file Tiger.png  
Tiger.png: PNG image data, 164 x 146, 8-bit/color RGBA, non-interlaced  

file Snake.png  
Snake.png: PNG image data, 111 x 96, 8-bit/color RGBA, non-interlaced
```

Original images:

![Alt text](/images/Crane.png)

![Alt text](/images/Snake.png)

![Alt text](/images/Tiger.png)

I put each image into [https://www.aperisolve.com/](https://www.aperisolve.com/) and got text from each image:

From `Crane.png`:

![Alt text](/images/crane_blue_bit_4.png)

From `Snake.png`:

![Alt text](/images/snake_green_bit_2.png)

From `Tiger.png`:

![Alt text](/images/tiger_red_bit_0.png)

`bronco{F33LTH3H34T}`

___

## Static Image

**Category**: forensics

**Author**: shwhale

**Description**: It's not often that you do static analysis outside of Reversing, is it?

I get the challenge file `static.mp4`:
```shell
file static.mp4  
static.mp4: ISO Media, MP4 Base Media v1 [ISO 14496-12:2003]
```

It is a 24 second video. When I play the video it mostly looks like static. However there are small differences in the static which spells out the flag. I can see it when I play the video. I dont know of a tool that would extract the characters automatically or filter out the noise so I manually typed out the flag.

`bronco{n0w_th4ts_dyn4m1c}`

___

## Suspicious Remix

**Category**: forensics

**Author**: blunderous_wonders

**Description**: So there's this guy SG who just sent me a copy of their new remix. Knowing what they are like with their remixes, I don't trust it, in more ways than one. Whatever remix, I always feel like they modified the audio specifically for me somehow. Maybe you should have a listen.

Wrap the secret you find in `bronco{}`.

 I get the file `sg_remix.wav`:
 ```shell
file sg_remix.wav  
sg_remix.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 48000 Hz
 ```

I opened the file in audacity and looked at the spectrogram view:

![Alt text](/images/sg_remix_spectro.png)

I can make out the text: `the flag is the release year of the song ??? ?????? ??? ????`. I dont know what the ending says, but the song is "Never Gonna Give You Up" by Rick Astley which was released on July 27, 1987.

`bronco{1987}`

___

## Suspicious Remix 2

**Category**: forensics

**Author**: blunderous_wonders

**Description**: I just listened to the first one, I'm sorry for making you listen to that.

But now they sent me another one. I guess that they didn't directly embed the message in the audio, but I still think they put something in there somehow.

They told me it's supposedly more _hide_-n, whatever that means. I keep seeing with a command prompt, typing the steg command...

Also they told me there's a password of this, apparently it's in this image?

They keep doing this to me, this is their 2nd Remix! I'm so sorry for putting you through this. I promise I'll leave you alone after this.

I get two challenge files:
```shell
file sg_remix2.wav  
sg_remix2.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 48000 Hz  

file tolerate_this.png  
tolerate_this.png: PNG image data, 500 x 409, 8-bit/color RGBA, non-interlaced
```

`tolerate_this.png`:

![Alt text](/images/tolerate_this.png)

I put the image into [https://www.aperisolve.com/](https://www.aperisolve.com/) and I find this:

![Alt text](/images/tolerate_this_red_bit_2.png)

I used `steghide` with the password `1988` to extract the flag file: 
```shell
steghide extract -sf sg_remix2.wav -p 1988  
wrote extracted data to "got_u_so_good.txt".  

cat got_u_so_good.txt  
bronco{7h3y_g07_y0u_4g4in_didn'7_7h3y?}
```

`bronco{7h3y_g07_y0u_4g4in_didn'7_7h3y?}`
