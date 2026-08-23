---
layout: post
title:  "Forensics Challenges"
date:   2026-08-23 16:01:00 -0400
author: uzi
tags: [CTF]
permalink: /hostile-takeover-ctf-2026-forensics/
---
* TOC
{:toc}

## QR Queries

**Category**: Forensics

**Description**: The flag is jumbled between these QR codes, can you revert it to something readable?

Flag SHA1: `97d2a5b4980d70bbbf16877c4b1039cb02d955a6`

File SHA1's:
- qr-24fb7b.gif: `d1c0ed1f934d643d316c0fd9e186c222639266ae`
- qr-44ea6a.gif: `7e1a00bce81b810f0a734899a28456872417f95e`
- qr-138875.gif: `7de773009db64fc6f53bcff7401b51d28f1752f5`
- qr-c22297.gif: `7a59dafba802ba63af7f8ed102f13c770acbc373`
- qr-da3121.gif: `0db2de0abf57493f974947021d5700c9cfda6aed`

I got the challenge files:
```shell
file *.gif  
qr1.gif: GIF image data, version 89a, 100 x 100  
qr2.gif: GIF image data, version 89a, 100 x 100  
qr3.gif: GIF image data, version 89a, 100 x 100  
qr4.gif: GIF image data, version 89a, 100 x 100  
qr5.gif: GIF image data, version 89a, 100 x 100
```

I scanned each file on [https://qrscanner.net/](https://qrscanner.net/) and got these flag parts:
```shell
TXT: ae7b53e10964} 
TXT: 6377- 
TXT: 89e1922e- 
TXT: 61a5-d788- 
TXT: TDHT{
```

I just need a combinated which matched this sha1 hash: `97d2a5b4980d70bbbf16877c4b1039cb02d955a6`

`TDHT{61a5-d788-89e1922e-6377-ae7b53e10964}`

___

## I can't hear you

**Category**: Forensics

**Description**: File SHA1: `4ab670ab1e135fbc6b3e0a6028814e5f654b71bd`

I get the challenge file:
```shell
file i_cant_hear_you.wav  
i_cant_hear_you.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 16000 Hz  

sha1sum i_cant_hear_you.wav  
4ab670ab1e135fbc6b3e0a6028814e5f654b71bd  i_cant_hear_you.wav
```

Open it in audacity and look at the spectrogram view:

![Alt text](/images/hostilespectro111.png)

`TDHT{EPMO1URCHXWZ}`

___

## Recycle Bits

**Category**: Forensics

**Description**: Oops, _I deleted the flag..._ File SHA1: `67f21a14069172deb011e06535e7b563020cd3d3`

I get the challenge file:
```shell
file recycled-bits.img  
recycled-bits.img: DOS/MBR boot sector, code offset 0x58+2, OEM-ID "mkfs.fat", Media descriptor 0xf8, sectors/track 32, heads 8, sectors 131072 (volumes > 32 MB), FAT (32 bit), sectors/FAT 1009, serial number 0x490185ec, unlabeled  

sha1sum recycled-bits.img  
67f21a14069172deb011e06535e7b563020cd3d3  recycled-bits.img
```

`strings` and `grep`:
```shell
strings recycled-bits.img | grep -i "TDHT{"  
TDHT{d92ivhxJmLuiUHhS}
```

`TDHT{d92ivhxJmLuiUHhS}`
