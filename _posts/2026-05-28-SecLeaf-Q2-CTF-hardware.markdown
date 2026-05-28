---
layout: post
title:  "Hardware Challenges"
date:   2026-05-28 13:58:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /SecLeaf-Q2-CTF-2026-hardware/
---
* TOC
{:toc}

## View

**Category**: Hardware

**Description**: I’m here in the view, but down beneath.

I download the challenge files:
```shell
unrar x Files.rar  
  
UNRAR 7.12 freeware      Copyright (c) 1993-2025 Alexander Roshal  
  
Extracting from Files.rar  
  
Extracting  Drill_NPTH_Through.DRL                                    OK  
Extracting  Drill_PTH_Through.DRL                                     OK  
Extracting  Drill_PTH_Through_Via.DRL                                 OK  
Extracting  Gerber_BoardOutlineLayer.GKO                              OK  
Extracting  Gerber_BottomLayer.GBL                                    OK  
Extracting  Gerber_BottomSolderMaskLayer.GBS                          OK  
Extracting  Gerber_DocumentLayer.GDL                                  OK  
Extracting  Gerber_InnerLayer1.G1                                     OK  
Extracting  Gerber_InnerLayer2.G2                                     OK  
Extracting  Gerber_InnerLayer3.G3                                     OK  
Extracting  Gerber_InnerLayer4.G4                                     OK  
Extracting  Gerber_TopLayer.GTL                                       OK  
Extracting  Gerber_TopPasteMaskLayer.GTP                              OK  
Extracting  Gerber_TopSilkscreenLayer.GTO                             OK  
Extracting  Gerber_TopSolderMaskLayer.GTS                             OK  
All OK
```

I havent seen this file format before. I looked it up and found this ([https://en.wikipedia.org/wiki/Gerber_format](https://en.wikipedia.org/wiki/Gerber_format)):
```shell
The Gerber format is an open, ASCII, vector format for printed circuit board designs. It is the de facto standard used by PCB industry software to describe the printed circuit board images: copper layers, solder mask, legend, drill data, etc.
```

I downloaded an open source Gerber viewer called `gerbv` and ran `gerbv *` inside the directory with the files. Once it opens I see the pcb:

![Alt text](/images/gerbv-view.png)

Decode the string from `base64`:
```shell
echo 'U2VjTGVhZnt0SDNfZzNyQjNyX3YxM3defQ' | base64 -d  
SecLeaf{tH3_g3rB3r_v13w^}
```

`SecLeaf{tH3_g3rB3r_v13w^}`
