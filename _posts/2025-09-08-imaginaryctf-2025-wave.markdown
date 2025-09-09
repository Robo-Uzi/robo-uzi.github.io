---
layout: post
title:  "wave"
date:   2025-09-08 21:17:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /imaginaryctf-2025-wave/
---

**Title:** wave

**Category:** Forensics

**Creator:** Eth007

**Description:** not a steg challenge i promise

I get this file:
```shell
file wave.wav  
wave.wav: Audio file with ID3 version 2.3.0, contains:\012- RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 44100 Hz
```

Simply run:
```shell
exiftool wave.wav
Comment : ictf{obligatory_metadata_challenge}
```
