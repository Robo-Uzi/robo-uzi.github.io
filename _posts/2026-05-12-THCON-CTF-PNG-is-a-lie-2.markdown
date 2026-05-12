---
layout: post
title:  "PNG is a lie (part 2/2)"
date:   2026-05-12 19:34:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-PNG-is-a-lie-2/
---

**Category**: Steganography

**Author**: [chepycou](https://www.ar-lacroix.fr/)

**Description**: Well, now we know for certain that something lies in this file. We have sent your image to an elite steganographer here at SNAFU.

He has since become totally insane and keeps repeating that "the music's URL is the key"... do with this as you please.

I take `output.png` from the first part of the challenge and put it into `stegsolve`. I find this qr code when looking at `red plane 0`:

![Alt text](/images/red_plane_0.bmp)

Online qr code scanners didnt want to read it. After a while I decided to try my phone. Eventually the phone scanned the code and I copied this link from it: `https://www.youtube.com/watch?v=lpiB2wMc49g?flqg=THC{Y'411_s0_r1Ckr0113D}`

`THC{Y'411_s0_r1Ckr0113D}`