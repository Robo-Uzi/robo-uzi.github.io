---
layout: post
title:  "BlackHat MEA Qualification CTF 2025"
date:   2025-09-08 21:57:10 -0400
author: robo.uzi
tags: [CTF]
---

**Title:** Down the Rabbit Hole

**Category:** Reversing

**Creator:** Flagyard

**Description:** Making the file huge means you can't just ChatGPT it.

I get a python script that is very long. 141021 lines.
```shell
cat main.py | wc -l  
141021
```

There is a ton of code here. Near the bottom I see this function:
```t
def _rebuild_expected_bytes():  
   out = []  
   for group in _PART_FN_GROUPS:  
       v = 0  
       for fname in group:  
           v ^= globals()[fname]("x") & 0xFF     
       out.append(v & 0xFF)  
   return bytes(out)
```

Run this script:
```python
import runpy

m = runpy.run_path("main.py")
flag_bytes = m["_rebuild_expected_bytes"]()
flag = flag_bytes.decode("utf-8")
print(flag)
```
`BHFlagY{i_th0ught_th1s_wa5_suposs3d_t0_b3_34sy_wh7_is_th3_fl4g_s0_l0ng}`
