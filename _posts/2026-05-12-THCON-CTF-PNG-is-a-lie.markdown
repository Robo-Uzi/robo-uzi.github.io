---
layout: post
title:  "PNG is a lie (part 1/2)"
date:   2026-05-12 19:18:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-PNG-is-a-lie/
---

**Category**: Steganography

**Author**: [chepycou](https://www.ar-lacroix.fr/)

**Description**: Some members of the COPS We have found this file lying on a very sus USB stick. Maybe try to find something hidden in it ? Or don't, I mean nobody does steganography anyway.

I get the challenge file:
```shell
file weird_file.thc  
weird_file.thc: Unicode text, UTF-8 text, with very long lines (37486), with no line terminators  

xxd weird_file.thc | wc -l  
2485430  

xxd weird_file.thc | head  
00000000: f09f 918d 684f 76f0 9f91 8e56 71f0 9f91  ....hOv....Vq...  
00000010: 8e44 6749 736d f09f 918e 53f0 9f91 8d55  .DgIsm....S....U  
00000020: 6a66 5350 f09f 918e 52f0 9f91 8e59 6e4f  jfSP....R....YnO  
00000030: 74f0 9f91 8d4f 4565 4d7a f09f 918e 6245  t....OEeMz....bE  
00000040: 786d f09f 918d 78f0 9f91 8e55 5a49 f09f  xm....x....UZI..  
00000050: 918d 4ff0 9f91 8e78 74f0 9f91 8e55 6255  ..O....xt....UbU  
00000060: f09f 918e 766d f09f 918e 5278 5073 f09f  ....vm....RxPs..  
00000070: 918e 6963 4b61 f09f 918d 5068 44f0 9f91  ..icKa....PhD...  
00000080: 8e67 5144 41f0 9f91 8e59 54f0 9f91 8d6f  .gQDA....YT....o  
00000090: 766e 6c55 f09f 918d 4257 4ef0 9f91 8d47  vnlU....BWN....G   

xxd weird_file.thc | tail  
025ecac0: 9f91 8d6d 61f0 9f91 8e75 f09f 918e 4d66  ...ma....u....Mf  
025ecad0: f09f 918e 6279 5251 f09f 918e 4bf0 9f91  ....byRQ....K...  
025ecae0: 8d51 5a4e 65f0 9f91 8e50 59f0 9f91 8e53  .QZNe....PY....S  
025ecaf0: 4343 f09f 918d 6a47 6445 63f0 9f91 8d72  CC....jGdEc....r  
025ecb00: 6a70 f09f 918e 4664 414b f09f 918e 7847  jp....FdAK....xG  
025ecb10: f09f 918e 4456 68f0 9f91 8e62 5547 f09f  ....DVh....bUG..  
025ecb20: 918e 6f5a 4b75 f09f 918d 5049 f09f 918e  ..oZKu....PI....  
025ecb30: 48f0 9f91 8e78 5655 f09f 918e 657a 5966  H....xVU....ezYf  
025ecb40: f09f 918e 4961 f09f 918e 4f6f 71f0 9f91  ....Ia....Ooq...  
025ecb50: 8d52 6356 f09f 918e 68                   .RcV....h
```

I put the file into cyberchef and see they are putting a combination of `👍` and `👎` in the data:
```shell
👍hOv👎Vq👎DgIsm👎S👍UjfSP👎R👎YnOt👍OEeMz👎bExm👍x👎UZI👍O👎xt👎UbU👎... etc
```

I write a short python script to replace `👍` with `1` and `👎` with `0`. This saves the file as an image:
```python
with open('weird_file.thc', 'r', encoding='utf-8') as f:
    text = f.read()

bits = []
for ch in text:
    if ch == '👍':
        bits.append('1')
    elif ch == '👎':
        bits.append('0')
    # ignore any other characters

bit_str = ''.join(bits)
# pad to multiple of 8
if len(bit_str) % 8 != 0:
    bit_str += '0' * (8 - len(bit_str) % 8)

# convert to bytes
data = bytes(int(bit_str[i:i+8], 2) for i in range(0, len(bit_str), 8))

with open('output.png', 'wb') as out:
    out.write(data)
```

`output.png`:

![Alt text](/images/output.png)

`THC{PNG3D}`
