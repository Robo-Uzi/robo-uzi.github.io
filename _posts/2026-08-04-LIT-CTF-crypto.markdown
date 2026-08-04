---
layout: post
title:  "Crypto Challenges"
date:   2026-08-04 18:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /LIT-CTF-2026-crypto/
---
* TOC
{:toc}
## double disguise

**Author**: modulus998244353

**Category**: crypto

**Description**: Oopps, I ran accidentally ran an encryption program on my flag. Now I only have: `ZmN+aX5sUWwbTEt1fRpYRk51SV9aVw==` Can you help me the original flag?

In cyberchef I did the recipe `from base64` then `XOR Brute Force`. [cyberchef link](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)XOR_Brute_Force(1,100,0,'Standard',false,true,false,'')&input=Wm1OK2FYNXNVV3diVEV0MWZScFlSazUxU1Y5YVZ3PT0)

`LITCTF{F1fa_W0rld_cup}`

___

## any secrets could instantly `[be]` imparted

**Author**: joshadoodle

**Category**: crypto

**Description**: Cody and Tiger have just had an argument, where Cody has shared Tiger's deepest, darkest secret. The following was one of their recorded interactions during their argument.
```shell
Tiger: Et tu, Brute?  
Cody: HEP?PBw=O?EE)fQh-qo)_0/o0n)casks]y.
```
Note that the flag will be in the format `LITCTF{...}`.

I put `HEP?PBw=O?EE)fQh-qo)_0/o0n)casks]y.` into cyberchef and did the recipe `ROT47 Brute Force`. 

