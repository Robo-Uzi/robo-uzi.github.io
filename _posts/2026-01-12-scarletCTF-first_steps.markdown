---
layout: post
title:  "first_steps"
date:   2026-01-12 01:02:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-first_steps/
---

**Title**: first_steps

**Category**: Rev

**Author**: s0s.sh

**Description**: Find the flag hidden in the binary!

```shell
file first_steps  
first_steps: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=946f43e421efccc0bf5139d9938d355a82b7cb2e, for GNU/Linux 3.2.0, not stripped

./first_steps  
I was up late last night exploring the .rodata section, but I seem to have lost my flag!  
  
I'm sure it's around here somewhere... Can you find it for me? <3

strings first_steps | grep RUSEC  
RUSEC{well_th4t_was_eZ_WllwnZMjMCjqCsyXNnrtpDomWMU}
```

`RUSEC{well_th4t_was_eZ_WllwnZMjMCjqCsyXNnrtpDomWMU}`