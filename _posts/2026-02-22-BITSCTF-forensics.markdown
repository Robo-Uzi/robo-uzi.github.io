---
layout: post
title:  "Forensics Challenges"
date:   2026-02-22 17:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BITSCTF-2026-forensics-challenges/
---
* TOC
{:toc}

## Marlboro

**Author**: Aur0r4

**Category**: forensics

**Description**: "The smoke rises, carrying secrets within... Where there's smoke, there's fire..."

We intercepted an encrypted transmission from a mysterious server. The data appears to be lost, but rumor has it some ancient legend is somewhere in this image.

The original sender was obsessed with a programming language from hell itself - one where code mutates, logic inverts, and sanity is optional and insanity is permanence.

I get the challenge file `Marlboro.jpg`:
```shell
file Marlboro.jpg  
Marlboro.jpg: JPEG image data, JFIF standard 1.02, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 3903x5854, components 3
```

`binwalk` extracts `smoke.png` and a file called `encrypted.bin` from a `zip` archive:
```shell
binwalk -e Marlboro.jpg  
  
DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
3754399       0x39499F        Zip archive data, at least v2.0 to extract, compressed size: 982911, uncompressed size: 982761, name: smoke.png  
4737377       0x484961        Zip archive data, at least v1.0 to extract, compressed size: 422, uncompressed size: 422, name: encrypted.bin

cd _Marlboro.jpg.extracted

ls  
39499F.zip  encrypted.bin  smoke.png
```

`smoke.png` contains a base64 encoded string in the author field of the metadata which decodes to the link [https://zb3.me/malbolge-tools/](https://zb3.me/malbolge-tools/). It also contains an XOR key:
```shell
zsteg smoke.png  
meta Author         .. text: "aHR0cHM6Ly96YjMubWUvbWFsYm9sZ2UtdG9vbHMv"  
b1,r,lsb,xy         .. text: "RfWWN\nvn&FSnCWcN#j"  
b1,rgb,lsb,xy       .. text: "# Marlboro Decryption Key\n# Format: 32-byte XOR key in hexadecimal\nKEY=c7027f5fdeb20dc7308ad4a6999a8a3e069cb5c8111d56904641cd344593b657\n# Usage: XOR each byte of encrypted.bin with key[i % 32]\np"  
b2,rgb,msb,xy       .. file: PDP-11 old overlay  
b3,r,lsb,xy         .. file: basic-16 executable  
b3,b,lsb,xy         .. text: "#,M-\rL?Pqodc"  
b3,b,msb,xy         .. text: "-6hJFBMv"  
b3,rgb,lsb,xy       .. file: zlib compressed data
```

On [https://gchq.github.io/](https://gchq.github.io/) I XOR `encrypted.bin` with the key `c7027f5fdeb20dc7308ad4a6999a8a3e069cb5c8111d56904641cd344593b657` and get this output:
```shell
Content-Type: text/x-malbolge
Interpreter: https://zb3.me/malbolge-tools/#interpreter
Language: Malbolge

D'`$##\!<5|jWVT/eRcONp_oJJHHF!3gfBc!?a_{)(xwYon4rqpi/mlkdibaf_%]ba`_XWVz=YRQPOsSRQ3ONGkKD,BAe(DCBA@?>7[5432V6v.3,P*).-&Jk)"!&}C#cy~wv<zyxZpotm3qSi/gOkd*KJ`ed]\"`Y^]\[TxRQVOTSLpPONMLEiIH*)?>bB$@987[Z:38765.Rsr*/.-&%$#G'~f$#z@~}|{t:[wvonm3kSonmlkdihg`&GFbaZ~^@VUTYRQPtNMLKPImGFKJCBf)E>bB;@?>7[5432Vw/43,POp.'&+*#G!~D1
```

Executing the malbolge code on [https://zb3.me/malbolge-tools/#interpreter](https://zb3.me/malbolge-tools/#interpreter) outputs the flag!

`BITSCTF{d4mn_y0ur_r34lly_w3n7_7h47_d33p}`