[cyberchef link](https://gchq.github.io/CyberChef/#recipe=ROT47_Brute_Force(100,0,true,'')&input=SEVQP1BCdz1PP0VFKWZRaC1xbylfMC9vMG4pY2Fza3NdeS4&oenc=65001)

`LITCTF{ASCII-jUl1us-c43s4r-gewowa}`

___

## faulty image 1 haha

**Author**: Ninjaprime

**Category**: crypto

**Description**: haha Someone tampered with an image haha, and now we can't view it haha! Help the LIT team recover the image haha!

I get the challenge file:
```shell
file corrupt.png  
corrupt.png: PNG image data, 400 x 150, 8-bit/color RGB, non-interlaced
```

The image is corrupted by inserting `haha` randomly:
```shell
xxd corrupt.png | tail  
00000860: 7083 6001 7083 6001 7083 6001 7083 6001  p.`.p.`.p.`.p.`.  
00000870: 7083 6001 7083 6001 7083 6001 7083 6001  p.`.p.`.p.`.p.`.  
00000880: 7083 6001 6861 6861 7083 6001 7083 6001  p.`.hahap.`.p.`.  
00000890: 7083 6001 7083 6001 7083 6001 7083 6001  p.`.p.`.p.`.p.`.  
000008a0: 7083 6001 7083 6001 7083 6001 7083 6001  p.`.p.`.p.`.p.`.  
000008b0: 7083 6001 7083 6001 7083 6001 7083 6001  p.`.p.`.p.`.p.`.  
000008c0: 7083 6001 7083 6001 7083 6001 7083 6001  p.`.p.`.p.`.p.`.  
000008d0: 7083 6001 7083 6001 7083 6001 7083 6001  p.`.p.`.p.`.p.`.  
000008e0: 102f fe07 b3de c9e1 da9e 37be 6861 6861  ./........7.haha  
000008f0: 0000 0000 4945 4e44 ae42 6082 6861 6861  ....IEND.B`.haha
```

I put the corrupted image into cyberchef and did the recipe `Find / Replace "haha"` with nothing. Then I get the image containing the flag:

![Alt text](/images/hahahahahahahafixed.png)

[cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Regex','string':'haha'%7D,'',true,false,true,false)Render_Image('Raw')&input=iVBORw0KGgoAAAANSUhEUgAAAZAAAACWCAIAAAB/80kyAAAIa0lEQVR4nO3abWxddR3A8d/v3D6vvX26d103t2qUxT2RqEwTWIYQZWJUNoywjcQH4IURFxONb4AXJpolxncaMGhhaGExviBmGBEMGtAZiRqcQALEDQdMYpbhJiv0eW1v29v2np/533vbbbf3tr1Zify67yd9cdqe/u/5n%2BR8%2Bz/nXt2%2Be68AgAfR//sAAGC5CBYANwgWADcIFgA3CBYANwgWADcIFgA3aGFoYQgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADdoYWhhCBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYANwgWADcIFgA3CBYAN2hhaGEIFgA3CBYANwgWADeu9mDpvk7d07Ziw62t1ZtbpWEVndUrmJHuadO9ne/CMeHqVSPvVdHhnvj%2B/yz8ie5o0t2t4fsPNMiZKRGxv49KY6Q3JGUqtunYHh%2BUkdmyu%2BnBtJzNFkazaGFoYVcn7NkLurM5fvBs6Ws3RHowrWsiy8T2y36Zipd7zHd32XOjEls4nr2duqOpZApFdar709qSkPrI/jhsr08sPvElLDpaCMc1jXpru82aJiR%2BakjezOrmRr0jJcOz4TycmbI/DFdoYWhhet3ijNpq9LYOrVHLxvbYgIzlwu9K5ripXrtq7aXx%2BaHsmQvRDzbZbwermAvgNFiV2MkJOzlRvCoe6g0X5OZGuSEZ//i8zJhuadQDKfvp2wt3C3vemZrfvii2kh/op9vk9FT87GhhaGEF/WSrfqrNnh5a7sG1JOzYaHjRQ%2BvsREZ2NJXdS3cl5Vw2/usFSSaib61fmJiqLDma7s/PemjWOmuje7viH/5Xkgn7y4g9P7bMGenX18mfL8T/ngzt29NuTwyEOd7bddkcz2ZtaGFoYe6fQaUTC1yhVXHzclOr/X5IZsLlYacmZXBWElrVANF3NkiqNmw1RNH979MtjXY8rBTs%2BLhuaSzuc7jn4v6XbM/T65NSH0X3dUt9FD/SVyhX8Vdf69L8ha13pPS6ZnthzP4WfqtoYWhh6%2Bokl7%2BqWxJ6T1d0qFsPpCse5OEe3Z%2BKHtio17foXenowY16Y1hClhmtRCaWNYmwsSaSuvyZSSZkNFdmCp9tj%2B7rjr67oXi0czPSDfV2ejK81ulJvaahsHPJHCudFmAFrYZghWhhaGEL9a3p%2BW/t1wPlr9uFf7i1yd7IX4fHx4uX6Icb7Z8T0pIo3vWM5sL28tjzozIdh7VMNi7%2B%2BfyvnhzUW9plU7201djL4zIZS870rrTe3RVuYEX0Cx1yIhP/pFdezUhNhdrWqL0waGFoYRY/3KtfTNmx0fihXr0pf8%2B7YLQS8RMD0aGQoeib3fab/A7JGtnaFPp4T5d01s6PL5lw/PEjfbqv89IZ2fmsbsufnx1rLp6Qy%2BdY/py8Malbyy8zgas0WFVMIqFhBZH/krW1YuFoYWhhJ%2BG6%2BkemcEHK9qbC2moJ1S3gREZm7eXx8DwofzNVYI/225E%2B2dkcxvtgg72SCT98bVIqPTEzkXPZ8OApZ8WNwnJpwWglos932JH%2B%2BEdv2aP9cm1%2BmmZyfjr08aVxvTN1cZAX82hhaGE3if0zJU/Zw3Or61r0G93SXiOz1dzo5SwcNrBCVkWw%2BmZkQ11xWyU8Wa8kZ2EFkf%2BSvhk7NaEfyt/gjMyG66o1oR21YbE2NrewSs4ttS6NVGNU7S1nUK/hmU59OOF6e6dE%2BVC%2BaGFoYT5RXIDMr6q0cg1zVmzZ7GUVKDNaifV1djJfw5MZ3bYmbBwbDaun8M5DRrvnVlg5C4u1cvSjzfaLPnu4116bCDlbtrCGPXVFT%2BiA1RYse25Ub20vXPP6keaKt1SLD3I8o7d12r9oYWhhwtVlpybDOPnRwkOxgslY1oUs6sfKrGKWkK7VzY3xz9%2BJbu8MPWqICneg8v6GUNvwVl1WtxfuuZqqXr4tGK1U30x4q1REehpkOOygn%2Bsopm1Tg/XO/ckiS6GN9bo1PMvTnc12PGhhaGG0b3Hx9xa88Qqs8ncJExod6i5s2pnsIm/V2YmMpmujb6%2B38VjGc8XHNMsXicRir2SifR1x/g1%2Be2ZED6aja5uKH2sovMqTg9FX1tpYLnwwoqrbIhH9Uip%2BeljOT9s70/qJFjs6aGFoYawH0tGupOUs/lUY3343GB1M666kvVn14AtHKxE/PhDln0mF7cfCPakdHY72p/TGVpuxcLu35Es8NRQdSOnNbXYua0eHl57vV7vsZ2%2Bviv%2BGeG/R7bv3ylUsPF3O5OxPI%2BGjRvloYWhhz0NU8bc7m/XjLYVte3Gs8BGkFTuwKxj8XT2wiiKJvt8TP3Dxk1x6S5s0JfgcFlbQ1R6sAt3WpJ9pD8uTS95tRFX0y2slNjtSZokHrBSCBcANHjMAcINgAXCDYAFwg2ABcINgAWhhaGFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABaGFoYXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFoYWhhcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgAXCDYAFwg2ABcINgARAv/gez3snh2p43vmhhaGEAAAAASUVORK5CYIJoYWhh&oeol=VT)

`LITCTF{y0u_f1x3d_m3_85hf91j!}`

___

## faulty image 2

**Author**: Ninjaprime

**Category**: crypto

**Description**: An anonymous user challenged the LIT team to find their identity, but they only sent us this image. Can you help us reveal them?

I get the challenge image:

![Alt text](/images/corrupt2.png)

I find an XOR key in the metadata:
```shell
exiftool corrupt2.png  
ExifTool Version Number         : 13.25  
File Name                       : corrupt2.png  
Directory                       : .  
File Size                       : 1444 bytes  
File Modification Date/Time     : 2026:08:02 20:51:06-04:00  
File Access Date/Time           : 2026:08:02 20:51:06-04:00  
File Inode Change Date/Time     : 2026:08:02 20:51:09-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 400  
Image Height                    : 150  
Bit Depth                       : 8  
Color Type                      : RGB  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Warning                         : [minor] Text/EXIF chunk(s) found after PNG IDAT (may be ignored by some readers)  
XOR-Key                         : 636f64657469676572  
Image Size                      : 400x150  
Megapixels                      : 0.060

echo '636f64657469676572' | xxd -r -p  
codetiger
```

The XOR key is `codetiger`. The key was found in a `tEXt` chunk. Theres also a second non standard chunk right after it named `tIGr` holding 60 bytes of data:
```shell
xxd corrupt2.png | grep -A 5 -B 5 tIGr  
00000500: 1043 b080 1882 05c4 102c 2086 6001 3104  .C......., .`.1.  
00000510: 0b88 2158 400c c102 6208 1610 43b0 8022  ..!X@...b...C.."  
00000520: c5df 4d82 7f1f 4102 f39f 0000 001a 7445  ..M...A.......tE  
00000530: 5874 584f 522d 4b65 7900 3633 3666 3634  XtXOR-Key.636f64  
00000540: 3635 3734 3639 3637 3635 3732 828b 35de  657469676572..5.  
00000550: 0000 003c 7449 4772 0000 0020 2f26 3026  ...<tIGr... /&0&  
00000560: 202f 1c06 4207 5c10 5413 5a15 3a05 571c   /..B.\.T.Z.:.W.  
00000570: 3b0d 471b 543a 0a53 1d45 1874 8091 47b6  ;.G.T:.S.E.t..G.  
00000580: 7271 76b3 9aa8 fd9a e7aa 89e3 fb82 df5a  rqv............Z  
00000590: 6d57 9d35 fd25 816e 0000 0000 4945 4e44  mW.5.%.n....IEND  
000005a0: ae42 6082                                .B`.
```

I ran this python to get the flag:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import struct  
...  
... path = "corrupt2.png"  
... data = open(path, "rb").read()  
...  
... # walk PNG chunks  
... i = 8  
... chunks = {}  
... while i < len(data):  
...     length, ctype = struct.unpack(">I4s", data[i:i+8])  
...     chunks[ctype.decode()] = data[i+8:i+8+length]  
...     i += 12 + length  
...  
... key = bytes.fromhex(chunks["tEXt"].split(b"\x00")[1].decode())  
... blob = chunks["tIGr"]  
...  
... for phase in range(len(key)):  
...     pt = bytes(b ^ key[(i+phase) % len(key)] for i, b in enumerate(blob))  
...     if b"LITCTF{" in pt:  
...         print(pt[pt.index(b"LITCTF{"): pt.index(b"}", pt.index(b"LITCTF{"))+1].decode())  
...         break  
...  
LITCTF{c0d3t1g3r_w4s_h3r3_x0r!}
```

`LITCTF{c0d3t1g3r_w4s_h3r3_x0r!}`

___

## faulty image 3

**Author**: Ninjaprime

**Category**: crypto

**Description**: A hacker sent us another suspicious image. We don't know what to do with it. Maybe you do? It's pretty colorful though...

I get the challenge image:

![Alt text](/images/corrupt3.png)

Near the top you can see some slight difference in the border. I make a script to look at the top of the image and pull pixels from the red channel which are abnormal to the regular color of the border (the green and blue channels contained decoy data):
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> from PIL import Image  
...  
... def get_flag(image_path):  
...     img = Image.open(image_path).convert('RGB')  
...     pixels = img.load()  
...     width, height = img.size  
...  
...     base_color = pixels[width // 2, 0]  
...  
...     anomalous_pixels = []  
...     last_color = None  
...  
...     for y in range(10):  
...         for x in range(width):  
...             r, g, b = pixels[x, y]  
...             color = (r, g, b)  
...  
...             # ignore inner black background  
...             is_bg = (r < 50 and g < 50 and b < 50)  
...  
...             if color != base_color and not is_bg:  
...                 if color != last_color:  
...                     anomalous_pixels.append(color)  
...                     last_color = color  
...             else:  
...                 last_color = None  
...  
...     # extract only the red value and convert to ASCII  
...     flag_chars = [chr(r) for r, g, b in anomalous_pixels]  
...  
...     flag = "".join(flag_chars)  
...     print(f"\nFlag: {flag}\n")  
...  
... get_flag("corrupt3.png")  
...  
  
Flag: LITCTF{p1x3l_p3rf3ct_98xzfq9!}
```

`LITCTF{p1x3l_p3rf3ct_98xzfq9!}`

___

## faulty image 4

**Author**: Ninjaprime

**Category**: crypto

**Description**: Something's off about this image. The flag you see isn't the flag you want... again. This time, the hacker corrupted some of the pixels, separating them from encoding information together as a whole.

I get the challenge image `corrupt4.png`:

![Alt text](/images/corrupt4.png)

I was given this hint:
```shell
Some pixels have unusually small blue values, almost like they've been clipped to 4 bits. Which pixels exactly?
```

So some the blue values are clipped to 4 bits (values from 0 to 15). Any of those pixels holds a 4 bit nibble in its blue channel. Pairing them up yields full bytes that spell out the hidden flag:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> from PIL import Image  
...  
... img = Image.open('corrupt4.png')  
... width, height = img.size  
... pixels = img.load()  
...  
... target_pixels = []  
...  
... # collect pixels where blue is clipped to 4 bits  
... for y in range(height):  
...     for x in range(width):  
...         r, g, b = pixels[x, y]  
...         if b < 16:  
...             target_pixels.append((x, y, r, g, b))  
...  
... print(f"Matching pixels found: {len(target_pixels)}")  
...  
... # extract the 4 bit blue values  
... blue_vals = [p[4] for p in target_pixels]  
...  
... # combine pairs of 4 bit nibbles into 8 bit bytes  
... flag_bytes = bytearray()  
... for i in range(0, len(blue_vals) - 1, 2):  
...     combined_byte = (blue_vals[i] << 4) | blue_vals[i+1]  
...     flag_bytes.append(combined_byte)  
...  
... try:  
...     print("Flag:", flag_bytes.decode('utf-8'))  
... except Exception as e:  
...     print("Raw Bytes:", flag_bytes)  
...  
Matching pixels found: 54  
Flag: LITCTF{pr1m3_n1bbl3s_77zq!}
```

`LITCTF{pr1m3_n1bbl3s_77zq!}`
