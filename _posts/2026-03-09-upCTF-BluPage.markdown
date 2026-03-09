---
layout: post
title:  "BluPage"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-BluPage/
---

**Title**: BluPage

**Category**: misc

**Author**: lrc

**Description**: A big friend of mine tried to work as a front-end dev, shipped a “minimal” page and forgot that the web isn’t just what you see.

Expires in 10 minutes  
[http://46.225.117.62:30013](http://46.225.117.62:30013)

I go to the website and on `view-source:http://46.225.117.62:30013/` I see this:
```html
<!doctype html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <title>forwebdevs — tips, snippets, UI puzzles</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="prefetch" href="/assets/artifacts.zip" as="fetch" crossorigin>
    <link rel="stylesheet" href="/static/style.css">
</head>

<body>
    <main class="card">
        <h1>BluPage</h1>
        <p>Practical tips, small snippets, and UI puzzles.</p>
        <a class="btn" href="#" aria-disabled="true">Soon</a>
        <hr class="hr">
        <p>Some things reveal themselves to those who look a little closer.</p>
    </main>
    <script src="/static/touch.js"></script>
</body>

</html>
```

I download the `zip` and get two `png` files:
```shell
curl http://46.225.117.62:30013/assets/artifacts.zip -o artifacts.zip  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  6988  100  6988    0     0   5090      0  0:00:01  0:00:01 --:--:--  5093

file artifacts.zip  
artifacts.zip: Zip archive data, made by v2.0 UNIX, extract using at least v2.0, last modified Mar 08 2026 17:20:08, uncompressed size 0, method=store

unzip artifacts.zip  
Archive:  artifacts.zip  
creating: artifacts/  
inflating: artifacts/f_right.png  
inflating: artifacts/f_left.png

file artifacts/*  
artifacts/f_left.png:  data  
artifacts/f_right.png: PNG image data, 640 x 200, 8-bit/color RGBA, non-interlaced
```
One png is malformed and the other is completely black.

On `view-source:http://46.225.117.62:30013/static/touch.js` I get this hint:
```html
console.log("Don't forget: LSB, 32bits.");
```

Looking at the headers for each png I notice `f_left.png` is malformed:
```shell
xxd artifacts/f_left.png | head  
00000000: 8950 4e58 0d0a 1a0a 0000 000d 4948 4452  .PNX........IHDR  
00000010: 0000 0280 0000 00c8 0802 0000 00f7 3bc7  ..............;.  
00000020: 5c00 001b b249 4441 5478 9ced dd79 5814  \....IDATx...yX.  
00000030: f7e1 c7f1 8545 2e01 0151 a46a 8207 0a8a  .....E...Q.j....  
00000040: a2a2 4623 08c1 c4aa 152c 2278 559f 478d  ..F#.....,"xU.G.  
00000050: f148 4213 8cb1 49ad 67d5 78a0 55ab 8f4a  .HB...I.g.x.U..J  
00000060: 0d18 452d 1af0 8a51 bc8f 18ad 47c0 40f0  ..E-...Q....G.@.  
00000070: 563c f011 5450 d045 cedf 1fd3 673a bf05  V<..TP.E....g:..  
00000080: 96e5 fcb2 f07e fd35 333b 33fb 4ddc e533  .....~.53;3.M..3  
00000090: fb3d 8db2 b2b2 5400 00a0 7619 8b2e 0000  .=....T...v.....  

xxd artifacts/f_right.png | head  
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR  
00000010: 0000 0280 0000 00c8 0806 0000 0078 5950  .............xYP  
00000020: 0b00 0003 5849 4441 5478 daed d639 0e80  ....XIDATx...9..  
00000030: 300c 0041 9bff ffd9 3429 1032 2155 84c4  0..A....4).2!U..  
00000040: 4c07 0472 d06c 4644 c556 39a6 cc71 5d97  L..r.lFD.V9..q].  
00000050: fb71 7956 b777 a219 df8d 997d 2f9b f5d4  .qyV.w.....}/...  
00000060: e4fb d1ac b51b fb36 ffea feea e17c b259  .......6.....|.Y  
00000070: 732c ee6b f3ef 0500 3e4f 2100 00fc cce1  s,.k....>O!.....  
00000080: 0800 0004 2000 0002 1000 0001 0800 8000  .... ...........  
00000090: 0400 4000 0200 2000 0100 1080 0000 0840  ..@... ........@
```

I change `PNX` to `PNG` and now the image loads. I get the first part of the flag:

![Alt text](/images/f_left_fixed.png)

For `f_right.png` I do this cyberchef recipe and get the second part of the flag `4r3_sn34ky}`: [cyberchef](https://gchq.github.io/CyberChef/#recipe=Extract_LSB('B','','','','Row',0)&input=iVBORw0KGgoAAAANSUhEUgAAAoAAAADICAYAAAB4WVALAAADWElEQVR42u3WOQ6AMAwAQZv//9k0KRAyIVWExEwHBHLQbEZExVY5psxxXZf7cXlWt3eiGd%2BNmX0vm/XU5PvRrLUb%2Bzb/6v7q4XyyWXMs7mvz7wUAPk8hAAD8zOEIAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAAABCACAAAQAQAACAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAAABCACAAAQAQAACAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAAABCACAAAQAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAAABCACAAAQAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAAABCAAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAAABCAAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAAABCAAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAEAAAgAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAAAIQAEAAAgAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAgAAEAEAAAgAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAABCAAgAAEAEAAAgAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAAAQgAgAAEAEAAAgAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAAAQgAgAAEAEAAAgAgAAEAEIAAAAhAAAAEIAAAAhAAAAEIAIAABABAAAIAIAABABCAAAAIQAAAAQgAgAAEAEAAAgAgAAEAEIAAAAhAAAAEIAAAG53N6RyNJTHx9QAAAABJRU5ErkJggg&oeol=VT)

Python script which also recovers the second part of the flag:
```python
from PIL import Image

def extract_lsb_blue(img_path):
    img = Image.open(img_path)
    bits = []

    for r, g, b, *rest in img.getdata():
        bits.append(str(b & 1))

    # convert every 8 bits to a byte
    data = bytes(int("".join(bits[i:i+8]), 2) for i in range(0, len(bits), 8))
    return data

hidden_data = extract_lsb_blue("artifacts/f_right.png")
print(hidden_data.decode(errors="ignore"))
```

Output:
```shell
python3 solve5.py  
/home/user/Desktop/ctf/upctf/solve5.py:7: DeprecationWarning: Image.Image.getdata is deprecated and will be removed in Pillow 14 (2027-10-15). Use get_flattened_data instead.  
for r, g, b, *rest in img.getdata():  
  
4r3_sn34ky}
```

`upCTF{PNG_hdrs_4r3_sn34ky}`