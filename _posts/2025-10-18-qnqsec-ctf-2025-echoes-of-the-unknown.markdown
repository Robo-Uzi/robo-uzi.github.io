---
layout: post
title:  "Echoes of the Unknown"
date:   2025-10-18 15:39:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-echoes-of-the-unknown/
---

**Title** Echoes of the Unknown

**Category:** stego

**Description:** A strange transmission was intercepted from deep space. It sounds like random noise — but maybe there’s more than what you hear.

**Author:** x.two

I get the file `alien.wav`:
```shell
file alien.wav  
alien.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 22050 Hz
```
The audio sounds like random noise. 

I open the file with audacity and switch the view from waveform to spectrogram:

![Alt text](/images/audio.png)

`QnQSec{h1dd3n_1n_4ud1o}`