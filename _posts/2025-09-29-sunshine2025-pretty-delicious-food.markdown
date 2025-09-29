---
layout: post
title:  "Pretty Delicious Food"
date:   2025-09-29 12:41:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-pretty-delicious-food/
---

**Title:** Pretty Delicious Food

**Category:** forensics

**Author:** oatzs

**Description:** This cake is out of this world! :DDDDDDD

omnomonmonmonmonm

...

something else is out of place too.

_Note: This is not a steganography challenge_

I download the challenge file `prettydeliciouscakes.pdf` and open the pdf in my browser. There is an attachment called `payload.txt`. I download it and find the flag base64 encoded:
```shell
cat payload.txt  
const data = 'c3Vue3AzM3BfZDFzX2ZsQGdfeTAhfQ==';  
echo -n 'c3Vue3AzM3BfZDFzX2ZsQGdfeTAhfQ==' | base64 -d  
sun{p33p_d1s_fl@g_y0!}
```

`sun{p33p_d1s_fl@g_y0!}`