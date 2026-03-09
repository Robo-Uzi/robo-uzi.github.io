---
layout: post
title:  "Happy new year"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Happy-new-year/
---

**Title**: Happy new year

**Category**: misc

**Author**: h11p13

**Description**: Learned from the best, Jacob Collier! Wrap the received message `[a-zA-Z_]` in `upCTF{...}`.

I get the challenge file:
```shell
file transmission.mp3  
transmission.mp3: MPEG ADTS, layer III, v1, 320 kbps, 48 kHz, Stereo
```

I opened the file in `audacity` and put it in the `spectrogram` view. There I see the message and get the flag:

![Alt text](/images/transmission_message.png)

`upCTF{XSTF_xo_Xo}`