---
layout: post
title:  "Forensics Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TJCTF-2026-forensics/
---
* TOC
{:toc}

## check-the-fine-print

**Author**: andrew

**Category**: forensics

**Description**: what? the details matter? but why can't i see them...

I get the challenge file:
```shell
file logo.png  
logo.png: PNG image data, 150 x 150, 8-bit/color RGBA, non-interlaced
```

`logo.png`:

![Alt text](/images/logo.png)

Running `binwalk` extracts 248 new images:
```shell
binwalk -e ../logo.png  
  
DECIMAL  HEXADECIMAL  DESCRIPTION  
--------------------------------------------------------------------------------  
41       0x29         Zlib compressed data, best compression  
14276    0x37C4       Zip archive data, at least v2.0 to extract, compressed size: 92, uncompressed size: 92, name: 001.png  
14405    0x3845       Zip archive data, at least v2.0 to extract, compressed size: 103, uncompressed size: 103, name: 002.png  
14545    0x38D1       Zip archive data, at least v2.0 to extract, compressed size: 88, uncompressed size: 88, name: 003.png  
14670    0x394E       Zip archive data, at least v2.0 to extract, compressed size: 101, uncompressed size: 101, name: 004.png  
14808    0x39D8       Zip archive data, at least v2.0 to extract, compressed size: 95, uncompressed size: 95, name: 005.png
... etc
```

```shell
cd _logo.png.extracted  

ls  
001.png  015.png  029.png  043.png  057.png  071.png  085.png  099.png  113.png 127.png  141.png  155.png  169.png  183.png  197.png  211.png  225.png  239.png  
002.png  016.png  030.png  044.png  058.png  072.png  086.png  100.png  114.png 128.png  142.png  156.png  170.png  184.png  198.png  212.png  226.png  240.png  
003.png  017.png  031.png  045.png  059.png  073.png  087.png  101.png  115.png 129.png  143.png  157.png  171.png  185.png  199.png  213.png  227.png  241.png  
004.png  018.png  032.png  046.png  060.png  074.png  088.png  102.png  116.png 130.png  144.png  158.png  172.png  186.png  200.png  214.png  228.png  242.png  
005.png  019.png  033.png  047.png  061.png  075.png  089.png  103.png  117.png 131.png  145.png  159.png  173.png  187.png  201.png  215.png  229.png  243.png  
006.png  020.png  034.png  048.png  062.png  076.png  090.png  104.png  118.png 132.png  146.png  160.png  174.png  188.png  202.png  216.png  230.png  244.png  
007.png  021.png  035.png  049.png  063.png  077.png  091.png  105.png  119.png 133.png  147.png  161.png  175.png  189.png  203.png  217.png  231.png  245.png  
008.png  022.png  036.png  050.png  064.png  078.png  092.png  106.png  120.png 134.png  148.png  162.png  176.png  190.png  204.png  218.png  232.png  246.png  
009.png  023.png  037.png  051.png  065.png  079.png  093.png  107.png  121.png 135.png  149.png  163.png  177.png  191.png  205.png  219.png  233.png  247.png  
010.png  024.png  038.png  052.png  066.png  080.png  094.png  108.png  122.png 136.png  150.png  164.png  178.png  192.png  206.png  220.png  234.png  248.png  
011.png  025.png  039.png  053.png  067.png  081.png  095.png  109.png  123.png 137.png  151.png  165.png  179.png  193.png  207.png  221.png  235.png  29  
012.png  026.png  040.png  054.png  068.png  082.png  096.png  110.png  124.png 138.png  152.png  166.png  180.png  194.png  208.png  222.png  236.png  29.zlib  
013.png  027.png  041.png  055.png  069.png  083.png  097.png  111.png  125.png 139.png  153.png  167.png  181.png  195.png  209.png  223.png  237.png  37C4.zip 014.png  028.png  042.png  056.png  070.png  084.png  098.png  112.png  126.png 140.png  154.png  168.png  182.png  196.png  210.png  224.png  238.png
```

Some of the images are corrupt. Some are not. 

