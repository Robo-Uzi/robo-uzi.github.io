---
layout: post
title:  "HeartBroken"
date:   2025-10-18 16:43:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-heartbroken/
---

**Title:** HeartBroken

**Category:** misc

**Description:** I wrote a farewell letter to Rima and signed it, but I am suspecting someone tampered with my signature, help huh?

**Author:** Carthica

I download the challenge file `HeartBroken.pdf`:
```shell
file HeartBroken.pdf  
HeartBroken.pdf: PDF document, version 1.7 (zip deflate encoded)
```

I open the pdf in my browser. There is this text and an image of broken hearts below the text:
```pdf
To Rima:
I like leaving records, of everything, … thank you for the sign, that was all I needed Rima, may you die of old age, far away, as your beauty fade, all you feel is void, as all the cards you played, acting smart, were all fake ones, may you die alone, heartbroken, an ugly death as you became an ugly grandma, may you die a painful death, may you die a sudden death, unable to utter any last words, only then you shallow eyes may realize how absurd this life is, may u just fade, may you find the happiness you seek.
```

I click `Add a new signature` and then do `Ctrl + a` to select all. It selects the photo which was a signature. Drag it away to reveal the flag below the image.

`QnQSec{I_4ctually_st1ll_l0v3_y0u_Rima!!}`