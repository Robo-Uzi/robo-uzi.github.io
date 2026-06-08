---
layout: post
title:  "Warmer Up"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-warmer-up/
---

**Title**: Warmer Up

**Author**: kidkraken

**Category**: Forensics

**Description**: What, the rules again?

I get the challenge file:
```shell
file rules1.pdf  
rules1.pdf: PDF document, version 1.5
```

run `pdftotext`:
```shell
pdftotext rules1.pdf  

cat rules1.txt | grep dalctf  
The schedule will be posted separately and can be found on the dalctf2026 discord.  
Flag format is dalctf{...} unless otherwise specified  
dalctf{d1d_y0u_r34d_th3m?}
```

`dalctf{d1d_y0u_r34d_th3m?}`