Hexdumps from `001.png` and `002.png`:
```shell
xxd 001.png  
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR  
00000010: 0000 0013 0000 0009 0802 0000 005f 7f80  ............._..  
00000020: 6600 0000 2349 4441 5478 9c63 f84f 2e60  f...#IDATx.c.O.`  
00000030: 204d 3503 423d 093a 19c0 80be 760e 2d9d   M5.B=.:....v.-.  
00000040: 0c30 408e 9dc8 0000 67d1 e12d 5c8c 0986  .0@.....g..-\...  
00000050: 0000 0000 4945 4e44 ae42 6082            ....IEND.B`.  

xxd 002.png  
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR  
00000010: 0000 0013 0000 0009 0802 0100 005e bdea  .............^..  
00000020: 5100 0000 2e49 4441 5478 9c63 f84f 2e60  Q....IDATx.c.O.`  
00000030: 2041 290c 90a6 13ae 01ce 26c1 4e4a 7592   A).......&.NJu.  
00000040: ec5a 346d a4e9 44d6 4682 4e06 5440 9a9d  .Z4m..D.F.N.T@..  
00000050: 6800 0029 75d5 39f7 4b34 1600 0000 0049  h..)u.9.K4.....I  
00000060: 454e 44ae 4260 82                        END.B`.
```

`002.png` has its IHDR compression byte set to `0x01`. That is why I cant view it. So taking 248 images, each with a single hidden bit in the IHDR compression byte, 248 bits should equal 31 bytes of ASCII text.

Solve script:
```python
import zipfile, io, struct

with open('../logo.png', 'rb') as f:
    data = f.read()

# find zip start
zip_start = data.index(b'PK\x03\x04')
zip_data = data[zip_start:]

# open the zip
zf = zipfile.ZipFile(io.BytesIO(zip_data))
names = sorted(zf.namelist())

bits = []
for name in names:
    png = zf.read(name)
    # IHDR compression byte is at offset 26
    compression_byte = png[26]
    bits.append(compression_byte & 1)  # 0 or 1

print('Bits:', ''.join(str(b) for b in bits))
print(f'Total bits: {len(bits)}')

# convert bits to bytes/ASCII
chars = []
for i in range(0, len(bits), 8):
    byte_bits = bits[i:i+8]
    if len(byte_bits) == 8:
        val = int(''.join(str(b) for b in byte_bits), 2)
        chars.append(chr(val))

result = ''.join(chars)
print('Decoded:', result)
```

Output:
```shell
python3 solve3.py  
Bits: 01110100011010100110001101110100011001100111101101110111011011110111011101011111011110010110111101110101010111110110000101100011011101000111010101100001011011000110110001111001010111110111001001100101011000010110010001011111011010010111010001111101  
Total bits: 248  
Decoded: tjctf{wow_you_actually_read_it}
```

`tjctf{wow_you_actually_read_it}`

___

## obscure-crusher-1

**Author**: ansh

**Category**: forensics

**Description**: What if I make it so you need 3 keys to unlock the flag...

I get the challenge file:
```shell
file chall.bin  
chall.bin: Mac OS X icon, 256 bytes, "icns" type
```

```shell
xxd chall.bin  
00000000: 6963 6e73 0000 0100 6963 6e73 0100 0000  icns....icns....  
00000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000030: 0000 0000 0000 0100 0000 0000 0000 0000  ................  
00000040: 0000 0000 0000 0000 0000 0000 006e 616d  .............nam  
00000050: 6500 0874 7466 0278 7900 0000 0000 0000  e..ttf.xy.......  
00000060: 0000 0000 0000 0000 0000 0000 005d 0000  .............]..  
00000070: 8000 6c7a 6d61 4b4c 5a4d 415f 4441 5441  ..lzmaKLZMA_DATA  
00000080: 3a1d 090d 0767 0f44 0471 1b0c 1e49 3202  :....g.D.q...I2.  
00000090: 391c 1006 4073 2b45 056c 0b26 180e 0b3e  9...@s+E.l.&...>  
000000a0: 2713 0e5d 0e00 0000 00                   '..].....
```

The file has XOR encrypted data starting at offset `0x81` after `KLZMA_DATA:`:
```shell
1d090d07670f4404711b0c1e493202391c100640732b45056c0b26180e0b3e27130e5d0e00000000
```

