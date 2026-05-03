---
layout: post
title:  "Capybara in Nightmare Land"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-capybara-in-nightmare-land/
---

**Author**: [@van_1pi](https://t.me/van_1pi)

**Category**: Steganography

**Description**: _A capybara from KubGTU fell asleep during an information security lecture and found herself in a strange nightmare..._ _In this dream, she left a secret message. Can you find it?_ Hint: Not everything is as it seems. Look deeper.

I get the challenge file:
```shell
file capybara_nightmare.png  
capybara_nightmare.png: PNG image data, 1376 x 768, 8-bit/color RGB, non-interlaced
```

Image:

![Alt text](/images/capybara_nightmare.png)

I run `binwalk`:
```shell
binwalk -e capybara_nightmare.png  
  
DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
41            0x29            Zlib compressed data, default compression  
1352062       0x14A17E        Zip archive data, at least v2.0 to extract, compressed size: 385, uncompressed size: 973, name: README.txt  
1352487       0x14A327        Zip archive data, at least v2.0 to extract, compressed size: 30, uncompressed size: 28, name: encrypted_flag.bin  
  
WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

I get `README.txt` and `encrypted_flag.bin`:
```shell
cat README.txt  
  
+--------------------------------------------------------------+  
|           CAPYBARA'S ENCRYPTED SECRET                        |  
+--------------------------------------------------------------+  
|  The flag is XOR encrypted.                                  |  
|  The key is hidden in the original image...                  |  
|  Look closer at the pixels!                                  |  
|                                                              |  
|  Hint: LSB (Least Significant Bit)                           |  
|  Password length: 19 characters                          |  
+--------------------------------------------------------------+  
  
Encrypted flag (hex): 0544053b20384f3a03333a6b3d49334b6f71573e482f09370605004e  
  
Python decrypt example:  
	key = "???"  # Find the key in the image!  
	encrypted = bytes.fromhex("0544053b20384f3a03333a6b3d49334b6f71573e482f09370605004e")  
	flag = ''.join(chr(b ^ ord(key[i % len(key)])) for i, b in enumerate(encrypted))
	
xxd encrypted_flag.bin  
00000000: 0544 053b 2038 4f3a 0333 3a6b 3d49 334b  .D.; 8O:.3:k=I3K  
00000010: 6f71 573e 482f 0937 0605 004e            oqW>H/.7...N
```

I find the key with `zsteg`:
```shell
zsteg -a capybara_nightmare.png | head -n 20  
[?] 645 bytes of extra data after image end (IEND), offset = 0x14a17e  
extradata:0         .. file: Zip archive data, made by v2.0, extract using at least v2.0, last modified Mar 26 2026 09:58:58, uncompressed size 973, method=deflate  
00000000: 50 4b 03 04 14 00 00 00  08 00 5d 4f 7a 5c f7 db  |PK........]Oz\..|  
00000010: 65 c2 81 01 00 00 cd 03  00 00 0a 00 00 00 52 45  |e.............RE|  
00000020: 41 44 4d 45 2e 74 78 74  ad 92 51 6b db 30 14 85  |ADME.txt..Qk.0..|  
00000030: df fd 2b 4e 33 4a 6c ba  1a 35 72 92 26 50 42 92  |..+N3Jl..5r.&PB.|  
00000040: 7a ec 21 6c 21 c9 c3 ca  68 41 b6 af 6d b5 8e 14  |z.!l!...hA..m...|  
00000050: 64 95 25 d0 1f 3f 5b dd  ba 3d ac db 58 76 9e 84  |d.%..?[..=..Xv..|  
00000060: d0 3d fa c4 27 ef ec fc  a8 9c 79 4f f8 91 f9 74  |.=..'.....yO...t|  
00000070: 79 33 9b ae a6 dd 35 e2  0f f3 d5 cd 72 13 5f 63  |y3....5.....r._c|  
00000080: 1d cf 57 f1 06 af e4 c9  fb 1f 04 9b 92 90 57 a2  |..W...........W.|  
00000090: 80 ac f1 e9 e3 0a a4 52  73 d8 59 ca c2 d7 ee fd  |.......Rs.Y.....|  
000000a0: 99 e0 5b c1 03 1d da f9  52 66 19 29 48 05 db 6c  |..[.....Rf.)H..l|  
000000b0: 6a 23 0b a9 44 05 b9 15  05 85 e1 2f 0a 5d c1 42  |j#..D....../.].B|  
000000c0: eb 07 a4 95 ae c9 40 58  37 ba 93 7b aa ea 93 bf  |......@X7..{....|  
000000d0: 24 38 26 ae e0 bd 54 76  8c c5 7a 06 7f 41 a2 b6  |$8&...Tv..z..A..|  
000000e0: 58 cb 42 c9 5c a6 42 59  cc a4 0d fe 54 b0 14 75  |X.B.\.BY....T..u|  
000000f0: fd 45 9b 0c 15 a9 c2 96  63 5c 8c 90 96 c2 88 d4  |.E......c\......|  
imagedata           .. text: "\r\n\n\n\n\t\t\t"  
b1,rgb,lsb,xy       .. text: "N1ghtm4r3_C4py_2026"  
b2,rgb,msb,xy       .. file: zlib compressed data
```

Decrypt the flag with repeating key XOR:
```python
key = "N1ghtm4r3_C4py_2026"
enc_hex = "0544053b20384f3a03333a6b3d49334b6f71573e482f09370605004e"
enc_bytes = bytes.fromhex(enc_hex)
flag = ''.join(chr(enc_bytes[i] ^ ord(key[i % len(key)])) for i in range(len(enc_bytes)))
print(flag)
```

Output:
```shell
python3 solve3.py  
KubSTU{H0ly_M0ly_CapyHaCk1r}
```

`KubSTU{H0ly_M0ly_CapyHaCk1r}`