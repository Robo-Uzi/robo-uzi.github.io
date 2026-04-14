---
layout: post
title:  "Take a Slice"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-take-a-slice/
---

**Category**: misc easy

**Author**: cosmicator

**Description**: It's in the name!

I get the challenge file:
```shell
file cake  
cake: data

xxd cake | head  
00000000: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000030: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000040: 0000 0000 0000 0000 0000 0000 0000 0000  ................  
00000050: 2a99 0000 59a9 053f 24a9 05bf 78a4 2cbf  *...Y..?$...x.,.  
00000060: f669 d740 6242 6140 2fd6 8b40 2a71 d940  .i.@bBa@/..@*q.@  
00000070: 5a24 6540 63e7 8b40 bbb3 d940 3d3c 6440  Z$e@c..@...@=<d@  
00000080: c774 8c40 0000 11a9 053f efa8 05bf d9a4  .t.@.....?......  
00000090: 2cbf 3c49 d940 2cc4 5e40 4840 8e40 c748  ,.<I.@,.^@H@.@.H  

xxd cake | tail  
001de9f0: 0000 76f9 8dbe cc07 5abf 0eae e33e 20e4  ..v.....Z....> .  
001dea00: 2041 8d1b 4940 1206 5e40 0150 0b41 56f3   A..I@..^@.P.AV.  
001dea10: 8e40 d14c a540 f287 2041 e706 4940 9af8  .@.L.@.. A..I@..  
001dea20: 5c40 0000 d9ef 7bbe 6a6c 59bf c021 ef3e  \@....{.jlY..!.>  
001dea30: 0150 0b41 56f3 8e40 d14c a540 e3d8 0a41  .P.AV..@.L.@...A  
001dea40: 9adc 8e40 fba5 a440 f287 2041 e706 4940  ...@...@.. A..I@  
001dea50: 9af8 5c40 0000 d9ef 7bbe 6a6c 59bf c021  ..\@....{.jlY..!  
001dea60: ef3e f287 2041 e706 4940 9af8 5c40 e3d8  .>.. A..I@..\@..  
001dea70: 0a41 9adc 8e40 fba5 a440 d410 2041 6ed9  .A...@...@.. An.  
001dea80: 4840 edaa 5b40 0000                      H@..[@..
```

The file is a binary `STL` file made up of:
- An 80 byte header
- A 4 byte unsigned integer indicating the number of triangles. In the `xxd` output: `2a99 0000` is `0x0000992a` in little endian which equals 39210 triangles
- The rest of the file made up of 50 byte records for each triangle

I load the file into [https://www.viewstl.com/#!](https://www.viewstl.com/#!):

![Alt text](/images/slicendice2.png)

Looks like a 3D printing file of a cake with lego studs on it.

Lowering the opacity reveals portions of the flag for me:

![Alt text](/images/slicendice.png)

I can read the text: `U   S SL1C3_&_D1`

Not sure why parts are missing but from that I can correctly guess the flag is:
`UMASS{SL1C3_&_D1C3}`