As the description mentions, you need 3 keys to decrypt the data. The keys are: `icns`, `ttf`, and `xylzmaK`:
```python
data = bytes.fromhex("1d090d07670f4404711b0c1e493202391c100640732b45056c0b26180e0b3e27130e5d0e00000000")

# repeating XOR key found in the headers
key = b"icns\x01ttf\x02xylzmaK"

flag = ""
for i in range(len(data)):
    flag += chr(data[i] ^ key[i % len(key)])

print(f"Flag: {flag}")
```

Output:
```shell
python3 solve5.py  
Flag: tjctf{0bscur3_crush3r_1cns_ttf_lzm3}ttf
```

Or use cyberchef: [cyber link](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')XOR(%7B'option':'Hex','string':'69636e73017474660278796c7a6d614b'%7D,'Standard',false)&input=MWQwOTBkMDc2NzBmNDQwNDcxMWIwYzFlNDkzMjAyMzkxYzEwMDY0MDczMmI0NTA1NmMwYjI2MTgwZTBiM2UyNzEzMGU1ZDBlMDAwMDAwMDA)

`tjctf{0bscur3_crush3r_1cns_ttf_lzm3}`

___

## skeleton

**Author**: tmm

**Category**: forensics

**Description**: I zipped up a picture of the flag, but I forgot the password. Luckily, I saved the zip2john hash. Can you recover the image?

I get the file `hash.txt`:
```shell
flag.zip/flag.png:$pkzip2$1*1*2*0*12c*120*c8a6617a*0*26*0*12c*c8a6*81bd*36bee62e49e2b2c41f6260bdc2e5fdd8cabd38956eb51f1d8a48c8f6228fd7392a8c53f3199068e3017e11c65e32cd55ea33033ab8b2fb52c4f86373098af1732591290e5c99a2a74239243b67108f232def15a73aac1537e75a593abe81fb3a8b0338afeb00835c67f8a31896a5f73facd1f481fd5ebc8882b5b183819f9b71c89506b3ae7d17bc07ab187ece8413a88af072018ccdc8a2db425082cec0715fd5aa3b3c47bb4f5c93b397154eb2212ffd593d0e4e614d83dafba289710be2e538f4610e8cb53c025aa722bfe832ec4d6cbe33350c09b690c92560292893f72c7e9894a50efaaf9635d64c86b053053b861a00e1717d7b2b963782ea4fe407008153d2d0564e2cbe3792eaa0dacd611b9eaf9d3e7d5b54ab63ae9906b62c830ef4b873d954c25c22e8a221c9*$/pkzip2$:flag.png:flag.zip::flag.zip
```
This is a `PKZIP` hash (uncompressed). 

I will need to use a python script to extract the ecrypted bytes from the hash. Then I can feed them into `bkcrack` using 16 known plaintext bytes of a png (standard PNG header). 

Run this python to get the encrypted bytes:
```python
import binascii

hash_str = "$pkzip2$1*1*2*0*12c*120*c8a6617a*0*26*0*12c*c8a6*81bd*36bee62e49e2b2c41f6260bdc2e5fdd8cabd38956eb51f1d8a48c8f6228fd7392a8c53f3199068e3017e11c65e32cd55ea33033ab8b2fb52c4f86373098af1732591290e5c99a2a74239243b67108f232def15a73aac1537e75a593abe81fb3a8b0338afeb00835c67f8a31896a5f73facd1f481fd5ebc8882b5b183819f9b71c89506b3ae7d17bc07ab187ece8413a88af072018ccdc8a2db425082cec0715fd5aa3b3c47bb4f5c93b397154eb2212ffd593d0e4e614d83dafba289710be2e538f4610e8cb53c025aa722bfe832ec4d6cbe33350c09b690c92560292893f72c7e9894a50efaaf9635d64c86b053053b861a00e1717d7b2b963782ea4fe407008153d2d0564e2cbe3792eaa0dacd611b9eaf9d3e7d5b54ab63ae9906b62c830ef4b873d954c25c22e8a221c9*$/pkzip2$"

# extract the hex data segment (14th field when splitting by '*')
parts = hash_str.split('*')
ciphertext_hex = parts[13]

# convert hex to raw binary bytes and save it
ciphertext_bytes = binascii.unhexlify(ciphertext_hex)
with open("ciphertext.bin", "wb") as f:
    f.write(ciphertext_bytes)

print("Created ciphertext.bin")
```

Output:
```shell
python3 solve6.py  
Created ciphertext.bin
```

Create the file that contains the known plaintext:
```shell
printf "\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52" > plaintext.bin
```

Use `bkcrack` to get keys:
```shell
./bkcrack-1.8.1-Linux-x86_64/bkcrack -c ciphertext.bin -p plaintext.bin -o 0  
bkcrack 1.8.1 - 2025-10-25  
[21:10:13] Z reduction using 8 bytes of known plaintext  
100.0 % (8 / 8)  
[21:10:14] Attack on 789686 Z values at index 7  
Keys: c639d1ca b1fd3d6c 25bb9b08  
87.6 % (692148 / 789686)  
Found a solution. Stopping.  
You may resume the attack with the option: --continue-attack 692148  
[22:21:30] Keys  
c639d1ca b1fd3d6c 25bb9b08
```

Get the flag png:
```shell
./bkcrack-1.8.1-Linux-x86_64/bkcrack -c ciphertext.bin -k c639d1ca b1fd3d6c 25bb9b08 -o 0 -d flag.png  
bkcrack 1.8.1 - 2025-10-25  
[22:22:54] Writing deciphered data flag.png  
Wrote deciphered data.
```

`flag.png`:

![Alt text](/images/hashtoflag.png)

`tjctf{1ts_4ll_ab0ut_th3_keys}`

___

## unfinished-file

**Author**: jonathan

**Category**: forensics

**Description**: my stupid friend tried downloading this file before i shut my laptop down, what was he trying to do?

I download the challenge file `secret_archive.zip.crdownload` and inspect it:
```shell
file secret_archive.zip.crdownload  
secret_archive.zip.crdownload: data  

xxd secret_archive.zip.crdownload  
00000000: 4352 444c 0100 0000 6b04 0000 0000 0000  CRDL....k.......  
00000010: 2600 6874 7470 733a 2f2f 6578 616d 706c  &.https://exampl  
00000020: 652e 636f 6d2f 7365 6372 6574 5f61 7263  e.com/secret_arc  
00000030: 6869 7665 2e7a 6970 0000 0000 0000 0000  hive.zip........  
00000040: 1a0d 100a 0b0c 1678 6d42 3628 2136 2439  .......xmB6(!6$9  
00000050: 2c71 3471 301d 2e71 361d 7236 2a27 301d  ,q4q0..q6.r6*'0.  
00000060: 3271 7232 2e27 1d36 7237 212a 1d37 301d  2qr2.'.6r7!*.70.  
00000070: 2172 2f32 3736 2730 3f00 0000 0000 0000  !r/276'0?.......  
00000080: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000090: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
000000a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
000000b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
000000c0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
000000d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
000000e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
000000f0: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000100: 504b 0304 1400 0000 0000 0000 0000 ce9e  PK..............  
00000110: b06e 2900 0000 2900 0000 0a00 0000 7265  .n)...).......re  
00000120: 6164 6d65 2e74 7874 5468 6973 2066 696c  adme.txtThis fil  
00000130: 6520 6973 2069 6e63 6f6d 706c 6574 652e  e is incomplete.  
00000140: 204b 6565 7020 6c6f 6f6b 696e 672e 2e2e   Keep looking...  
00000150: 0a50 4b03 0414 0000 0000 0000 0000 003d  .PK............=  
00000160: e22e cb2f 0000 002f 0000 0010 0000 0068  .../.../.......h  
00000170: 6964 6465 6e2f 2e66 6c61 6764 6174 6136  idden/.flagdata6  
00000180: 2821 3624 392c 7134 7130 1d2e 7136 1d72  (!6$9,q4q0..q6.r  
00000190: 362a 2730 1d32 7172 322e 271d 3672 3721  6*'0.2qr2.'.6r7!  
000001a0: 2a1d 3730 1d21 722f 3237 3627 303f 504b  *.70.!r/276'0?PK  
000001b0: 0102 1400 1400 0000 0000 0000 0000 ce9e  ................  
000001c0: b06e 2900 0000 2900 0000 0a00 0000 00    .n)...)........
```

I put the file into cyberchef and cut off the beginning part of the data so I just have the zip file:
```shell
xxd temp-zip-downloaded.zip  
00000000: 504b 0304 1400 0000 0000 0000 0000 ce9e  PK..............  
00000010: b06e 2900 0000 2900 0000 0a00 0000 7265  .n)...).......re  
00000020: 6164 6d65 2e74 7874 5468 6973 2066 696c  adme.txtThis fil  
00000030: 6520 6973 2069 6e63 6f6d 706c 6574 652e  e is incomplete.  
00000040: 204b 6565 7020 6c6f 6f6b 696e 672e 2e2e   Keep looking...  
00000050: 0a50 4b03 0414 0000 0000 0000 0000 003d  .PK............=  
00000060: e22e cb2f 0000 002f 0000 0010 0000 0068  .../.../.......h  
00000070: 6964 6465 6e2f 2e66 6c61 6764 6174 6136  idden/.flagdata6  
00000080: 2821 3624 392c 7134 7130 1d2e 7136 1d72  (!6$9,q4q0..q6.r  
00000090: 362a 2730 1d32 7172 322e 271d 3672 3721  6*'0.2qr2.'.6r7!  
000000a0: 2a1d 3730 1d21 722f 3237 3627 303f 504b  *.70.!r/276'0?PK  
000000b0: 0102 1400 1400 0000 0000 0000 0000 ce9e  ................  
000000c0: b06e 2900 0000 2900 0000 0a00 0000 00    .n)...)........

file temp-zip-downloaded.zip  
temp-zip-downloaded.zip: Zip archive data, at least v2.0 to extract, compression method=store
```

After I extract the archive I get 2 new files:
```shell
pwd  
/home/user/Desktop/ctf/tjctf2026/temp-zip-downloaded  

ls -la  
total 4  
drwxrwxr-x 1 user user  32 May 16 17:21 .  
drwxrwxr-x 1 user user 796 May 16 17:21 ..  
drwxrwxr-x 1 user user  18 May 16 17:21 hidden  
-rw-rw-r-- 1 user user  41 May 16 17:20 readme.txt  

cat readme.txt  
This file is incomplete. Keep looking...  

cd hidden && ls -la  
total 4  
drwxrwxr-x 1 user user 18 May 16 17:21 .  
drwxrwxr-x 1 user user 32 May 16 17:21 ..  
-rw-rw-r-- 1 user user 47 May 16 17:20 .flagdata  

cat .flagdata  
6(!6$9,q4q0.q6r6*'02qr2.'6r7!*70!r/276'0?
```

I take the flag data: `6(!6$9,q4q0.q6r6*'02qr2.'6r7!*70!r/276'0?` and put it into cyberchef and run a XOR brute force. I find a valid key at `42`: [cyberchef link](https://gchq.github.io/CyberChef/#recipe=XOR(%7B'option':'Hex','string':'42'%7D,'Standard',false)&input=NighNiQ5LHE0cTAucTZyNionMDJxcjIuJzZyNyEqNzAhci8yNzYnMD8)

Output:
```shell
tjctf{n3v3rl3t0therp30plet0uchurc0mputer}
```

Add underscores to get the final valid flag!

`tjctf{n3v3r_l3t_0ther_p30ple_t0uch_ur_c0mputer}`

___

## invisible-ink

**Author**: samuel

**Category**: forensics

**Description**: Check out this cool PDF I found... I wonder if there's anything hidden inside!

I download the challenge file and inspect it:
```shell
file chall.pdf  
chall.pdf: PDF document, version 1.4, 2 page(s)

pdftotext chall.pdf

cat chall.txt  
Roses are red, violets are blue,  
CTFs are fun, finding flags is new.  
From web to crypto, we break the code,  
On the path to root, we lighten the load.  
  
There’s nothing here lol why are you looking  
  
  
Ok fine here’s the password: DBf8nEBgwRhZ
```

I get a password for something. These two lines were invisible when looking at the pdf:
```shell
There’s nothing here lol why are you looking

Ok fine here’s the password: DBf8nEBgwRhZ
```

Running `binwalk` on the file:
```shell
binwalk -e chall.pdf  
  
DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
198           0xC6            Zlib compressed data, default compression  
1755          0x6DB           Zlib compressed data, default compression  
8394          0x20CA          Zlib compressed data, default compression  
29326         0x728E          Zlib compressed data, default compression  
31464         0x7AE8          Zip archive data, encrypted at least v2.0 to extract, compressed size: 606709, uncompressed size: 613542, name: original_distorted.png  
  
WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

Run `unzip` on `7AE8.zip` and supply the password `DBf8nEBgwRhZ`:
```shell
unzip 7AE8.zip  
Archive:  7AE8.zip  
[7AE8.zip] original_distorted.png password:  
  inflating: original_distorted.png
```

Looking at `original_distorted.png`:

![Alt text](/images/original_distorted.png)

I opened the image in `GIMP` and went to `Filters > Distorts > Whirl and Pinch`. I set the `Whirl` setting to `-300` and save the image:

![Alt text](/images/original_undistorted.png)

`tjctf{p0lygl0t_f1les_4r3_50_c00l}`

___

## thomas-schools-of-china

**Author**: jonathan

**Category**: forensics

**Description**: I infiltrated our rival counterpart in china and found this file on one of the computers... never heard of this filetype before... hm.

Download the challenge file `chall.tsc`:
```shell
file chall.tsc  
chall.tsc: data  

xxd chall.tsc | wc -l  
917  

xxd chall.tsc | head  
00000000: 5453 4346 0100 0000 3c00 0000 3d00 0005  TSCF....<...=...  
00000010: 39db e9cc 00db e9cc 00db e9cc 00db e9cc  9...............  
00000020: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00000030: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00000040: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00000050: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00000060: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00000070: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00000080: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00000090: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  

xxd chall.tsc | tail  
000038b0: 00da e6cb 00d9 e7cb 00db e7ca 00db e7ca  ................  
000038c0: 00da e8ca 00da e8ca 00db e9cb 00db e9cc  ................  
000038d0: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
000038e0: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
000038f0: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00003900: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00003910: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00003920: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00003930: 00db e9cc 00db e9cc 00db e9cc 00db e9cc  ................  
00003940: 00                                       .
```

The file is a custom RGBA image format with a 16 byte header. The pixel data is 4 byte tuples (alpha, red, green, blue). The flag is hidden in pixels where the alpha channel is `0` and the RGB values are printable ASCII. I run this script which iterates over every 4 byte chunk, ignores any where `alpha != 0` or RGB values are non printable, and concatenates the valid text:
```shell
python3  
Python 3.13.5 (main, Apr  6 2026, 12:24:14) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> with open('chall.tsc', 'rb') as f:  
...     data = f.read()  
...  
... flag_chars = []  
... for i in range(0, len(data), 4):  
...     chunk = data[i:i+4]  
...     if len(chunk) == 4 and chunk[0] == 0:  
...         r, g, b = chunk[1], chunk[2], chunk[3]  
...         if 32 <= r <= 126 and 32 <= g <= 126 and 32 <= b <= 126:  
...             flag_chars.extend([r, g, b])  
...  
... flag = bytes(flag_chars).decode()  
... print(flag)  
...  
[ZZBBB555[[[GGG333NNN===!!!TTTJJJXXX676HHHfffRRRxxxbbbfffdddeee(((TTT---tjctf{c0ngr4ts_kkktttu_s0lv3d_my_f1st_CTF_chall!CCCOOO_btw_1_l1ke_b1rds}666  
>>> exit
```
The extra characters (`BBB`, etc) appear because some image pixels also contain printable ASCII.

`tjctf{c0ngr4ts_u_s0lv3d_my_f1st_CTF_chall!_btw_1_l1ke_b1rds}`

___

## triplets

**Author**: ansh

**Category**: forensics

**Description**: I was messing around with my image and it got really messed up... I see patterns...

I get the challenge file `chall.png`:

![Alt text](/images/chall888.png)

```shell
file chall.png  
chall.png: PNG image data, 1888 x 1888, 8-bit grayscale, non-interlaced  

exiftool chall.png  
ExifTool Version Number         : 13.25  
File Name                       : chall.png  
Directory                       : .  
File Size                       : 2.5 MB  
File Modification Date/Time     : 2026:05:16 19:27:44-04:00  
File Access Date/Time           : 2026:05:16 19:27:43-04:00  
File Inode Change Date/Time     : 2026:05:16 19:27:48-04:00  
File Permissions                : -rw-rw-r--  
File Type                       : PNG  
File Type Extension             : png  
MIME Type                       : image/png  
Image Width                     : 1888  
Image Height                    : 1888  
Bit Depth                       : 8  
Color Type                      : Grayscale  
Compression                     : Deflate/Inflate  
Filter                          : Adaptive  
Interlace                       : Noninterlaced  
Background Color                : 255  
Modify Date                     : 2025:10:26 21:47:42  
Datecreate                      : 2025-10-26T21:47:30+00:00  
Datemodify                      : 2025-10-26T21:47:30+00:00  
Datetimestamp                   : 2025-10-26T21:47:41+00:00  
Comment                         : 2000x594  
Image Size                      : 1888x1888  
Megapixels                      : 3.6
```

The comment in the metadata tells me the original size of the image: `2000x594`

Looking at it with `xxd`:
```shell
xxd chall.png | head  
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR  
00000010: 0000 0760 0000 0760 0800 0000 002b 6f03  ...`...`.....+o.  
00000020: e900 0000 0262 4b47 4400 ff87 8fcc bf00  .....bKGD.......  
00000030: 0000 0774 494d 4507 e90a 1a15 2f2a 5bc1  ...tIME...../*[.  
00000040: df90 0000 0025 7445 5874 6461 7465 3a63  .....%tEXtdate:c  
00000050: 7265 6174 6500 3230 3235 2d31 302d 3236  reate.2025-10-26  
00000060: 5432 313a 3437 3a33 302b 3030 3a30 3045  T21:47:30+00:00E  
00000070: ad21 6700 0000 2574 4558 7464 6174 653a  .!g...%tEXtdate:  
00000080: 6d6f 6469 6679 0032 3032 352d 3130 2d32  modify.2025-10-2  
00000090: 3654 3231 3a34 373a 3330 2b30 303a 3030  6T21:47:30+00:00  

xxd chall.png | tail  
00258ef0: ad81 ee74 32d7 c837 4aaf 29bd d5c8 4627  ...t2..7J.)...F'  
00258f00: e73a b9d3 f015 471f 0c34 e6e8 2f8e cef7  .:....G..4../...  
00258f10: c89d a36b 035d 6ae4 4627 1b0b 9d3b 3077  ...k.]j.F'...;0w  
00258f20: f0ad 49d7 3a5b 53f5 5627 5f39 de70 3cb7  ..I.:[S.V'_9.p<.  
00258f30: d09d 85ce 2db4 b0d0 9585 2ea8 fa49 6753  ....-........IgS  
00258f40: 0b5d 3b70 e9c0 8f36 ca38 bed1 c935 a72b  .];p...6.8...5.+  
00258f50: fe2b 7e63 c6e9 179d 4df5 bd8e 2197 1af9  .+~c....M...!...  
00258f60: a091 0b8d 7cd6 f0a5 866f 34fc 4dc3 ffb9  ....|....o4.M...  
00258f70: f0bf 8fff 7dfc ff1f ff2f b1fa 005d 6098  ....}..../...]`.  
00258f80: 8e91 0000 0000 4945 4e44 ae42 6082       ......IEND.B`.
```

`chall.png` stores a hidden color image as a sequence of RGB triplets 

The original PNG has `1888×1888 = 3,564,544` grayscale pixels. The hidden RGB image needs `2000×594×3 = 3,564,000`. So there are an extra 544 values at the end that need to be stripped when generating the final image. I run this python to generate the image:
```python
from PIL import Image
import numpy as np

img = Image.open('chall.png')
data = np.array(img).flatten()

total_rgb_pixels = 2000 * 594
needed = total_rgb_pixels * 3
# remove extra 544 bytes from the end
data = data[:needed]
rgb = data.reshape((total_rgb_pixels, 3))
# reshape to (594, 2000, 3)
result = rgb.reshape((594, 2000, 3))
Image.fromarray(result.astype(np.uint8)).save('output.png')
```

Output image:

![Alt text](/images/greyscale-to-color.png)

`tjctf{my_1m3g3_b3c3m3_bl3ck_&_wh1t3}`
