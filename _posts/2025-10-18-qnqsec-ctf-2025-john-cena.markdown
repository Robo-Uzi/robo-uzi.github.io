---
layout: post
title:  "John Cena"
date:   2025-10-18 16:45:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-john-cena/
---

**Title:** John Cena

**Category:** stego

**Description:** You Can't See Me!

**Author:** x.two

I get the file `john-cena.gif`:
```shell
file john-cena.gif  
john-cena.gif: GIF image data, version 89a, 476 x 266
```

I read about the program `gifsicle` and install it. I run it with the `--info` flag on the gif to see how many frames there are:
```shell
gifsicle --info john-cena.gif  
* john-cena.gif 42 images  
 logical screen 476x266  
 global color table [256]  
 background 255  
 loop forever  
 + image #0 476x266  
   disposal background delay 0.04s  
 + image #1 476x266  
   local color table [256]  
   disposal background delay 0.05s  
 + image #2 476x266  
   local color table [256]  
   disposal background delay 0.04s  
 + image #3 476x266  
   local color table [256]  
   disposal background delay 0.04s  
 + image #4 476x266  
   local color table [256]  
   disposal background delay 0.05s  
 + image #5 476x266  
   local color table [256]  
   disposal background delay 0.04s  
 
 ...
   
 + image #38 476x266  
   local color table [256]  
   disposal background delay 0.05s  
 + image #39 476x266  
   local color table [256]  
   disposal background delay 0.04s  
 + image #40 476x266  
   local color table [256]  
   disposal background delay 0.04s  
 + image #41 476x266  
   local color table [4]  
   disposal background
```

I see on the final image (41) it contains 4 colors as compared to the normal 256. I run this command to save that frame: `convert john-cena.gif[41] frame_041.png`.

Finally I run stegsolve: `java -jar StegSolve-1.4.jar` and open the image. I look at the gray bits and can see the flag!

Final image:

![Alt text](/images/solved.jpg)

`QnQSec{HOW_CAN_YOU_SEE_ME?}`