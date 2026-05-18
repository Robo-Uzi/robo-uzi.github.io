---
layout: post
title:  "Misc Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RAMunchers-CTF-2026-misc/
---
* TOC
{:toc}

## Robot rambles

**Author**: not given

**Description**: Our team identified some strange sounds near where the Gibson is believed to be located. They recorded a sample of the audio and brought it back with them. Can you analyse it and find out the source?

I get the challenge file (almost 1 minute of audio):
```shell
file rambles.wav  
rambles.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 44100 Hz
```

I opened the file in audacity and it looks like this:

![Alt text](/images/encoded-signal.png)

It is some sort of encoded signal.

I tried to decode the audio on [https://www.modembin.com/sstv/decode](https://www.modembin.com/sstv/decode) and I got a blurry image:

![Alt text](/images/decoded_maybe.png)

I tried decoding the audio on [https://sstv-decoder.mathieurenaud.fr/](https://sstv-decoder.mathieurenaud.fr/) and I get a clear image! 

![Alt text](/images/decoded-image.png)

I'm not sure why one site failed and one produced a clear image.

`RMCTF{I_c4N_533_c13Ar1y_n0W!}`
